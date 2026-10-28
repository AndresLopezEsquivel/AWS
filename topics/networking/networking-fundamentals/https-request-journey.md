# The journey of an HTTPS request

This note follows one HTTPS request, `GET /cart` for `example.com`, from a
browser to a web server. It goes one step at a time and shows what each layer
adds on the client and removes on the server. For an overview of the layers,
see [The OSI model](osi-model.md).

The client is `203.0.113.10` and the server is `198.51.100.20`. Both are
addresses reserved for documentation.

**Sources for this section:**
- [RFC 5737, IPv4 Address Blocks Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc5737):
  Section 3, the documentation address blocks.

## The path

```mermaid
%%{init: {"flowchart": {"wrappingWidth": 420}}}%%
flowchart LR
    subgraph CLIENT ["CLIENT MACHINE · 203.0.113.10"]
        direction TB
        subgraph C_APP ["Browser (application)"]
            direction TB
            J1["(1) HTTP · layer 7<br/>writes the request<br/>GET /cart · Host: example.com"] --> J2["(2) TLS · often shown as layer 6<br/>encrypts the request"]
        end
        subgraph C_OS ["Operating system (kernel)"]
            direction TB
            J3["(3) TCP · layer 4<br/>adds ports<br/>source 51514 → destination 443"] --> J4["(4) IP · layer 3<br/>adds IP addresses<br/>source 203.0.113.10 → destination 198.51.100.20"] --> J5["(5) Ethernet · layer 2<br/>adds MAC addresses<br/>source: client → destination: home router"]
        end
        subgraph C_NIC ["Network card + driver"]
            direction TB
            J6["(6) Physical · layer 1<br/>sends the bits as signals<br/>electrical, light or radio"]
        end
        C_APP -- socket --> C_OS --> C_NIC
    end

    subgraph NET ["NETWORK"]
        direction TB
        SW["(7) Switch · layer 2<br/>reads the destination MAC"] --> R1["(8) Home router · layer 3<br/>reads the destination IP"] --> INET["(9) Internet · layer 3<br/>many routers, each reads the destination IP"] --> LAN["(10) Server's LAN · layers 3 and 2<br/>last router, then a switch"]
    end

    subgraph SERVER ["SERVER MACHINE · 198.51.100.20"]
        direction BT
        S_NIC["Network card + driver"] --> S_OS["Operating system<br/>(kernel)"] -- socket --> S_APP["Web server<br/>(application)"]
    end

    CLIENT -- signals --> NET -- signals --> SERVER
```

## How to read the steps

Each step ends with a line called "Message so far". It shows what the data
looks like once that step is done:

- **Square brackets** enclose the whole message as it exists at that point.
- **Each name** inside is a piece added by one layer, such as `TCP` for the
  TCP header.
- **A vertical bar** separates the pieces. They appear in the order they are
  sent, leftmost first.
- **Curly braces** mark content that is encrypted: `{HTTP}` is the HTTP
  request, present but unreadable.

For example, `[ TCP | TLS {HTTP} ]` is a message with a TCP header followed by
the HTTP request encrypted by TLS.

On the client, each step adds a piece, normally in front of what was already
there, so the message grows. On the server, each step removes one, until only
the HTTP request is left.

## Step 1: HTTP writes the request (layer 7)

The journey starts inside the browser. The user has asked for a page, and the
browser turns that into a message the web server can understand: an HTTP
request.

```
GET /cart HTTP/1.1
Host: example.com
```

`GET` is the method (the action), `/cart` is the path (what to fetch), and
`Host` is a header that names the site the request is for.

Before step 1, DNS resolution has already turned the domain name `example.com`
into the server's IP address. The port is inferred from the scheme: `https://`
means port 443 and `http://` means port 80, the default ports defined by the
HTTP standard.

The IP address identifies a machine on a network (in this case, the internet).
Strictly, it identifies one of the machine's network interfaces: a machine can
have several, each with its own address. The port identifies one program on
that machine: here, the web server software (for example Nginx, Apache or a
Node.js app) listening on port 443. That one program can serve several sites
on the same IP address and port, so the request needs the `Host` header to
identify the site.

The request doesn't include data such as the IP address, the port or the MAC
address, even though the browser already knows some of it (the IP address and
the port). Adding that information is the job of the lower layers.

This step shows what layer 7 is about: letting applications communicate with
each other. It does this through application-level protocols such as HTTP, SSH
and DNS. Layer 7 isn't the browser window or the business logic behind an API;
it's the protocols that make the communication possible, and the part of each
program that builds and parses their messages. A layer 7 protocol defines both
the structure of its messages and their meaning (for example, what a `GET`
request asks for). It is usually implemented by the application itself, while
the operating system normally provides layers 3 and 4. Layer 7 does nothing to
transport or deliver its messages; it hands them to the layers below.

"Application" here means neither the frontend nor the backend. Both are code
that uses layer 7; they don't implement it. The application is the program at
each end of the conversation that builds and parses the HTTP messages. In this
case, that's the browser (Chrome, Firefox) on the client side and the web
server software (Nginx, Apache, Node.js's built-in HTTP module) on the server
side. Your frontend JavaScript calls `fetch()`, and the browser builds the
HTTP request. Your backend business logic receives a request that has already
been parsed.

Message so far: `[ HTTP ]`

**Sources for this step:**
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110):
  Section 9.3.1, the GET method; Section 7.2, the `Host` header; Sections 4.2.1
  and 4.2.2, default ports 80 and 443.
- [RFC 9112, HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112):
  Section 3, the request line; Section 3.2, `Host` is required in every HTTP/1.1
  request.
- [WHATWG Fetch Standard](https://fetch.spec.whatwg.org/#forbidden-request-header):
  Section 2.2.2, `Host` is a forbidden request-header, so the browser sets it.
- [RFC 1034, Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034):
  Section 5.2.1, resolvers translate host names into addresses.
- [RFC 1122, Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122):
  Section 1.3.3, an IP address identifies a logical network interface, and a
  host can have several; Section 1.1.3, the application layer.
- [RFC 9293, Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293):
  Section 2.2, ports identify application services.
- [nginx: How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html):
  "Name-based virtual servers", choosing a site by the `Host` header.
- [Node.js `http` module](https://nodejs.org/api/http.html):
  introduction, the module parses messages into headers and body.
- [ITU-T X.200 (07/94), the OSI reference model](https://www.itu.int/rec/T-REC-X.200-199407-I/en):
  clause 7.1, the Application Layer.

## Step 2: TLS encrypts the request (often shown as layer 6)

The HTTP request from step 1 isn't encrypted. If the browser sent it as is,
anyone who could see the traffic on its way to the server (such as someone
on the same Wi-Fi network, or any router along the path) could read it.
That's why the browser passes it through TLS before handing it to the lower
layers for delivery.

TLS (Transport Layer Security) is a protocol that creates a secure channel
between two programs communicating over a network, such as a browser and a
web server. It gives that channel three guarantees: the server is always
authenticated with a certificate and a signature, so the browser knows who
it's talking to; only the two ends can read the data, because TLS encrypts it
(confidentiality); and nobody in between can change the data without being
detected (integrity). TLS doesn't deliver data itself: it relies on the layer
below to deliver its bytes complete and in order, which TCP does.

The channel is set up by the TLS handshake, a short exchange in which the
browser and the server agree on the cryptographic algorithms to use, the
server proves its identity, and both sides end up with shared keys that only
they know. It happens once per connection, right after the TCP connection is
established and before the first HTTP request; every request and response on
that connection is then protected with those keys.

HTTPS is HTTP sent through TLS. The messages are the same HTTP messages from
step 1, with the same methods and headers; what changes is that they travel
inside the TLS channel instead of as they are. An `https://` URL tells the
browser three things: it must set up TLS before sending any request, it must
check that the server's certificate matches the host in the URL (here,
`example.com`), and it connects to port 443. `http://example.com/cart` and
`https://example.com/cart` aren't the same resource reached two ways; the
HTTP standard treats them as separate sites.

Message so far: `[ TLS {HTTP} ]`. Together, `TLS {HTTP}` is what HTTPS
means: an HTTP request inside the TLS channel.

**Sources for this step:**
- [RFC 9846, TLS 1.3](https://www.rfc-editor.org/rfc/rfc9846) (replaces
  RFC 8446): Section 1, what TLS guarantees and its need for a reliable,
  in-order transport; Section 2, the handshake and the keys that protect traffic
  after it; Sections 4.5.1 and 4.5.2, the server's certificate and signature.
- [RFC 5280, X.509 Certificates](https://www.rfc-editor.org/rfc/rfc5280):
  Sections 3.1 and 3.2, certificates bind a public key to a subject and are
  signed by a certificate authority.
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110):
  Section 4.2.2, the `https` scheme, port 443, and no shared identity with
  `http`; Section 4.3.4, the certificate must match the host.

## Step 3: TCP adds ports (layer 4)

The browser hands `TLS {HTTP}` to the operating system through the socket.
From here down, the work is done by the operating system's kernel, not the
browser. The kernel first passes the data to TCP (Transmission Control
Protocol), which puts a TCP header in front of it.

The header carries two ports. The destination port, 443, identifies the web
server program on the server machine. The source port is a temporary number
the operating system picks for this connection, such as 51514. Ports chosen
this way are called ephemeral ports. The server sends its reply to that port,
which is how the client's operating system knows which connection, and so
which browser tab, the reply belongs to.

TCP has two jobs. The first is reaching the right program. The IP address
added in step 4 gets the data to the right machine, and the port gets it to
the right program on that machine. The second is delivering the data complete
and in order. Data can be lost, damaged or reordered on the way. TCP detects
that, resends what was lost and puts the bytes back in order. This is the
reliable delivery that step 2 said TLS depends on. To provide it, TCP first
sets up a connection between the two ends. That's the TCP connection the TLS
handshake waits for.

TCP sends data in pieces called segments, each with its own TCP header. A
large response may need many segments, but our request is small and fits in
one.

The ports are what layer 4 devices and services read later. On AWS, a
security group allows traffic by protocol, port and source, for example "TCP
port 443 from anywhere". It can see the port, but not `/cart`, which is inside
the encrypted HTTP request. A Network Load Balancer (NLB) also works at this
layer. It listens on a protocol and port and forwards each TCP connection to a
target, without reading the HTTP request. On the server, the operating system
uses destination port 443 to hand the data to the web server.

Message so far: `[ TCP | TLS {HTTP} ]`

**Sources for this step:**
- [RFC 9293, Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293):
  Section 2.2, ports, segments, and reliable, in-order delivery; Section 3.8,
  retransmission.
- [RFC 6056, Transport-Protocol Port Randomization](https://www.rfc-editor.org/rfc/rfc6056):
  Section 1, ephemeral ports are chosen by the networking stack.
- [ITU-T X.200 (07/94), the OSI reference model](https://www.itu.int/rec/T-REC-X.200-199407-I/en):
  clause 7.4.4.1.1, transport functions such as sequence control and error
  recovery.
- [Amazon VPC User Guide: Security group rules](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html):
  "Components of a security group rule", protocol, port range and source.
- [Elastic Load Balancing: What is a Network Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html):
  "Network Load Balancer overview", it works at layer 4; "Network Load Balancer
  components", listeners use a protocol and port.

## Step 4: IP adds addresses (layer 3)

Still inside the kernel, TCP hands the segment to IP (Internet Protocol),
which puts an IP header in front of it.

The header carries two addresses. The destination address, `198.51.100.20`,
is the server; DNS resolution found it before step 1. The source address,
`203.0.113.10`, is the client, so the server knows where to send the reply.
The header also records which protocol is inside (here, TCP) and a counter
called TTL (time to live), explained below. With the IP header added, the
data is called a packet.

IP has one job: getting data from one machine to another across many
networks. The port from step 3 picks the program, but only once the data has
reached the right machine. Getting it there is IP's job.

No device knows the whole path from the client to the server. Each router
only decides where the packet should go next, and each of those moves is
called a hop:

```
client → home router → ISP router → … many routers … → server's router → server
```

The first decision is made on the client. The kernel checks whether the
destination is on its own local network. It isn't, so the packet goes to the
default gateway, the nearest router. From there, every router does the same
thing: it reads the destination IP address, looks it up in its route table,
and forwards the packet to the next router. When several routes match, it
uses the most specific one. Routers don't read the ports or the HTTP request;
the destination address is all they need.

Each router also lowers the TTL by one. If the TTL reaches zero, the router
discards the packet and tells the sender. This stops a packet from circling
forever when routes are misconfigured. In IPv6, the same field is called Hop
Limit, which describes what it does in practice.

Two things follow from working hop by hop. First, the IP addresses stay the
same from end to end: every router sees source `203.0.113.10` and destination
`198.51.100.20`. Second, reaching the next router requires that router's
address on the local network, which IP doesn't provide. That's step 5.

IP doesn't guarantee delivery. Packets can arrive damaged, duplicated, out
of order, or not at all, and IP doesn't fix that. That's why TCP's
reliability from step 3 sits on top of it: IP moves each packet toward the
right machine, and TCP makes sure all the data arrives complete and in order.

On AWS, a VPC route table decides where traffic leaving a subnet goes next,
as a router's route table does. Each route has a destination, a range of IP
addresses written as a CIDR block, and a target, such as an internet gateway.
Traffic is matched by its destination IP address, and the most specific route
wins. A route `0.0.0.0/0 → internet gateway` matches every address, so
anything without a more specific route goes out to the internet. A network
ACL (NACL) filters traffic as it enters and leaves a subnet. Its rules match
on protocol, port range, and source or destination IP range, so like a
security group it reads layers 3 and 4. Unlike a security group, a NACL is
stateless: allowing inbound port 443 doesn't automatically allow the reply.
The reply goes back to the client's ephemeral port from step 3, so it needs
its own outbound rule. When the packet reaches the server, the server's
operating system sees that the destination address is its own, removes the
IP header, and passes the segment to TCP.

Message so far: `[ IP | TCP | TLS {HTTP} ]`

**Sources for this step:**
- [RFC 791, Internet Protocol](https://www.rfc-editor.org/rfc/rfc791):
  Section 1.2, delivery across interconnected networks, with no reliability
  mechanisms; Section 2.2, a datagram's path through a gateway; Section 3.1,
  the Time to Live and Protocol fields.
- [RFC 1122, Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122):
  Section 1.1.3, IP gives no delivery guarantees; Section 3.2.1.3, a host
  discards datagrams not addressed to it; Sections 3.3.1 and 3.3.1.1, the
  local/remote decision and sending to a gateway.
- [RFC 1812, Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812):
  Section 5.2.4.3, choosing the longest matching route; Section 5.3.1, each
  router lowers the TTL, and a packet at zero is discarded and reported.
- [RFC 8200, IPv6](https://www.rfc-editor.org/rfc/rfc8200):
  Section 3, the Hop Limit field.
- [Amazon VPC User Guide: Configure route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html):
  routes have a destination and a target, matched by destination IP address.
- [Amazon VPC User Guide: How route priority works](https://docs.aws.amazon.com/vpc/latest/userguide/route-tables-priority.html):
  "Longest prefix match", the most specific route wins.
- [Amazon VPC User Guide: Control subnet traffic with network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html):
  "Network ACL basics", subnet level and stateless.
- [Amazon VPC User Guide: Network ACL rules](https://docs.aws.amazon.com/vpc/latest/userguide/nacl-rules.html):
  the parts of a rule: protocol, port range, source or destination, allow or
  deny.

## Step 5: Ethernet adds the next device's address (layer 2)

Still inside the kernel, IP hands the packet down to Ethernet, together with
the address of the next hop it chose in step 4: the default gateway, the home
router. Ethernet puts an Ethernet header in front of the packet. The packet
then goes to the network card through its driver.

The header carries two MAC addresses. A MAC address identifies one network
interface, such as a network card. The destination MAC address,
`00:00:5e:00:53:02`, is the home router's interface. The source MAC address,
`00:00:5e:00:53:01`, is the client's network card. The header also records
what is inside, here an IPv4 packet. With the Ethernet header added, the data
is called a frame.

Ethernet has one job: getting the frame to the next device on the same local
network. IP and Ethernet answer different questions. The destination IP
address says where the data is going in the end. The destination MAC address
says who gets it next. Both kinds of address belong to a network interface,
not to a whole machine: a router has one interface on each network it
connects, each with its own addresses.

The client knows the router's IP address from its settings, but a frame needs
the router's MAC address. To find it, the client uses ARP (Address Resolution
Protocol). It sends a request to every device on the local network, asking
which device has that IP address. The router replies with its MAC address, and
the client keeps the answer in a cache, so it doesn't ask again for every
packet. IPv6 does the same job with Neighbor Discovery.

This is how the hops from step 4 happen. Each router removes the frame it
received, updates the IP header (for example, it lowers the TTL), and puts the
packet in a new frame for the next network. In the new frame, the source MAC
address is the router's own interface and the destination is the next router.
So the MAC addresses change at every hop, while the IP addresses stay the
same. The server's MAC address only appears on the last link, from the
server's router to the server.

Several kinds of device handle the frame on the way, and each reads only what
its job needs:

| Device | Layer | Connects | Reads |
|---|---|---|---|
| Switch | 2 | Devices on one local network | Destination MAC address |
| Router | 3 | Different networks | Destination IP address |
| Wi-Fi access point | 2 | Wireless devices to the local network | MAC addresses |

A switch learns which MAC addresses are on each of its ports and forwards
each frame by its destination MAC address. It works below IP, so it doesn't
read IP addresses. A router is the device that removes the frame and builds a
new one, as described above.

If the client is on Wi-Fi, the first link uses Wi-Fi frames instead of
Ethernet frames. They have the same job and also use MAC addresses. Ethernet
also covers layer 1: an "Ethernet cable" is a cable built to the physical
part of the Ethernet standard. That part is step 6.

On AWS, layer 2 is hidden. There are no switches to configure; what you
configure is routing, in route tables (step 4). Each elastic
network interface (ENI) has a MAC address, along with its IP addresses and
security groups. When an ENI is moved to another instance, its attributes,
including the MAC address, move with it. The VPC doesn't forward broadcast
frames between interfaces, and AWS checks that the MAC addresses in a frame
belong to the interfaces sending and receiving it. When the frame reaches the
server, the server's network card sees that the destination MAC address is
its own, the kernel removes the Ethernet header, and passes the packet to IP.

Message so far: `[ ETH | IP | TCP | TLS {HTTP} ]`

**Sources for this step:**
- [RFC 894, Transmission of IP Datagrams over Ethernet Networks](https://www.rfc-editor.org/rfc/rfc894):
  "Frame Format", IP packets are carried in Ethernet frames, with a type
  field for IP.
- [RFC 826, An Ethernet Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826):
  "Packet Generation", ARP finds the 48-bit Ethernet address for an IP address
  by broadcasting a request, and keeps answers in a table.
- [RFC 1122, Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122):
  Section 2.3.2.1, the ARP cache.
- [RFC 4861, Neighbor Discovery for IPv6](https://www.rfc-editor.org/rfc/rfc4861):
  Section 1, address resolution in IPv6.
- [RFC 1812, Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812):
  Section 1, a router removes the link layer header, changes the IP header,
  and adds a new link layer header.
- [RFC 4188, Definitions of Managed Objects for Bridges](https://www.rfc-editor.org/rfc/rfc4188):
  Section 3, bridges (switches) connect local network segments below the
  network layer.
- [RFC 7368, IPv6 Home Networking Architecture Principles](https://www.rfc-editor.org/rfc/rfc7368):
  Section 3.2.1, a switch learns MAC addresses on its ports.
- [RFC 9542, IANA Considerations and IETF Protocol and Documentation Usage for IEEE 802 Parameters](https://www.rfc-editor.org/rfc/rfc9542):
  Section 2.1.4, MAC addresses reserved for documentation.
- [IEEE 802.3-2022, Standard for Ethernet](https://standards.ieee.org/ieee/802.3/10422/):
  Ethernet defines both the MAC (layer 2) and the physical layer (layer 1).
- [Amazon EC2 User Guide: Elastic network interfaces](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html):
  a network interface has a MAC address, and its attributes follow it to
  another instance.
- [AWS Networking blog: Setting up Layer 2 networking on Amazon EC2](https://aws.amazon.com/blogs/networking-and-content-delivery/setting-up-layer-2-networking-on-amazon-ec2/):
  the VPC doesn't forward broadcast and multicast frames between ENIs; the
  hypervisor checks MAC addresses.

## Step 6: The network card sends the bits as signals (layer 1)

The driver hands the frame to the network card. To the card, the frame is a
string of bits, a row of 0s and 1s. A bit can't travel down a cable as a
number, so the card turns the bits into signals: physical changes that the
device at the other end can detect and turn back into the same bits. The
kind of signal depends on the medium, the path the signal travels through:

| Medium | Signal | Used by |
|---|---|---|
| Copper cable | Electrical | Ethernet |
| Optical fiber | Light | Ethernet |
| Air | Radio | Wi-Fi |

The physical layer has one job: carrying bits to the device at the other end
of the cable or radio connection. It reads nothing. It doesn't know which bits
are a MAC address or part of the HTTP request, and it adds no header. Its
standards define the plug and the cable, the signal levels that mean 0 and 1,
what each wire is for, and the steps to bring a connection up. For an
Ethernet cable plugged into a laptop, the shape of the plug and the voltages
both come from the physical part of the Ethernet standard.

A signal travels over one cable or radio connection, never further. Every
device that receives it turns it back into bits and sends a fresh signal on
the next connection. Take a laptop on Wi-Fi whose home router reaches the ISP
(Internet Service Provider, the company that sells the internet access) over
fiber. The bits reach the router as radio and leave as light. In between, the
router builds a new frame (step 5). So the signal changes on every
connection and the frame on every hop, while the IP packet inside stays the
same. One hop can include several connections: laptop → switch → home router
is one hop over two cables, because the switch doesn't read IP addresses. A
repeater works only at layer 1. It receives a signal and sends the same bits
on as a fresh signal, without reading them.

The physical layer delivers bits in the order they were sent, but not always
intact. A signal can be disturbed on the way, and bits can arrive damaged.
The physical layer doesn't fix this; at most, it reports a fault to layer 2.
The layers above deal with it. Ethernet checks each frame on arrival and
doesn't pass a damaged one up, and TCP on the client notices the missing data
and sends it again (step 3). A damaged bit costs a resend, which shows up as
delay, not as a failed request. A physical connection is described by its
transmission rate, or bandwidth, in bits per second (Ethernet ranges from
1 Mb/s to 400 Gb/s), by its delay, and by its error rate.

On AWS, the physical layer belongs to AWS. Under the shared responsibility
model, AWS runs the hardware, networking and facilities, so there are no
cables or network cards for you to manage. What you choose is bandwidth, and
only indirectly: an EC2 instance's network speed depends on its instance
type, and enhanced networking with the Elastic Network Adapter (ENA) reaches
up to 100 Gbps. The exception is AWS Direct Connect, a fiber-optic cable from
your router to an AWS router at a Direct Connect location. There you deal
with layer 1 yourself:
- the fiber type and transceiver AWS requires;
- a port speed of 1, 10, 100 or 400 Gbps, fixed once requested;
- the cross connect, the cable the facility installs between your equipment
  and AWS's.

AWS's Direct Connect troubleshooting guide starts with "layer 1 (physical)
issues". When the signals reach the server, its network card turns them back
into bits, and the frame goes up to be checked as described in step 5.

Message so far: `[ ETH | IP | TCP | TLS {HTTP} ]`, unchanged. Layer 1 adds
nothing; it sends these bits as signals.

**Sources for this step:**
- [ITU-T X.200 (07/94), the OSI reference model](https://www.itu.int/rec/T-REC-X.200-199407-I/en):
  - clause 7.7.2, the mechanical, electrical, functional and procedural
    means for bit transmission, and relaying within layer 1;
  - clause 7.7.3.3.1, bits are carried transparently;
  - clause 7.7.3.7, bits are delivered in order;
  - clause 7.7.3.8, fault notification to layer 2;
  - clause 7.7.3.9, error rate, transmission rate and transit delay.
- [IEEE 802.3-2022, Standard for Ethernet](https://standards.ieee.org/ieee/802.3/10422/):
  speeds from 1 Mb/s to 400 Gb/s, over coaxial, twisted pair or fiber optic
  cable.
- [IEEE 802.11-2024, Wireless LAN MAC and PHY](https://standards.ieee.org/ieee/802.11/10548/):
  the physical layer for Wi-Fi.
- [RFC 2108, IEEE 802.3 Repeater Devices MIB](https://www.rfc-editor.org/rfc/rfc2108):
  Section 2.2, Ethernet repeaters.
- [RFC 4861, Neighbor Discovery for IPv6](https://www.rfc-editor.org/rfc/rfc4861):
  Section 2.1, a link can span bridged (switched) Ethernets.
- [RFC 1812, Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812):
  Section 1, a router builds a new link layer header.
- [RFC 3635, Ethernet-like Interface Types MIB](https://www.rfc-editor.org/rfc/rfc3635):
  - Section 3.2.5, invalid frames are normally not passed up;
  - Section 4 (`dot3StatsFCSErrors`), frames that fail the frame check are
    counted as errors.
- [RFC 9293, Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293):
  Section 3.8, retransmission.
- [AWS shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/):
  AWS runs the hardware, networking and facilities.
- [Amazon EC2 User Guide: Enhanced networking](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html):
  speed per instance type, and ENA up to 100 Gbps.
- [Direct Connect User Guide: What is Direct Connect?](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html):
  fiber-optic cable to an AWS router, and the fiber and transceiver
  requirements.
- [Direct Connect User Guide: Dedicated connections](https://docs.aws.amazon.com/directconnect/latest/UserGuide/dedicated_connection.html):
  port speeds, and the cross connect.
- [Direct Connect User Guide: Troubleshoot layer 1 (physical) issues](https://docs.aws.amazon.com/directconnect/latest/UserGuide/ts_layer_1.html).

## Who reads what on the way

On its way to the server, the message passes devices and AWS services that
read parts of it to decide what to do with it. Reading a layer means looking
at its header to make that decision. Each of these players reads only as
deep as its job needs. Everything above that is payload: bytes it carries
without reading them. In the diagram, these are (7) to (10). For which
layers get changed on the way, see "What changes on the way" in
[The OSI model](osi-model.md#what-changes-on-the-way).

### Each device reads only what its job needs

| Device | Its job | Reads up to | What it looks at |
|---|---|---|---|
| Switch | Get the frame out of the right switch port | Layer 2 | Destination MAC address |
| Router | Get the packet one hop closer to the server | Layer 3 | Destination IP address |
| Web server | Answer the request | Layer 7 | Everything |

A switch's job is to move the frame out of the right switch port, on the
local network. For that, it needs only the destination MAC address: it
learns which MAC addresses sit behind each of its switch ports and sends the
frame out of the matching one. The IP header names machines across networks,
and the switch's job ends at "which switch port".

A router's job is to move the packet one hop closer to the server. For that,
it needs only the destination IP address, which it looks up in its route
table (step 4). The TCP header names programs, through TCP ports such as
443, and the router's job ends at "which machine".

"Port" means two different things here. A switch port is a physical socket
on the switch, where a cable plugs in. A TCP port is a number in the TCP
header that names a program on a machine.

The web server is where the message ends, so it removes and reads every
layer, as each step describes.

### The AWS players, by depth

On AWS, you don't configure switches or routers. You configure services, and
each one reads to a certain depth:

| Player | Where it sits | What its rules look at | Reads up to |
|---|---|---|---|
| Network ACL | At the edge of a subnet | Protocol, port range, source or destination IP range | Layers 3–4 |
| Security group | On a resource, such as an EC2 instance | Protocol, port range, source or destination (IP range or another security group) | Layers 3–4 |
| Network Load Balancer (NLB) | In front of its targets | Listener protocol and port; picks a target by protocol, IP addresses and ports | Layer 4 |
| Application Load Balancer (ALB) | In front of its targets | Path, host header, other HTTP headers, method, query string, source IP | Layer 7 |
| AWS WAF | Inside the service it's attached to, such as an ALB or CloudFront | Source IP, country, headers, text patterns, SQL injection, cross-site scripting | Layer 7 |

A load balancer spreads incoming requests across several targets, the
servers behind it. A listener is the protocol and port it accepts
connections on. WAF (web application firewall) allows, blocks or counts
HTTP requests.

AWS states the layer only for the load balancers: an NLB "functions at the
fourth layer" of the OSI model, and an ALB at "the seventh layer". For the
others, the layer follows from the rule fields. Take a security group rule
that allows TCP port 443 from `0.0.0.0/0`, which means any IPv4 address. The
source address comes from the IP header (layer 3), and so does the protocol
number that says the content is TCP. The port comes from the TCP header
(layer 4). No field reaches into the HTTP request. WAF works on HTTP and
HTTPS requests, so it reads layer 7.

### You can only decide on what you can see

A player can base a decision only on the fields it reads, so how deep it
reads sets how fine its choices can be:

| Reads up to | Can tell apart | So it can allow or block |
|---|---|---|
| Layer 3 | Machines, by IP address | Everything from or to a machine or network |
| Layer 4 | Programs, by TCP port | One service, such as HTTPS on port 443 |
| Layer 7 | Requests, by path, headers or content | One page, or one kind of request |

Take `https://example.com/home` and `https://example.com/admin`, requested
by the same visitor. To a security group, the two look identical: same
protocol, same port 443, same source IP address. The only difference, the
path, is at layer 7. So a security group can let HTTPS in or keep it out,
but only as a whole. For the same reason, it can't block a request that
contains SQL injection, since that is text inside the request. WAF can.

A rule has to read as deep as its deepest field. To make `/admin` reachable
only from an office's IP range, a rule needs the source IP address
(layer 3) and the path (layer 7), so it needs a layer 7 player. That can be
an ALB listener rule with both conditions, or a WAF rule that matches the
path and excludes the office's IP range.

Routing follows the same pattern. An NLB chooses a target for each
connection, by protocol, IP addresses and ports; its listener rules can only
route by IP version (IPv4 or IPv6). An ALB reads each request, so it can
send `/api` to one group of targets and everything else to another.

### Encryption sets a ceiling

With HTTPS, TLS encrypts everything above it (step 2): the path, the headers
and the body. Only the two ends of the TLS connection can read it.
Everything below TLS stays readable, because the devices on the way need it
to deliver the message. What a device can read is not the same as what it
does read:

| Field | Can a router on the way read it? | Does it read it? |
|---|---|---|
| Destination IP address | Yes | Yes: its job needs it |
| Destination TCP port | Yes: it isn't encrypted | No: its job doesn't need it |
| Path `/admin` | No: it's encrypted | — |

So without the keys, a device can read at most up to layer 4. For an ALB to
read the request, the ALB has to be one end of the TLS connection. This is
called terminating TLS: the device holds the server's certificate and
private key, decrypts the request, reads it, and sends it to the target over
a new connection, either as plain HTTP or encrypted again with HTTPS. The
opposite is passing TLS through: the device forwards the encrypted bytes
without reading them, and the target decrypts them.

| Player | What it does with TLS |
|---|---|
| ALB, HTTPS listener | Terminates it, with a certificate such as one from AWS Certificate Manager (ACM). An HTTPS target group encrypts again to the targets. ALB listeners are HTTP or HTTPS only, so an ALB can't pass TLS through. |
| CloudFront | Uses HTTPS with the visitor, with its own certificate or one from ACM. The connection to the origin, the server behind CloudFront, is set up separately, as HTTP or HTTPS. |
| NLB, TLS listener | Terminates it, with a server certificate. |
| NLB, TCP listener on port 443 | Passes it through; the targets decrypt. |
| WAF | Doesn't decrypt. The ALB or CloudFront forwards the request to WAF for inspection. |

Terminating TLS at the load balancer gives layer 7 choices, such as routing
by path and attaching WAF, and takes the work of decryption off the servers.
If the servers must decrypt for themselves, an NLB with a TCP listener fits:
it still sees IP addresses and TCP ports, which is all it needs to spread
connections. It gives up routing by path, and WAF, which can't be attached
to an NLB.

A device that terminates TLS can also change layer 7, the exception noted in
"What changes on the way". For example, the server behind an ALB sees the
ALB's IP address as the source, not the visitor's. So by default, the ALB
adds the visitor's IP address to the request, in a header called
`X-Forwarded-For`.

These depths are the minimum each player needs, not a hard limit. An NLB
works at layer 4, but with a TLS listener it decrypts. And some information
above layer 4 can be read without decrypting. At the start of TLS, the
browser sends the server's name, `example.com`, in a field called SNI
(Server Name Indication), and in TLS 1.3 this field is not encrypted. AWS
Network Firewall, which filters traffic entering and leaving a VPC, can
allow or deny connections by that name.

### Deeper costs more

Reading deeper takes more work. A router reads a few fields at fixed places
in the IP header. A layer 7 player has to decrypt first, then read the
request as text and compare it with patterns. AWS shows this cost in two
ways:
- **Decryption is work.** A TLS listener on an NLB, or an HTTPS listener on
  an ALB, offloads "the work of encryption and decryption" from the servers.
- **WAF prices its rules by effort.** Each WAF rule uses web ACL capacity
  units (WCUs) that reflect its relative cost. A regex check uses more than
  a size check, and inspecting a JSON body more again. WAF also inspects
  only part of a large request: the first 8 KB of the body behind an ALB,
  and by default the first 16 KB behind CloudFront (up to 64 KB).

A DDoS (distributed denial of service) attack is many machines flooding a
target to make it unavailable. AWS names three kinds, by layer:

| Layer | Kind of attack | Example |
|---|---|---|
| 3 | Volumetric: fills the network | A flood of packets |
| 4 | Protocol: abuses a protocol | A SYN flood: masses of TCP connection openings that use up the target's connection state |
| 7 | Application: valid-looking requests, in huge numbers | A flood of `GET /search` requests |

AWS Shield Standard is automatic and free. It defends against common
layer 3 and layer 4 attacks at the perimeter, the first point where traffic
enters AWS. With CloudFront, Route 53 or Global Accelerator in front, that
perimeter is the edge of the AWS network, far from the servers. Shield
Advanced is a paid service that also mitigates layer 7 attacks
automatically. WAF inspects the content at layer 7 and can also limit how
many requests a client sends in a given window, with a rate-based rule.

Put together: floods are stopped cheaply at layers 3 and 4, near the edge,
and only the remaining traffic gets the costlier inspection at layer 7. AWS
doesn't state this as a rule; it follows from how the services split the
work.

**Sources for this section:**
- [RFC 7368, IPv6 Home Networking Architecture Principles](https://www.rfc-editor.org/rfc/rfc7368):
  Section 3.2.1, a switch learns MAC addresses on its ports.
- [RFC 1812, Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812):
  Section 5.2.4.3, a router looks up the destination address.
- [RFC 9846, TLS 1.3](https://www.rfc-editor.org/rfc/rfc9846):
  Section 1, data is visible only to the endpoints.
- [RFC 6066, TLS Extensions](https://www.rfc-editor.org/rfc/rfc6066):
  Section 3, the Server Name Indication extension.
- [RFC 9849, TLS Encrypted Client Hello](https://www.rfc-editor.org/rfc/rfc9849):
  Section 1, the SNI is left unencrypted in TLS 1.3.
- [Amazon VPC User Guide: Security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
  and [Security group rules](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html):
  the parts of a rule: protocol, port range, source or destination.
- [Amazon VPC User Guide: Network ACL rules](https://docs.aws.amazon.com/vpc/latest/userguide/nacl-rules.html):
  the parts of a rule.
- [ELB: What is a Network Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html):
  works at layer 4; target selection by flow hash.
- [ELB: Listeners for your Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-listeners.html):
  TLS listeners terminate and decrypt; TCP listeners pass encrypted traffic
  through; listener rules route by IP version.
- [ELB: What is an Application Load Balancer?](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html):
  works at layer 7; routing on path, host, headers, method, query string
  and source IP; certificates from ACM.
- [ELB: Listeners for your Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html):
  HTTP and HTTPS listeners; rules with several conditions.
- [ELB: Target groups for your Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html):
  HTTP and HTTPS to targets.
- [ELB: HTTP headers and Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/x-forwarded-headers.html):
  `X-Forwarded-For`, appended by default.
- [CloudFront Developer Guide: Require HTTPS between viewers and CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/using-https-viewers-to-cloudfront.html):
  HTTPS with viewers, separate from HTTPS to the origin.
- [AWS WAF Developer Guide: What is AWS WAF?](https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html):
  resources it protects; request criteria and actions.
- [AWS WAF: Logical rule statements](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statements-logical.html):
  combining and negating statements.
- [AWS WAF: Rate-based rule statement](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html):
  limiting requests that come too fast.
- [AWS WAF: Web ACL capacity units](https://docs.aws.amazon.com/waf/latest/developerguide/aws-waf-capacity-units.html):
  rule capacity reflects relative cost.
- [AWS WAF: Oversize request components](https://docs.aws.amazon.com/waf/latest/developerguide/waf-oversize-request-components.html):
  body inspection limits.
- [AWS Shield: How Shield works](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-overview.html):
  attack classes at layers 3, 4 and 7.
- [AWS Shield Standard overview](https://docs.aws.amazon.com/waf/latest/developerguide/ddos-standard-summary.html):
  network and transport layer protection at the perimeter.
- [AWS Network Firewall: What is AWS Network Firewall?](https://docs.aws.amazon.com/network-firewall/latest/developerguide/what-is-aws-network-firewall.html)
  and [Stateful domain list rule groups](https://docs.aws.amazon.com/network-firewall/latest/developerguide/stateful-rule-groups-domain-names.html):
  filtering at the VPC perimeter, by the TLS SNI or HTTP `Host` header.
