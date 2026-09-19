## Known Limitations of the Current Design

> - **Single point of failure**: one EC2 instance, one Availability Zone, one tunnel
> - **No real-time monitoring** of tunnel health/performance
> - **Full dependency on internet availability** at the on-premise site, cause yeah it's a S2S VPN over internet, wyw🤷‍♂️

## Future Work #1: High-Availability Multi-AZ Redesign

> - Two IPsec tunnels to two EC2 instances in different Availability Zones
> - Auto Scaling Group + Lambda/EventBridge to reassign a pre-created ENI and Elastic IP if an instance is replaced, keeping the routing of the private subnet static
> - MikroTik maintains both peers simultaneously; failover is route-distance/DPD-based, with only one tunnel actually carrying traffic at a time

However there will be a problem about TCP connections break on failover this is a VPN-inherent limitation, not something this design fixes

## Future Work #2: Infrastructure as Code with Terraform

> This architecture can be rebuild (VPC, subnets, EC2, Security Groups, Elastic IP, strongSwan bootstrap) as Terraform modules, to make the deployment reproducible

## Future Work #3: Stronger Authentication at Scale

> PSK is fine for two gateways but becomes a management burden as more sites/VPCs join, and X.509 certificate-based authentication would be the natural next step at that scale.
