### IPsec & IKEv2 Deep Dive
This is the technical core of the project.

## What IPsec Actually Is

IPsec is a suite of protocols and mechanisms (not a single protocol) defined across several IETF RFCs, designed to secure IP traffic at the network layer, independent of the application.

## The Three Security Guarantees

- **Confidentiality** means that the content of the communication cannot be read by an unauthorized party. In your implementation, this is provided by encryption, specifically AES-based protection through the protocol ESP
- **Integrity** means that the receiver can detect whether the packet has been modified during transmission. With classic ESP proposals, this can be provided by HMAC; with AES-GCM, encryption and integrity/authentication are combined in an AEAD algorithm
- **Authentication** verifies the identity of the VPN peer before protected traffic is exchanged

## Why ESP, Not AH

AH, defined by RFC 4302, provides integrity, data-origin authentication and anti-replay protection, but it does not provide confidentiality. AH uses IP protocol number 51.

ESP, defined by RFC 4303, can provide confidentiality, integrity, data-origin authentication and anti-replay protection. ESP uses IP protocol number 50

AH is problematic with NAT because it protects IP-header information, while NAT modifies IP addressing. ESP, on the other hand, can be encapsulated in UDP/4500 using NAT Traversal (NAT-T). IKEv2 detects NAT during IKE_SA_INIT; when NAT is detected, subsequent IKE and ESP traffic use UDP 4500
(NAT-T was particularly important in our architecture because the MikroTik router is behind NAT)

## Tunnel Mode vs. Transport Mode

**Transport mode**: the original IP header remains visible, and IPsec mainly protects the upper-layer payload.
**Tunnel mode**: The entire original IP packet is encapsulated and protected, then a new outer IP header is added for the VPN gateways. RFC 4301 defines both transport and tunnel modes

I used tunnel mode because the objective was to connect two IP networks

### IKEv2: Establishing the Tunnel Before Any Data Flows

## IKE_SA_INIT 

During IKE_SA_INIT, the peers negotiate the cryptographic parameters, exchange nonces and perform a Diffie-Hellman exchange. The resulting shared secret is calculated independently by both peers; it is never transmitted directly over the network. The nonces add fresh randomness to the subsequent key derivation

### IKE_AUTH

During IKE_AUTH, the two peers authenticate each other using the authentication methode (PSK, X.509 certificates, EAP methods) configured on both sides. Once authentication succeeds, the first Child SA is established to protect the actual user traffic

## IKE SA vs. Child SA

The IKE SA protects the control channel used to negotiate and manage the VPN. The Child SA protects the actual user traffic through ESP. A single IKE SA can manage multiple Child SAs, each with its own security parameters and lifetime

## Perfect Forward Secrecy (PFS)

PFS limits the impact of a compromise by using a fresh Diffie-Hellman exchange when establishing new Child SA keys. Therefore, compromising one set of session keys does not automatically reveal previously established session keys.

## Anti-Replay Protection

ESP uses the SPI to identify the Security Association and a sequence number to track packets. The receiver uses these values with an anti-replay window to detect and reject replayed packets

## Dead Peer Detection (DPD)

Dead Peer Detection periodically checks whether the remote peer is still reachable. If the peer is considered unreachable, the IPsec state can be renegotiated according to the configured actions
