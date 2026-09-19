# 06 — AWS-Side Setup: VPC, EC2 & strongSwan

## VPC & Subnets

We first created a VPC with the address range 10.0.0.0/16. We then divided it into two subnets according to their roles.

<img width="881" height="160" alt="image" src="https://github.com/user-attachments/assets/ebd0aa8f-0f4b-49f3-9503-46bd6ca53f99" />

The first subnet, 10.0.10.0/24, is the public subnet containing the VPN gateway (as a reminder, it needs Internet connectivity to establish the IPsec tunnel with the on-premise MikroTik)

The second subnet, 10.0.20.0/24, is the private subnet containing the application server (as a reminder it has no direct Internet access, so access to this resource from the on-premise network goes through the VPN gateway)

## EC2 Instance (VPN gateway) Setup

- OS: Ubuntu: strongSwan compatibility, tooling familiarity, simple administration
- Private IP: 10.0.10.107
- Elastic IP: 13.63.102.32 (static public IP, required so the MikroTik side always knows where to reach the gateway)

  <img width="889" height="193" alt="image" src="https://github.com/user-attachments/assets/57731def-e2ce-43e4-8a3c-2634148dc5a1" />

The EC2-strongswan is deployed in the public subnet while the EC2-utilisateurs (simulating the enterprise resources, an EC2 running Ubuntu, ip 10.0.20.101) is in the private one

## Installing strongSwan

We needed a VPN gateway on AWS that could establish a site-to-site IPsec tunnel with the MikroTik router on the on-premise side.

We therefore looked for an IPsec implementation supporting IKEv2, compatible with MikroTik, and giving us direct control over the VPN configuration. strongSwan met these requirements and could be deployed directly on an Ubuntu EC2 instance.

It was therefore selected as the software VPN gateway between the AWS VPC and the on-premise network.

You can install it like this:
The latest version of strongSwan at the time i'm writing this is the 6.0.7 but you can check on their website at ```https://download.strongswan.org/```:
```bash
wget https://download.strongswan.org/strongswan-6.0.7.tar.bz2
```
Unzip it:
```bash
tar -xjf strongswan-6.0.7.tar.bz2
```
Navigate into the directory and configure strongSwan (I configure it with this options but):
```bash
cd strongswan-6.0.7
./configure --prefix=/usr/local --sysconfdir=/etc --enable-systemd --enable-swanctl --with-systemdsystemunitdir=/usr/lib/systemd/system
```
(If you run into errors at this step, it's probably because you lack one or all of the following: gcc, libssl-dev, pkg-config, libsystemd-dev (easily installed))

Now you can build the binaries and install them as root
```bash
make
sudo make install
```

## Verifying the Service Is Running

Charon is the main daemon of strongSwan responsible for handling IKE negotiations and managing the IPsec Security Associations. In our case, it is the component that establishes and maintains the IKEv2 connection with the MikroTik and manages the IPsec SAs used to protect the traffic.

Before modifying the configuration, checking the state of the daemon is important to make sure that the strongSwan service was actually running and operational:
```bash
sudo systemctl status strongswan
```
You'll see active (running), enabled. If not, something definetely wrong.

## The swanctl.conf Configuration

All the configuration for the tunnel sits in this file. You need to specify the remote VPN gateway, traffic selectors, authentication method, and some other parameters. You can check their website for reference and syntax ```https://docs.strongswan.org/docs/latest/swanctl/swanctlConf.html```

This is the content of the /etc/swanctl/swanctl.conf for this project. 

```
connections {

    mikrotik-aws {

        version = 2

        remote_addrs = <YOUR PUBLIC IP>

        proposals = aes256-sha256-ecp256

        local {
            auth = psk
            id = keyid:EC2-vpn-aws
        }

        remote {
            auth = psk
            id = keyid:mikrotik-onprem-canal
        }

        children {

            aws-onprem {

                local_ts = 10.0.20.0/24
                remote_ts = 192.168.0.0/16

                esp_proposals = aes256gcm16-ecp256

                start_action = trap
                rekey_time = 1h
            }
        }

        dpd_delay = 30s
        rekey_time = 8h
    }
}


secrets {

    ike-psk {

        id-1 = keyid:EC2-vpn-aws
        id-2 = keyid:mikrotik-onprem-canal

        secret = "<YOUR SECRET>"
    }
}
```

Lets Walk through it parameter by parameter:

1. ```version = 2```

This specifies that we use IKEv2 for the VPN negotiation. We chose IKEv2 because it is the modern version of IKE and is supported by both strongSwan and the MikroTik router.

2. ```proposals = aes256-sha256-ecp256```

This defines the cryptographic proposal for the IKE SA: AES-256 for encryption, SHA-256 for integrity, and ECP-256 for the Diffie-Hellman key exchange. These parameters define how the two gateways secure the IKE control channel.

3. Local/remote identities
   
```
id = keyid:EC2-vpn-aws
id = keyid:mikrotik-onprem-canal
```

These are the identities used to identify the two VPN peers during IKE authentication. We explicitly defined them instead of relying only on their IP addresses, so each side can verify which peer it is communicating with.

4. Authentication by PSK
   
```
auth = psk
```

PSK means Pre-Shared Key. Both gateways authenticate using the same secret key, which is stored in the secrets section and associated with the two configured identities.
The PSK is not used to encrypt the application traffic directly; it is used to authenticate the peers during IKE

5. children, Traffic Selectors
   
```
local_ts = 10.0.20.0/24
remote_ts = 192.168.0.0/16
```

The Child SA defines which traffic must be protected by IPsec. local_ts represents the AWS application network, while remote_ts represents the on-premise network. Therefore, traffic between these two networks is selected for encryption through the IPsec tunnel.

6. ```esp_proposals = aes256gcm16-ecp256```
   
This defines the cryptographic parameters for the Child SA, which protects the actual data traffic through ESP. AES-256-GCM provides authenticated encryption, while ECP-256 can be used for the Diffie-Hellman exchange during Child SA rekeying, providing PFS when configured and negotiated.

8. ```remote_addrs = <YOUR PUBLIC IP>```
   
This specifies the public IP address of the remote MikroTik peer. We used the site's actual static public IP because the remote peer is known and has a fixed public address. This provides an additional restriction on which peer the gateway expects to establish the connection with.
The same public IP is also used to restrict administrative and testing access in the AWS Security Group. So the restriction exists both at the VPN configuration level and at the AWS network-access level, where applicable.

10. Rekey timers
    
```rekey_time = 8h```

The IKE SA is rekeyed periodically, here after eight hours, so the long-lived IKE security association is not kept indefinitely.

And :

```rekey_time = 1h```

The Child SA protecting the actual application traffic is rekeyed more frequently, every hour in this configuration. This limits the lifetime of the traffic-encryption keys and can also be combined with PFS when a DH group is configured for Child SA rekeying.

## Loading & Verifying the Configuration

Load the configs from the '/etc/swanctl/' directory:

```bash
sudo swanctl --load-all
```

You can confirm whether the configs were successfully loaded or not with:
```bash
sudo swanctl --load-conns
```
