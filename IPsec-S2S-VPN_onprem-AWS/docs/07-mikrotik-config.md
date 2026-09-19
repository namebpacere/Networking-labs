## Hardware & Software

- Model: MikroTik RB4011iGS+RM
- RouterOS 7.23.2
- Configured via Winbox (MikroTik's official GUI management tool)

**This router has the AL21400 CPU which has an IPsec hardware acceleration, that it matters ? not really. But be aware that some routers from vendors have this feature. They have a dedicated ship for crypto processes which offloads crypto work from the main CPU keeps performance high even at higher throughput --> faster operations**

## Step 1: IPsec Profile (Phase 1 parameters)

This is your IKE SA proposal --> `proposals` in swanctl.conf 

I noted one limitation though. RouterOS's Profile menu doesn't expose AES-GCM as an option, unlike the Proposal menu used for ESP.

<img width="599" height="731" alt="image" src="https://github.com/user-attachments/assets/5df526ed-e7c5-43ac-9387-17fae575925b" />


For my case, NAT-T must be enabled here.

## Step 2: IPsec Peer (Phase 1 endpoint)

The remote endpoint that terminates the tunnel, the EC2-strongSwan instance. The connection is initiated by the MikroTik router.

## Step 3: Proposal & Identity (Phase 2 / ESP parameters)

`aes-256 gcm` alone is sufficient here, no separate authentication algorithm is needed because it's built into GCM. It provides encrytion + integrity/authentication in one operation --> faster operation

Identity: both sides must use matching identity strings (`EC2-vpn-aws` / `mikrotik-onprem-canal`). A mismatch here is one of the most common tunnel failure causes

<img width="567" height="557" alt="image" src="https://github.com/user-attachments/assets/f0a58f07-18bb-48f7-baa4-038404094afc" />

<img width="578" height="667" alt="image" src="https://github.com/user-attachments/assets/2a427a46-7022-4716-b8da-88eac4bd35d5" />


## Step 4: IPsec Policy

It defines *which traffic* gets encrypted (source 192.168.0.0/24 --> destination 10.0.20.0/24), pointing to the peer and the proposal created above. 

**If a Policy doesn't match, IPsec never gets invoked at all** and traffic just goes out in the clear or gets dropped, depending on firewall rules.

<img width="566" height="494" alt="image" src="https://github.com/user-attachments/assets/6d06d982-0ba8-4b68-8f24-b6db57bf0568" />


## Firewall Rule

An explicit accept rule is needed for 192.168.0.0/24 --> 10.0.20.0/24
