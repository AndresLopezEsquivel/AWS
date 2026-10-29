# The OSI model

The OSI model divides network communication into seven layers. Each layer has
one job and relies on the layer below to do its own. These notes use OSI layer
numbers to say where a protocol or an AWS service sits, and what it can see.
The Internet uses the TCP/IP model, which groups the same work into four
layers. The table maps one model onto the other.

The "Example devices" column names the device whose main job sits at that
layer. A device reads its own layer and the ones below it, and a dash means
no common device sits there.

[The journey of an HTTPS request](https-request-journey.md) follows one
request through these layers: each layer adds its piece on the client, and
the server removes them in reverse order.

| Layer | Job | Unit | Example protocols | Example devices | TCP/IP layer |
|---|---|---|---|---|---|
| 7 Application | Lets applications communicate: defines their messages and what they mean | Message | HTTP, DNS, SSH | — | Application |
| 6 Presentation | Agrees on how data is represented, such as format and compression | — | None separate (see notes) | — | Application |
| 5 Session | Organizes the dialogue between the two ends: opening, closing, synchronizing | — | None separate (see notes) | — | Application |
| 4 Transport | Delivers data between two programs, identified by ports, and (TCP) makes delivery reliable and in order | Segment (TCP), datagram (UDP) | TCP, UDP | — | Transport |
| 3 Network | Delivers data between machines across networks, hop by hop, using IP addresses | Packet (IP datagram) | IPv4, IPv6 | Router | Internet |
| 2 Data link | Delivers data to the next device on the same local network, using MAC addresses | Frame | Ethernet, Wi-Fi | Switch, Wi-Fi access point | Link |
| 1 Physical | Turns bits into signals on a cable or over the air | Bit | Ethernet and Wi-Fi physical layers | Repeater | Link (see notes) |

**Layers 5 and 6 in TCP/IP.** TCP/IP has no separate session or presentation
layer. Its application layer covers OSI layers 5 to 7, so applications such as
HTTP do that work themselves.

**TLS at layer 6 is a convention.** The OSI model describes layer 6 as data
representation and doesn't mention encryption. TLS sits between the
application and TCP, so textbooks often place it at layer 6, and these notes
follow that.

**Layer 1 in TCP/IP.** The TCP/IP standard defines a link layer and doesn't
mention a physical layer. Putting layers 1 and 2 under "Link" is a textbook
mapping. Ethernet and Wi-Fi each cover both layers: their standards define the
MAC (layer 2) and the physical layer (layer 1).

## What changes on the way

The sender adds the layers and the receiver removes them. This table is
about the devices in between: which layers they change while the message
travels.

A **hop** is one trip from one device that reads IP addresses (a computer,
a server or a router) to the next. A switch doesn't read IP addresses, so
it doesn't end a hop: laptop → switch → home router is one hop over two
cable connections.

| Layer | Changed on the way? | By whom |
|---|---|---|
| 7 HTTP | No | Nobody: it's encrypted, so the devices in between can't read it |
| 6 TLS | No | Nobody: TLS detects any change made on the way |
| 4 TCP | No | Nobody: only the two ends use it |
| 3 IP | Only a little | Every router lowers the TTL by one; the addresses stay |
| 2 Ethernet | Yes, on every hop | Every router builds a new frame, with new MAC addresses |
| 1 Physical | Yes, on every cable or radio connection | Every device sends a fresh signal; the bits stay the same |

The lower the layer, the more often it changes. Layers 4 to 7 travel
untouched from end to end, while layer 1 changes on every cable or radio
connection.

Two exceptions:
- **NAT** (network address translation), which home routers usually do,
  rewrites the IP addresses and TCP ports.
- **A device that decrypts TLS**, because it holds the keys (such as a load
  balancer), ends the TLS connection. It can read layer 7 and change it.

## Sources

- [ITU-T X.200 (07/94), the OSI reference model](https://www.itu.int/rec/T-REC-X.200-199407-I/en):
  clauses 7.1–7.7, the purpose of each layer; clause 7.7.2, devices in
  between may relay bits within layer 1; clause 7.7.3.3.1, bits are carried
  transparently.
- [RFC 1122, Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122):
  the four TCP/IP layers and how the application layer relates to OSI
  (Section 1.1.3); segment, datagram, packet and frame (Section 1.3.3).
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110):
  HTTP messages.
- [RFC 9846, TLS 1.3](https://www.rfc-editor.org/rfc/rfc9846):
  TLS runs over a reliable transport; Section 1, data is visible only to
  the endpoints and can't be modified without detection.
- [RFC 9293, Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293):
  segments, ports, and reliable, in-order delivery (Section 2.2).
- [RFC 768, User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768):
  datagrams and ports.
- [RFC 791, Internet Protocol](https://www.rfc-editor.org/rfc/rfc791):
  datagrams delivered across networks to fixed-length addresses.
- [RFC 8200, IPv6](https://www.rfc-editor.org/rfc/rfc8200):
  IPv6 as the successor to IPv4.
- [RFC 1812, Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812):
  Section 1, routers forward packets at the network layer, removing the
  link layer header and adding a new one; Section 5.3.1, each router
  decreases the TTL.
- [RFC 4861, Neighbor Discovery for IPv6](https://www.rfc-editor.org/rfc/rfc4861):
  Section 2.1, a link can span bridged (switched) Ethernets.
- [RFC 3022, Traditional NAT](https://www.rfc-editor.org/rfc/rfc3022):
  Section 2.2, NAPT translates addresses and ports.
- [RFC 4188, Definitions of Managed Objects for Bridges](https://www.rfc-editor.org/rfc/rfc4188):
  Section 3, bridges (switches) connect local network segments below the
  network layer.
- [RFC 2108, IEEE 802.3 Repeater Devices MIB](https://www.rfc-editor.org/rfc/rfc2108):
  Section 2.2, Ethernet repeaters.
- [IEEE 802.3-2022, Standard for Ethernet](https://standards.ieee.org/ieee/802.3/10422/):
  MAC and physical layer.
- [IEEE 802.11-2024, Wireless LAN MAC and PHY](https://standards.ieee.org/ieee/802.11/10548/):
  MAC and physical layer for Wi-Fi.
- [RFC 4118, Architecture Taxonomy for CAPWAP](https://www.rfc-editor.org/rfc/rfc4118):
  Section 1.1, an access point provides the 802.11 MAC services to wireless
  clients and connects to a layer 2 network; Section 3.1, the 802.11
  definition of an access point.
- [IEEE Registration Authority FAQ](https://standards.ieee.org/faqs/regauth/):
  MAC addresses identify network interfaces.
