## What a VPN Actually Does

A VPN establishes a logical secure tunnel over a public network, guaranteeing confidentiality, integrity, and authentication between two endpoints. Data traveling through the tunnel is encrypted (unreadable) to anyone except the two endpoints.

## Two Architectural Types

- **Site-to-site VPN**: connects two or more distinct networks (this project's case), used when whole networks need to talk to each other transparently.
- **Remote-access VPN**: connects an individual user to central resources from anywhere.

## Protocols

Build a comparison covering, at minimum:

| Layer / Technology        | Main characteristics                                                                                                                                | Limitations / Reason for not choosing it                                                                                                                                                                                                                                         |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Layer 2 – PPTP**        | Tunnels PPP over IP.                                                                                                                                | Considered obsolete due to weaknesses in its security mechanisms. It is therefore not appropriate for a new secure interconnection. ([RFC Editor][1])                                                                                                                            |
| **Layer 2 – L2TP/IPsec**  | L2TP provides the tunnelling of PPP traffic, while IPsec provides authentication, confidentiality, integrity and replay protection.                 | L2TP itself does **not** provide tunnel protection; IPsec is required. It adds an additional tunnelling layer that was unnecessary for the routed site-to-site architecture of the project. ([RFC Editor][2])                                                                    |
| **Layer 3 – IPsec/IKEv2** | Protects IP traffic at the network layer. IKEv2 handles negotiation and authentication, while ESP protects the traffic.                             | **Selected solution.** Well suited to routed site-to-site connectivity and interoperable between MikroTik and strongSwan.                                                                                                                                                        |
| **Layer 3 – WireGuard**   | Modern, lightweight VPN using modern cryptographic primitives such as Curve25519, ChaCha20 and Poly1305. ([WireGuard][3])                           | A valid alternative, but the project retained IKEv2/IPsec because of the existing MikroTik/strongSwan architecture and the established IPsec feature set. Do not claim WireGuard has no key rotation: it performs periodic handshakes and rotates session keys. ([WireGuard][3]) |
| **TLS/SSL – OpenVPN**     | Flexible VPN using TLS for authentication/key exchange and supporting UDP/TCP transport. ([GitHub][4])                                              | More suited to flexible client/server VPN deployments. Traditional OpenVPN also had userspace data-plane overhead, although modern DCO significantly reduces this overhead. ([OpenVPN Blog][5])                                                                                  |
| **TLS/SSL – SSTP**        | Encapsulates PPP over HTTPS/TLS, typically using TCP 443, which makes it effective at traversing many firewalls and proxies. ([Microsoft Learn][6]) | Closely associated with the Microsoft ecosystem and less appropriate for this multi-vendor site-to-site architecture. Microsoft is also retiring SSTP support in Azure VPN Gateway because of limited capability and performance. ([Microsoft Learn][7])                         |

[1]: https://www.rfc-editor.org/info/rfc2637/ "RFC 2637: Point-to-Point Tunneling Protocol (PPTP) | RFC Editor"
[2]: https://www.rfc-editor.org/info/rfc3193/ "RFC 3193: Securing L2TP using IPsec | RFC Editor"
[3]: https://www.wireguard.com/protocol/ "Protocol & Cryptography - WireGuard"
[4]: https://github.com/OpenVPN/openvpn/blob/master/doc/man-sections/tls-options.rst "openvpn/doc/man-sections/tls-options.rst at master · OpenVPN/openvpn · GitHub"
[5]: https://blog.openvpn.net/openvpn-data-channel-offload-dco-the-definitive-guide-to-the-performance-boost-making-openvpn-the-fastest-vpn-protocol? "OpenVPN Data Channel Offload (DCO): The Definitive Guide to the Performance Boost Making OpenVPN The Fastest VPN Protocol"
[6]: https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-sstp/70adc1df-c4fe-4b02-8872-f1d8b9ad806a? "[MS-SSTP]: Overview | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/azure/vpn-gateway/ikev2-openvpn-from-sstp "SSTP protocol retirement and connections migration - Azure VPN Gateway | Microsoft Learn"


## Why This Matters for Site-to-Site Specifically

For a site-to-site VPN, the objective is to securely connect entire networks, not individual users or devices. The VPN gateways establish the tunnel between the two sites, and the traffic between their private subnets is protected automatically.

This makes Layer 3 VPN technologies such as IPsec/IKEv2 particularly suitable for the project. They can securely route traffic between networks with different private address spaces, such as the enterprise network 192.168.0.0/24 and the AWS application network 10.0.20.0/24, without requiring users to establish a VPN connection themselves.

The choice is therefore not only about encryption. It also concerns routing, interoperability, authentication, availability, and the ability to connect complete networks in a controlled way
