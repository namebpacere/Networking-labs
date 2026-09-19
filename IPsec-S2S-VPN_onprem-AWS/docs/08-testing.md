> A tunnel showing "up" in a status page proves nothing on its own. This section is about the layered evidence needed to actually trust the result.


Technicaly, a successful ping proves *reachability*, and that traffic went through the encrypted tunnel, but not that it's actually encrypted. Each test below closes one of those gaps.

## Test 1: Reachability

From the on-prem local network, i issued a ping to the EC2-utilisateurs in the private subnet in AWS. This alone proves that traffic went through the encrypted tunnel because this instance is inside a private subnet, which has not direct Internet access. The only way to reach it is by the tunnel, so from the on-prem MikroTik router.

<img width="831" height="245" alt="image" src="https://github.com/user-attachments/assets/a82bd5e6-5377-4052-97a8-d1a6639965ae" />

But we can still verify that Traffic actually uses the tunnel by just looking at the statistics of the tunnel on strongswan:

```bash
sudo swanctl --list-sas
```

You'll see somthing like that
<img width="862" height="194" alt="image" src="https://github.com/user-attachments/assets/d231079a-43bb-4157-b75e-ec5e787a32ea" />
`in` for reveived and decrypted packets and `out` for encrypted sent packets

## Test 2: Confirming Encryption

To be able to do this capture I mirrored the `ether1` port to `ether5` port on the MikroTik and did a ping to a PC on the on-prem network, so Wireshark could observe WAN-side traffic. and what the capture showed:
> - ESP encapsulated inside UDP/4500 (confirms NAT-T is active)
> - Payload unreadable — only transport metadata visible (source/destination gateway IPs, sequence number, SPI)
> - This confirms confidentiality was observed on the traffic captured, not a formal cryptographic audit of the cipher's strength.

<img width="886" height="729" alt="image" src="https://github.com/user-attachments/assets/84a54c9f-a723-4e2e-b01e-d2aaa3de9d4a" />
