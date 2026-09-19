## Hardware & Software

- Model: MikroTik RB4011iGS+RM
- RouterOS 7.23.2
- Configured via Winbox (MikroTik's official GUI management tool)

**This router has the AL21400 CPU which has an IPsec hardware acceleration, that it matters ? not really. But be aware that some routers from vendors have this feature. They have a dedicated ship for crypto processes which offloads crypto work from the main CPU keeps performance high even at higher throughput --> faster operations**

## Step 1: IPsec Profile (Phase 1 parameters)

This is your IKE SA proposal —-> `proposals` in swanctl.conf 

I noted one limitation though. RouterOS's Profile menu doesn't expose AES-GCM as an option, unlike the Proposal menu used for ESP.

<img width="599" height="731" alt="image" src="https://github.com/user-attachments/assets/5df526ed-e7c5-43ac-9387-17fae575925b" />


For my case, NAT-T must be enabled here.

## Step 2: IPsec Peer (Phase 1 endpoint)

The remote endpoint that terminates the tunnel, the EC2-strongSwan instance. The connection is initiated by the MikroTik router.

```
<img width="526" height="421" alt="image" src="https://github.com/user-attachments/assets/32fac177-10f2-4601-a4de-e9ad4ad07059" />

```

## Step 3: Proposal & Identity (Phase 2 / ESP parameters)

> ✍️ YOUR TURN:
> - Proposal: explain why `aes-256 gcm` alone was sufficient here (no separate authentication algorithm needed — it's built into GCM) — this should mirror the ESP explanation from doc 04
> - Identity: explain that both sides must use matching identity strings (`EC2-vpn-aws` / `mikrotik-onprem-canal`) — a mismatch here is one of the most common tunnel failure causes

```
✍️ YOUR TURN: insert your Proposal + Identity config screenshots
```

## Step 4: IPsec Policy

> ✍️ YOUR TURN: Explain what a policy actually does — it defines *which traffic* gets encrypted (source 192.168.0.0/24 → destination 10.0.20.0/24), pointing to the peer and the proposal created above. Emphasize: **if a Policy doesn't match, IPsec never gets invoked at all** — traffic just goes out in the clear or gets dropped, depending on firewall rules. This is a great "what happens if X is misconfigured" answer to have ready.

```
✍️ YOUR TURN: insert your Policy config screenshot
```

## Firewall Rule

> ✍️ YOUR TURN: One line — why an explicit accept rule was needed for 192.168.0.0/24 → 10.0.20.0/24 even after IPsec was configured (IPsec policy alone doesn't bypass the firewall; RouterOS still evaluates firewall rules on the decrypted/pre-encrypted traffic).
