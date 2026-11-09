# Multi-site OSPF Network

## Overview

This project implements a multi-site enterprise network connecting a **Headquarters (Agency A)** and a **Branch Office (Agency B)** through a dedicated inter-site backbone with redundant Internet edges in a simulated environment.

Agency A implements a **fully redundant three-tier network architecture** composed of Core, Distribution, and Access layers to maximize availability and minimize single points of failure. Agency B uses a **collapsed-core architecture**, combining core and distribution functions to provide a more cost-effective design suitable for a smaller site.

Internal departmental networks are segmented using dedicated **VLANs**, with each VLAN assigned a `/27` IPv4 subnet from the `172.16.0.0/16` private address space. Public-facing services, including **HTTPS and DNS servers**, are isolated within a dedicated **DMZ (Demilitarized Zone)** to limit the exposure of internal resources.

Point-to-point routed links use `/30` subnets from the `10.0.0.0/8` private address space, while simulated Internet-facing links use public IPv4 addressing.

The routing architecture relies primarily on **OSPF** for dynamic internal routing, while **BGP** is used at the Internet edge.

## Objectives

The main objectives of this architecture are:

* **High Availability:** Provide network redundancy at the headquarters through dual-homed distribution switches, redundant paths, HSRP, and link aggregation.
* **Perimeter Security:** Isolate public-facing services within a dedicated DMZ and control traffic between security zones using a Zone-Based Firewall.
* **Network Segmentation:** Separate departmental traffic into dedicated VLANs to reduce broadcast domains and improve traffic isolation.
* **Dynamic Routing:** Use OSPF to automatically exchange internal routes and provide dynamic path selection across the multi-site infrastructure.
* **Internet Edge Redundancy:** Simulate redundant Internet connectivity using two edge routers.
* **Cost-Optimized Scalability:** Deploy a full three-tier architecture at the headquarters while using a collapsed-core model at the branch to reduce infrastructure complexity and hardware requirements.
* **Efficient IPv4 Allocation:** Apply VLSM to allocate `/27` subnets to departmental networks and `/30` subnets to point-to-point transit links, minimizing IPv4 address waste.

## Technologies

The architecture uses the following networking technologies and protocols:

* **VLAN** : Logical segmentation of the Layer 2 network into separate broadcast domains.
* **IPv4 & VLSM** : Structured and efficient IPv4 address allocation.
* **Inter-VLAN Routing** : Layer 3 communication between departmental VLANs.
* **DHCP** : Automatic assignment of IP configuration to end devices.
* **OSPF** : Dynamic Interior Gateway Protocol used for routing within the enterprise network.
* **HSRP** : First-hop redundancy protocol providing a highly available default gateway.
* **LACP** : Link Aggregation Control Protocol used to combine multiple physical links into a logical link.
* **BGP** : Exterior Gateway Protocol used to simulate routing between the enterprise network and external networks.
* **Zone-Based Firewall** : Stateful traffic filtering between defined security zones.
* **NAT** : Translation between private and public IPv4 address spaces.


### Agency A : Headquarters

Agency A uses a **three-tier hierarchical architecture** also known as Cisco Borderless network architecture in cisco gear:

* **Core Layer:** Provides high-speed Layer 3 connectivity and redundant paths between major network segments.
* **Distribution Layer:** Aggregates access switches, provides inter-VLAN routing, implements redundancy, and applies network policies.
* **Access Layer:** Connects end-user devices and provides VLAN-based segmentation.

Redundant uplinks between the distribution and access layers improve resilience and provide alternative forwarding paths in case of link or device failure.

### Agency B : Branch Office

Agency B uses a **collapsed-core architecture**, where core and distribution functions are consolidated.

This approach reduces the number of required network devices while maintaining Layer 3 routing, VLAN segmentation, and connectivity with the headquarters.

### Inter-Site Connectivity

Agency A and Agency B are interconnected through a routed backbone.

OSPF dynamically exchanges internal routes between the sites, allowing traffic to use available paths without relying exclusively on static routes.

### Internet Edge and DMZ

The Internet edge provides connectivity between the enterprise network and simulated external networks.

Public-facing services are placed in a dedicated DMZ rather than directly inside the internal network. NAT and firewall policies control the communication between the Internet, DMZ, and internal networks.

## Documentation

* **[Design](design/)** : Network topology diagrams and architectural documentation.
* **[Addressing](addressing/)** : IPv4 addressing plan, VLANs, subnets, and point-to-point links.
* **[Configurations](configurations/)** : Device configurations for routers, switches, and security components.
* **[Validation](validation/)** : Connectivity tests, routing verification, redundancy tests, and troubleshooting results.
* **[Packet Tracer Files](packet-tracer/)** : Cisco Packet Tracer `.pkt` lab files.
* **[Changelog](CHANGELOG.md)** : Project changes and configuration history.

## Limitations

This project is implemented in a **Cisco Packet Tracer simulation environment**. Consequently, some behaviors and capabilities are simplified or not fully represented. Packet tracer also struggle when the lab get big. As a result, some commands can be successfully applied, but have no effect although they appear in the running configuration. I had this issue a couple of times in some personal labs and also with JeremysIT Lab's CCNA mega lab. The command `ip helper-address 10.0.0.76` completely refused, thus preventing clients to lease addressed automaticaly.

Key limitations include:

* Headquarters internet connectivity depends on just one edge router and the wan link (between the two core switches) which leads to the other edge router. If one edge router falls, traffic takes more time to reach to other gateway. So an improvement of this topology would be to connect each core switch to both edge routers. The automatic failover will be handled be default floating static routes. Thus providing three paths to the internet. Or originate the default on both edge routers rather than configure statics on SW-C1/SW-C2: the two campus halves then each get a working default from their nearest edge router, and if WAN A fails, OSPF removes Router-E1's default and traffic converges on Router-E2 across Po1 without any floating-static AD tuning.
* There is no network monitoring protocol configured. The server in Headquarter B could be configured for it. So there is no control over who entered which command on which device at what time (by the way NTP is also no configured)
* SSH administration is not configured and is quite a limitation in an enterprise network. 
* The switchports at the access layer are not secured (port security isn't enabled). DAI and DHCP snooping would also be a great security improvement
* SW-A3 is single-homed to one trunk only, so it has no path redundancy at Layer 2
* VLAN 50/60 routes hinge on SW-AD1, which is a single switch becoming a SPO
* SW-C1 is a single point of failure. Both distribution switches reach both edge routers only through SW-C1

## Future Improvements

The architecture could be further developed by introducing:

* **Migration to a virtualized or emulated environment**, such as EVE-NG or GNS3, to validate behavior closer to real Cisco IOS/IOS-XE environments.
* **Enhanced network monitoring and observability** using SNMP, Syslog
* **Implementation of a defense-in-depth security architecture** by deploying a next-generation firewall such as FortiGate right after the edge routers to  strengthen security and provide intrusion prevention
* **IPv6 deployment** alongside IPv4 to support dual-stack operation 
* **High-availability improvements** 
* **Centralized configuration and backup management** to improve operational reliability and simplify recovery.
