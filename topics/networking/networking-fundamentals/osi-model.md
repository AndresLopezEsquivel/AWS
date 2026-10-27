# The OSI model

The OSI model divides network communication into seven layers. Each layer has
one job and relies on the layer below to do its own. These notes use OSI layer
numbers to say where a protocol or an AWS service sits, and what it can see.
The Internet uses the TCP/IP model, which groups the same work into four
layers. The table maps one model onto the other.

The "Journey step" column points to the step in
[The journey of an HTTPS request](https-request-journey.md) where each layer
does its work. A dash means the journey doesn't show that layer.

| Layer | Job | Unit | Example protocols | TCP/IP layer | Journey step |
|---|---|---|---|---|---|
| 7 Application | Lets applications communicate: defines their messages and what they mean | Message | HTTP, DNS, SSH | Application | Step 1 |
| 6 Presentation | Agrees on how data is represented, such as format and compression | — | None separate (see notes) | Application | Step 2 (TLS, by convention) |
| 5 Session | Organizes the dialogue between the two ends: opening, closing, synchronizing | — | None separate (see notes) | Application | — |
| 4 Transport | Delivers data between two programs, identified by ports, and (TCP) makes delivery reliable and in order | Segment (TCP), datagram (UDP) | TCP, UDP | Transport | Step 3 |
| 3 Network | Delivers data between machines across networks, hop by hop, using IP addresses | Packet (IP datagram) | IPv4, IPv6 | Internet | Step 4 |
| 2 Data link | Delivers data between devices on the same local network, using MAC addresses | Frame | Ethernet, Wi-Fi | Link | Step 5 |
| 1 Physical | Turns bits into signals on a cable or over the air | Bit | Ethernet and Wi-Fi physical layers | Link (see notes) | Step 6 |

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

## Sources

- [ITU-T X.200 (07/94), the OSI reference model](https://www.itu.int/rec/T-REC-X.200-199407-I/en):
  clauses 7.1–7.7, the purpose of each layer.
- [RFC 1122, Requirements for Internet Hosts: Communication Layers](https://www.rfc-editor.org/rfc/rfc1122):
  the four TCP/IP layers and how the application layer relates to OSI
  (Section 1.1.3); segment, datagram, packet and frame (Section 1.3.3).
- [RFC 9110, HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110):
  HTTP messages.
- [RFC 9846, TLS 1.3](https://www.rfc-editor.org/rfc/rfc9846):
  TLS runs over a reliable transport.
- [RFC 9293, Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293):
  segments, ports, and reliable, in-order delivery (Section 2.2).
- [RFC 768, User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768):
  datagrams and ports.
- [RFC 791, Internet Protocol](https://www.rfc-editor.org/rfc/rfc791):
  datagrams delivered across networks to fixed-length addresses.
- [RFC 8200, IPv6](https://www.rfc-editor.org/rfc/rfc8200):
  IPv6 as the successor to IPv4.
- [IEEE 802.3-2022, Standard for Ethernet](https://standards.ieee.org/ieee/802.3/10422/):
  MAC and physical layer.
- [IEEE 802.11-2024, Wireless LAN MAC and PHY](https://standards.ieee.org/ieee/802.11/10548/):
  MAC and physical layer for Wi-Fi.
- [IEEE Registration Authority FAQ](https://standards.ieee.org/faqs/regauth/):
  MAC addresses identify network interfaces.
