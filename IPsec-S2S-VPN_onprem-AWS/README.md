# Secure Site-to-Site Interconnection: On-Premise Network to AWS Cloud

> A deep dive into designing, implementing, and validating an IPsec/IKEv2 VPN tunnel between an enterprise LAN (MikroTik) and an AWS VPC (strongSwan on EC2), built during my engineering internship, documented here as a teaching resource.

---

## Why This Repo Exists

This isn't just a copy of my end-of-studies report. It's a rewritten, explanation-first walkthrough of the same project, built so that someone unfamiliar with IPsec, AWS networking, or MikroTik can follow the reasoning behind every decision, not just the end configuration.

---

## Architecture

<img width="1166" height="666" alt="image" src="https://github.com/user-attachments/assets/f0a28305-5bb6-46ae-8404-bda991f09d07" />

---

## Table of Contents

| Doc | Topic |
|---|---|
| [01 Context](docs/01-context.md) | The business problem this project solves |
| [02 Cloud Fundamentals](docs/02-cloud-fundamentals.md) | Why cloud |
| [03 VPN Landscape](docs/03-vpns.md) | VPN types & protocols compared |
| [04 IPsec Deep Dive](docs/04-ipsec-ikev2.md) | IPsec/IKEv2 explained in depth |
| [05 Architecture](docs/05-Architecture&Design.md) | Design decisions |
| [06 AWS Setup](docs/06-aws-setup.md) | VPC, EC2, strongSwan configuration |
| [07 MikroTik Setup](docs/07-mikrotik-config.md) | Router-side configuration |
| [08 Testing & Validation](docs/08-testing.md) | Proving it actually works |
| [09 Lessons & Future Work](docs/09-future-work.md) | What's next: Terraform, HA, more |

---

## Tech Stack

`IPsec` `IKEv2` `strongSwan` `MikroTik RouterOS` `AWS VPC` `EC2` `Security Groups` `Elastic IP` `Wireshark`

---
