## Architecture

You can find the actual draw.io file in the diagram folder

<img width="1168" height="667" alt="image" src="https://github.com/user-attachments/assets/44bd6e26-16ba-4290-a6b3-0cb8a0b52d10" />

## IP Addressing Plan

| Segment | Network | Role |
|---|---|---|
| On-premise LAN | 192.168.0.0/24 | User-facing local network |
| AWS VPC | 10.0.0.0/16 | Overall cloud address space |
| AWS Public subnet | 10.0.10.0/24 | Hosts the VPN gateway |
| AWS Private subnet | 10.0.20.0/24 | Hosts application resources |

## Why Two AWS Subnets, Not One

The two subnets allow us to separate the VPN gateway from the application resources. The VPN gateway is placed in the public subnet because it needs Internet connectivity to establish the IPsec tunnel. The application resources, on the other hand, are placed in a private subnet with no direct Internet access. They can therefore only be reached from the company network through the VPN tunnel.

This separation is primarily a security decision. The gateway that needs to be exposed to the Internet is isolated from the application resources, which reduces their exposure. So, it is not simply a networking convenience; it is part of the security architecture

## Why the Gateway Is a Self-Managed EC2 Instance, Not AWS Site-to-Site VPN

AWS Site-to-Site VPN would have provided a fully managed VPN service, with two tunnels designed for redundancy and automatic failover on the AWS side.

With our EC2-based approach, we had to manage the VPN gateway ourselves, including the strongSwan configuration, updates, monitoring and availability. The EC2 instance is also a single point of failure in the current implementation.

However, this approach give us much more control over the VPN configuration and allowed us to work directly with IKEv2 and IPsec at the protocol level. It also reduced the cost compared with using a managed VPN service, although the exact cost depends on the AWS region and traffic

## Critical EC2 Configuration Details

By default, AWS expects an EC2 instance to be either the source or the final destination of the traffic it receives. But our EC2 instance is acting as a VPN gateway, so it must forward packets whose source and destination are other machines.

For example, a packet coming from the on-premise network may be destined for 10.0.20.101, not for the VPN gateway itself. Therefore, the source/destination check must be disabled so that AWS allows the instance to forward this traffic.”

AWS explicitly requires this setting for instances performing routing, NAT or firewall functions.

However this is not enough. You need to allow IP packets forwarding at the OS layer, i mean directly on the VPN gateway console with:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## Routing: The Return Path

The private subnet has a route for the on-premise network 192.168.0.0/24, with the VPN gateway's network interface as the next hop. Therefore, when the application server sends a packet to an address in 192.168.0.0/24, AWS forwards it to the gateway.

The gateway then routes the packet through the IPsec tunnel toward the MikroTik router and the on-premise network.

Without this return route, the tunnel could technically be established and traffic could reach the application server, but the response would not know where to go. We would therefore have a return-path problem and the communication would fail

## Security Group Design

You will need to allow the protocols required by the VPN and by administration and testing:

- UDP 500 is used by IKE for negotiating the IPsec security association. 
- UDP 4500 is required for NAT Traversal, which is relevant because the MikroTik is behind a NAT device. 
- ESP, IP protocol 50, carries the encrypted IPsec traffic when NAT-T is not encapsulating ESP in UDP.

I also allowed SSH on TCP port 22 for administration and ICMP for connectivity testing.

SSH and ICMP were not exposed to the whole Internet. Access was restricted to the known static public IP of the on-premise site.”
