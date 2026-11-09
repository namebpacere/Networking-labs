# Routing Plan



\---

## 1\. Subnet inventory

### 1.1 Campus VLAN subnets `172.16.0.0/24` block

|VLAN|Name|Subnet|Mask|Usable range|Broadcast|Count|
|-|-|-|-|-|-|-|
|10|Comptability|172.16.0.0/27|255.255.255.224|172.16.0.1 – 172.16.0.30|172.16.0.31|30|
|20|Financy|172.16.0.32/27 |255.255.255.224|172.16.0.33 – 172.16.0.62|172.16.0.63|30|
|30|ServiceClient|172.16.0.64/27|255.255.255.224|172.16.0.65 – 172.16.0.94|172.16.0.95|30|
|40|IT|172.16.0.96/27|255.255.255.224|172.16.0.97 – 172.16.0.126|172.16.0.127|30|
|50|SecurityDepartment|172.16.0.128/27|255.255.255.224|172.16.0.129 – 172.16.0.158|172.16.0.159|30|
|60|ResearchDepartment|172.16.0.160/27|255.255.255.224|172.16.0.161 – 172.16.0.190|172.16.0.191|30|

### 

### 1.2 Point-to-point transit links `10.0.0.0` block

|#|Subnet|Mask|Usable|Endpoint A|Endpoint B|
|-|-|-|-|-|-|
|T1|10.0.0.0/30|255.255.255.252|10.0.0.1 – 10.0.0.2|SW-D1 G1/0/4|10.0.0.1|
|T2|10.0.0.4/30|255.255.255.252|10.0.0.5 – 10.0.0.6|SW-D2 G1/0/3|10.0.0.6|
|T3|10.0.0.8/30|255.255.255.252|10.0.0.9 – 10.0.0.10|Router-E1 Fa0/0|10.0.0.9|
|T4|10.0.0.12/30|255.255.255.252|10.0.0.13 – 10.0.0.14|Router-E2 Fa0/0|10.0.0.14|
|T5|10.0.0.16/30|255.255.255.252|10.0.0.17 – 10.0.0.18|SW-AD1 G1/0/24|10.0.0.17|
|T6|10.0.0.20/30|255.255.255.252|10.0.0.21 – 10.0.0.22|SW-C2 Po1 (G1/0/4-6)|10.0.0.21|



\---

## 2\. Device inventory and interface addressing

### 2.1 ISPRouter Router 1841

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|FastEthernet 0/0|192.168.0.254|192.168.0.0/24|TO THE SIMULATED INTERNET NETWORK|
|Serial 0/0/0|203.0.113.1|203.0.113.0/24|TO ROUTER-E1|
|Serial 0/0/1|203.0.114.1|203.0.114.0/24|TO Router-E2|

### 2.2 Router-E1 Router 1841 

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|FastEthernet 0/0|10.0.0.9|10.0.0.8/30|TO THE INTERN NETWORK|
|FastEthernet 0/1|192.168.1.253|192.168.1.0/24|TO THE DMZ SWITCH|
|Serial 0/0/0|203.0.113.2|203.0.113.0/24|TO THE INTERNET|

### 

### 2.3 Router-E2 Router 1841 

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|FastEthernet 0/0|10.0.0.14|10.0.0.12/30|TO THE INTERN NETWORK|
|FastEthernet 0/1|192.168.1.252|192.168.1.0/24|TO THE DMZ SWITCH|
|Serial 0/0/0|203.0.114.2|203.0.114.0/24 |TO THE INTERNET|





### 2.4 SW-C1 — Multilayer Switch 3650-24APS

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|GigabitEthernet 1/0/1|10.0.0.2|10.0.0.0/30|TO SW-D1|
|GigabitEthernet 1/0/2|10.0.0.5|10.0.0.4/30|TO SW-D2|
|GigabitEthernet 1/0/3|10.0.0.10|10.0.0.8/30|TO ROUTER-E1|
|GigabitEthernet 1/0/4-6|10.0.0.22|10.0.0.20/30|PORTCHANNEL 1 TO SW-C2|

### 

### 2.5 SW-C2 Multilayer Switch 3650-24APS

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|GigabitEthernet 1/0/1|10.0.0.13|10.0.0.12/30|TO ROUTER-E2|
|GigabitEthernet 1/0/2|10.0.0.18|10.0.0.16/30|TO SW-AD1|
|GigabitEthernet 1/0/4-6|10.0.0.21|10.0.0.20/30|PORTCHANNEL 1 TO SW-C1|

### 

### 2.6 SW-D1 Multilayer Switch 3650-24APS

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|GigabitEthernet 1/0/1|—|—|TO SW-A1|
|GigabitEthernet 1/0/2|—|—|TO SW-A2|
|GigabitEthernet 1/0/3|—|—|TO SW-A3|
|GigabitEthernet 1/0/4|10.0.0.1|10.0.0.0/30|TO SW-C1|

### 

### 2.7 SW-D2 Multilayer Switch 3650-24APS

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|GigabitEthernet 1/0/1|—|—|TO SW-A1|
|GigabitEthernet 1/0/2|—|—|TO SW-A2|
|GigabitEthernet 1/0/3|10.0.0.6|10.0.0.4/30|TO SW-C1|

### 

### 2.8 SW-AD1 Multilayer Switch 3650-24APS

|Interface|IP address|Subnet|Description|
|-|-|-|-|
|GigabitEthernet 1/0/24|10.0.0.17|10.0.0.16/30|TO SW-C2|
|GigabitEthernet 1/0/23|172.16.1.1|172.16.1.0/24|TO DHCP SERVER|
|GigabitEthernet 1/0/1-4|172.16.0.129|172.16.0.128/27|VLAN 50|
|GigabitEthernet 1/0/5-8|172.16.0.161|172.16.0.160/27|VLAN 60|

### 

### 2.9 Access switches Switch 2950-24 (Layer 2 only)

|Device|Interface|IP|Subnet|Description|
|-|-|-|-|-|
|SW-A1|FastEthernet 0/21|—|TRUNK|TO SW-C1|
|SW-A1|FastEthernet 0/22|—|TRUNK|TO SW-C2|
|SW-A1|FastEthernet 0/1-10|—|172.16.0.0/27|VLAN 10|
|SW-A1|FastEthernet 0/11-20|—|172.16.0.32/27|VLAN 20|
|SW-A2|FastEthernet 0/21|—|TRUNK|TO SW-C2|
|SW-A2|FastEthernet 0/22|—|TRUNK|TO SW-C1|
|SW-A2|FastEthernet 0/1-10|—|172.16.0.64/27|VLAN 30|
|SW-A2|FastEthernet 0/11-20|—|172.16.0.96/27|VLAN 40|
|SW-A3|FastEthernet 0/21|—|TRUNK|TO SW-C1|
|SW-A3|FastEthernet 0/1-10|—|172.16.0.32/27|VLAN 20|
|SW-A3|FastEthernet 0/11-20|—|172.16.0.96/27|VLAN 40|

No management IP for SW-A1, SW-A2 or SW-A3.

\---

## 

## 3\. Routing design

The natural design for this topology is a **single OSPF process 1, area 0** carrying the transit links, with VLAN SVIs advertised but passive.

|Device|Router-ID|OSPF-enabled networks and wildcard|OSPF-enabled interfaces but Passive|
|-|-|-|-|
|SW-C1|10.0.0.2|10.0.0.0 0.0.0.3 \| 10.0.0.4 0.0.0.3 \| 10.0.0.20 0.0.0.3|—|
|SW-C2|10.0.0.13|10.0.0.12 0.0.0.3 \| 10.0.0.16 0.0.0.3 \| 10.0.0.20 0.0.0.3|—|
|SW-D1|10.0.0.1|10.0.0.0 0.0.0.3|Vlan10, Vlan20, Vlan30, Vlan40|
|SW-D2|10.0.0.6|10.0.0.4 0.0.0.3|Vlan10, Vlan20, Vlan30, Vlan40|
|SW-AD1|10.0.0.17|10.0.0.16 0.0.0.3|Vlan50, Vlan60, 172.16.1.0|
|Router-E1|10.0.0.9|10.0.0.8 0.0.0.3|—|
|Router-E2|10.0.0.14|10.0.0.12 0.0.0.3|—|



```cisco
! SW-D1 pattern for both distribution switches
router ospf 1
 router-id 10.0.0.1
 network 10.0.0.0 0.0.0.3 area 0
 passive-interface Vlan10
 passive-interface Vlan20
 passive-interface Vlan30
 passive-interface Vlan40
!
interface GigabitEthernet1/0/4
 ip ospf network point-to-point
```

Two design notes. 

* First, `ip ospf network point-to-point` on all six `/30` links suppresses DR/BDR election which is correct for point-to-point, and it makes adjacency verification unambiguous (`FULL/ -`). 
* Second, because all six transit links sit inside `10.0.0.0/27`, a single `network 10.0.0.0 0.0.0.63 area 0` on SW-C1 and SW-C2 would cover them, but on SW-D1/SW-D2 it would also capture nothing else, so per-link statements are safer and self-documenting.





### Static and default routes (future improvement)

|Device|Route|Purpose|
|-|-|-|
|Router-E1|`ip route 0.0.0.0 0.0.0.0 203.0.113.1`|Upstream to simulated Internet|
|Router-E2|`ip route 0.0.0.0 0.0.0.0 203.0.114.1`|Upstream to simulated Internet|
|Router-E1|`default-information originate` (in OSPF)|Injects default into area 0|
|Router-E2|`default-information originate` (in OSPF)|Second, equal-cost default|
|ISPRouter|`ip route 172.16.0.0 255.255.255.0 203.0.113.2`|Return path to VLANs via WAN A|
|ISPRouter|`ip route 172.16.0.0 255.255.255.0 203.0.114.2`|Return path to VLANs via WAN B|
|ISPRouter|`ip route 192.168.1.0 255.255.255.0 203.0.113.2`|Return path to DMZ|
|ISPRouter|`ip route 192.168.1.0 255.255.255.0 203.0.114.2`|Return path to DMZ|



**Summarisation:** `172.16.0.0/24` is a single clean summary for all six VLANs, and `10.0.0.0/24` for the transit block. Because the address plan is this well-behaved, OSPF needs no `area range` — the prefixes are already summarisable by hand at any boundary.



\---

## 4\. First-hop redundancy status: HSRP

|VLAN|Subnet|VIP|Active|Standby|
|-|-|-|-|-|
|10|172.16.0.0/27|172.16.0.1|SW-D1 (.3)|SW-D2 (.2)|
|20|172.16.0.32/27|172.16.0.33|SW-D1 (.35)|SW-D2 (.34)|
|30|172.16.0.64/27|172.16.0.65|SW-D2 (.67)|SW-D1 (.66)|
|40|172.16.0.96/27|172.16.0.97|SW-D2 (.99)|SW-D1 (.98)|

VLANs 50 and 60 have no redundancy by design (single SVI on SW-AD1).

&#x20;---



