# The journey of an HTTPS request

This note follows one HTTPS request, `GET /cart` for `example.com`, from a
browser to a web server. It goes one step at a time and shows what each layer
adds on the client and removes on the server. For an overview of the layers,
see [The OSI model](osi-model.md).

The client is `203.0.113.10` and the server is `198.51.100.20`. Both are
addresses reserved for documentation.

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
        C_OS["Operating system<br/>(kernel)"]
        C_NIC["Network card + driver"]
        C_APP -- socket --> C_OS --> C_NIC
    end

    subgraph NET ["NETWORK"]
        direction TB
        SW["Switch"] --> R1["Router"] --> INET["Internet<br/>many routers"] --> LAN["Server's LAN<br/>last router and switch"]
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

## Sources

- [ITU-T X.200 (07/94), the OSI reference model](https://www.itu.int/rec/T-REC-X.200-199407-I/en):
  clause 7.1, the Application Layer.
- [RFC 1034, Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034):
  name-to-address translation.
- [RFC 1122, Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122):
  the application layer, and IP addresses per network interface.
- [RFC 5737, IPv4 Address Blocks Reserved for Documentation](https://www.rfc-editor.org/rfc/rfc5737).
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110):
  default ports, methods, the `Host` header, the `https` scheme and
  certificate verification.
- [RFC 9112, HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112):
  the request line, and `Host` as a required header.
- [RFC 9293, Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293):
  what ports identify.
- [RFC 9846, TLS 1.3](https://www.rfc-editor.org/rfc/rfc9846) (replaces
  RFC 8446): what TLS guarantees, the handshake, certificates and signatures.
- [WHATWG Fetch Standard](https://fetch.spec.whatwg.org/):
  headers the browser controls, such as `Host`.
- [nginx: How nginx processes a request](https://nginx.org/en/docs/http/request_processing.html):
  choosing a site by the `Host` header.
- [Node.js `http` module](https://nodejs.org/api/http.html):
  message parsing.
