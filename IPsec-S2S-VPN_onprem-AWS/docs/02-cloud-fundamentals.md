### 02 Cloud Fundamentals: Why Cloud, Why AWS

## What Is Cloud Computing Really?

The NIST definition is the most widely cited: cloud computing is a model that enables on-demand access to a shared pool of configurable computing resources (servers, storage, applications) that can be rapidly provisioned with minimal management effort.

In plain terms: instead of buying and maintaining physical servers, you rent computing capacity from a provider and pay for what you use.

## The Three most used Service Models

| Model | What the provider manages | What you manage | Example |
|---|---|---|---|
| **IaaS** | Physical infrastructure, virtualization | OS, runtime, applications | AWS EC2, Azure VMs |
| **PaaS** | Infrastructure + runtime | Just your application code | Google App Engine, Heroku |
| **SaaS** | Everything | Just your usage | Salesforce, Google Workspace |

This project sits squarely in **IaaS**: a self-managed EC2 instance running our own VPN software.

## The Five Essential Characteristics (NIST)

- On-demand self-service
- Broad network access
- Resource pooling
- Rapid elasticity
- Measured service

## Why I chose AWS Specifically

The same architecture would work on Azure or GCP with equivalent services swapped in (VPC → VNet, EC2 → Virtual Machine, etc.).

## ✍️ Cloud vs. On-Premise:
The architecture is based on a balance between on-premise infrastructure and cloud resources that many companies use.

Why not host everything on-premise at one subsidiary?
Because the other subsidiaries would depend heavily on that site's power supply, Internet connection, infrastructure availability, and administration. It would also create a central point of dependency and provide less flexibility when resources need to scale.

Why not move everything to the cloud?
Because some resources may need to remain locally hosted due to operational requirements, internal policies, or data sovereignty and regulatory considerations. Keeping these resources on-premise also allows each subsidiary to retain control over its local services.

Those are common reasons for opting for a hybrid model: local resources remain on-premise, while shared resources are centralized in the cloud and accessed securely through the VPN.
