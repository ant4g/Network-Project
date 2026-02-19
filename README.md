# Campus Network 3-tier (Core / Distribution / Access) — Cisco Packet Tracer

A campus network project built in a 3-tier architecture (Core / Distribution / Access) using Cisco Packet Tracer. The topology includes two buildings (A and B), redundancy at the Core/Distribution layers, and service segmentation: DATA / VOICE / Wi-Fi / MGMT / SERVERS.

![Topology](Topology.png)

---

## Features

Implemented:

- VLANs + trunking
- Inter-VLAN routing (SVIs)
- OSPF (area 0, routed /30 links)
- Centralized DHCP on the edge router + DHCP relay (ip helper-address)
- NAT: PAT (overload) + static NAT
- Rapid-PVST + root split (distribution layer)
- EtherChannel (PAgP/LACP)
- L2 security: DHCP Snooping, DAI, Port-Security, BPDU Guard (+ PortFast)
- VoIP: voice VLAN (Cisco 7960)
- WLAN basics: WLC + AP
- IPv6 on uplinks (primary/backup IPv6 default routes)

---

## Architecture

### Edge
- R1 (2911) — edge router with two ISP links (primary/backup)
- Services: DHCP, NAT (PAT + static NAT), OSPF + default route advertisement, IPv6 + IPv6 default routes (primary/backup)

### Core
- CSW1, CSW2 (3650) — L3 core (ip routing), routed /30 links, OSPF area 0

### Distribution
- Building A: DSW-A1, DSW-A2 (3650) — SVIs + inter-VLAN routing, OSPF uplinks to Core, Rapid-PVST root split, EtherChannel to Access
- Building B: DSW-B1, DSW-B2 (3650) — SVIs + inter-VLAN routing, OSPF uplinks to Core, Rapid-PVST root split, EtherChannel to Access

### Access
- Building A: ASW-A1/A2/A3 (2960) — L2 hardening, data/voice ports, trunk uplinks
- Building B: ASW-B1/B2 (2960) — L2 hardening, data/voice ports, server port in VLAN 30
- Additional: WLC1, LWAP1, SRV1, hosts (PC/Laptop), Cisco 7960 IP phones

---

## VLANs & IP plan

### Building A (SVIs on DSW-A1/DSW-A2)

| VLAN | Role | Subnet | DSW-A1 SVI | DSW-A2 SVI |
|---:|---|---|---|---|
| 10 | A-PC (DATA) | 10.1.0.0/24 | 10.1.0.2 | 10.1.0.3 |
| 20 | A-Phone (VOICE) | 10.2.0.0/24 | 10.2.0.2 | 10.2.0.3 |
| 40 | Wi-Fi | 10.6.0.0/24 | 10.6.0.2 | 10.6.0.3 |
| 99 | A-MGMT | 10.0.0.0/28 | 10.0.0.2 | 10.0.0.3 |

### Building B (SVIs on DSW-B1/DSW-B2)

| VLAN | Role | Subnet | DSW-B1 SVI | DSW-B2 SVI |
|---:|---|---|---|---|
| 10 | B-PC (DATA) | 10.3.0.0/24 | 10.3.0.2 | 10.3.0.3 |
| 20 | B-Phone (VOICE) | 10.4.0.0/24 | 10.4.0.2 | 10.4.0.3 |
| 30 | SERVERS | 10.5.0.0/24 | 10.5.0.2 | 10.5.0.3 |
| 99 | B-MGMT | 10.0.0.16/28 | 10.0.0.18 | 10.0.0.19 |

---

## DHCP

Centralized DHCP runs on R1. Distribution switches use DHCP relay via ip helper-address.

- DNS: 10.5.0.4
- Domain: MyITProject
- Option 43 in the A-MGMT pool (AP/WLC discovery scenario)

---

## NAT + Internet

- Two ISP links (primary/backup) + static default routes (backup AD=2)
- PAT (overload) using public pool 203.0.113.200–203.0.113.207/29
- Static NAT: 10.5.0.4 → 203.0.113.113

---

## STP & EtherChannel

### Rapid-PVST root split

Building A
- DSW-A1: root primary for VLAN 10, 99
- DSW-A2: root primary for VLAN 20, 40

Building B
- DSW-B1: root primary for VLAN 10, 99
- DSW-B2: root primary for VLAN 20, 30

### EtherChannel

- Building A: Po1 (PAgP desirable), trunk, native VLAN 1000, allowed 10/20/40/99
- Building B: Po1 (LACP active), trunk, native VLAN 1000, allowed 10/20/30/99

### Access hardening

- PortFast + BPDU Guard
- Trunk uplinks: switchport nonegotiate
- Example access ports:
  - ASW-A2/ASW-A3/ASW-B1: access VLAN 10 + voice VLAN 20 (PC + IP Phone)
  - ASW-B2: access VLAN 30 (server port)

---

## Quick verification (single checklist)

Below are example outputs captured from this project (R1) to confirm routing, reachability, and NAT behavior. Use them as a reference for what “good” looks like, then re-run the same commands in your lab.

### OSPF / routing (R1)

Command:
show ip ospf neighbor

Example output:
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.77         0   FULL/  -        00:00:34    10.0.0.34       GigabitEthernet0/0
10.0.0.78         0   FULL/  -        00:00:34    10.0.0.38       GigabitEthernet0/1

Command:
show ip route | include 0.0.0.0

Example output:
Gateway of last resort is 203.0.113.1 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 203.0.113.1

Command:
ping 10.5.0.4

Example output:
!!!!!
Success rate is 100 percent (5/5)

### Interfaces (R1)

Command:
show ip int brief

Example output:
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     10.0.0.33       YES manual up                    up 
GigabitEthernet0/1     10.0.0.37       YES manual up                    up 
GigabitEthernet0/0/0   203.0.113.2     YES manual up                    up 
GigabitEthernet0/1/0   203.0.113.6     YES manual up                    up 
Loopback0              10.0.0.76       YES manual up                    up 

### NAT (R1)

Step: generate traffic from an inside host (example: 10.1.0.14) to an upstream target (example: 203.0.113.1)
ping 203.0.113.1

Then verify NAT on R1:

Command:
show ip nat translations

Example output:
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.201:1   10.1.0.14:1        203.0.113.1:1      203.0.113.1:1
icmp 203.0.113.201:2   10.1.0.14:2        203.0.113.1:2      203.0.113.1:2
icmp 203.0.113.201:3   10.1.0.14:3        203.0.113.1:3      203.0.113.1:3
icmp 203.0.113.201:4   10.1.0.14:4        203.0.113.1:4      203.0.113.1:4
---  203.0.113.113     10.5.0.4           ---                ---

Command:
show ip nat statistics

Example output:
Total translations: 5 (1 static, 4 dynamic, 4 extended)
Outside Interfaces: GigabitEthernet0/0/0 , GigabitEthernet0/1/0
Inside Interfaces: GigabitEthernet0/0 , GigabitEthernet0/1
Hits: 4  Misses: 87
Dynamic mappings:
-- Inside Source
access-list 2 pool POOL1 refCount 4
 pool POOL1: netmask 255.255.255.248
       start 203.0.113.200 end 203.0.113.207
       type generic, total addresses 8 , allocated 1 (12%), misses 0

### DHCP (end-host)

PC -> Desktop -> IP Configuration -> DHCP

Expected:
- correct IP address from the right pool
- correct default gateway (SVI)
- DNS set to 10.5.0.4

### Layer 2 spot checks (Access/Distribution)

- show vlan brief
- show interfaces trunk
- show etherchannel summary
- show spanning-tree vlan 10
- show spanning-tree vlan 20
- show spanning-tree vlan 30
- show spanning-tree vlan 40

Expected:
- VLANs present and trunks carry the correct VLAN list
- Port-Channel up and member ports bundled
- STP root split matches the design per VLAN

---

## Limitations (Packet Tracer)

- HSRP/VRRP is not used due to Packet Tracer limitations/instability. Redundancy is provided by dual Core/Distribution, OSPF, and STP root split.
- ACL on SVI: Packet Tracer does not reliably accept/apply ip access-group on interface Vlan10 (SVI) in this topology, so enforcing an A -> B policy via an SVI ACL may not be possible.

---

## Author

Antoni Gąsiorowski

---

# Projekt sieci kampusowej 3-tier (Core / Distribution / Access) — Cisco Packet Tracer

Projekt sieci kampusowej w architekturze 3-tier (Core / Distribution / Access) zrealizowany w Cisco Packet Tracer. Topologia obejmuje dwa budynki (A i B), redundancję w warstwie Core/Distribution oraz segmentację usług: DATA / VOICE / Wi-Fi / MGMT / SERVERS.

![Topologia](Topology.png)

---

## Zakres funkcjonalny

Zaimplementowane elementy:

- VLAN + trunking
- Inter-VLAN routing (SVI)
- OSPF (area 0, routed links /30)
- Centralny DHCP na routerze brzegowym + DHCP relay (ip helper-address)
- NAT: PAT (overload) + static NAT
- Rapid-PVST + root split (distribution)
- EtherChannel (PAgP/LACP)
- L2 security: DHCP Snooping, DAI, Port-Security, BPDU Guard (+ PortFast)
- VoIP: voice VLAN (Cisco 7960)
- WLAN: podstawy (WLC + AP)
- IPv6: na uplinkach (z trasami domyślnymi primary/backup)

---

## Architektura

### Edge
- R1 (2911) — router brzegowy z dwoma łączami do ISP (primary/backup)
- Usługi: DHCP, NAT (PAT + static NAT), OSPF + default route, IPv6 + default IPv6 (primary/backup)

### Core
- CSW1, CSW2 (3650) — L3 core (ip routing), routed links /30, OSPF area 0

### Distribution
- Budynek A: DSW-A1, DSW-A2 (3650) — SVI + inter-VLAN routing, OSPF uplinki do Core, Rapid-PVST root split, EtherChannel do Access
- Budynek B: DSW-B1, DSW-B2 (3650) — SVI + inter-VLAN routing, OSPF uplinki do Core, Rapid-PVST root split, EtherChannel do Access

### Access
- Budynek A: ASW-A1/A2/A3 (2960) — hardening L2, porty data/voice, uplinki trunk
- Budynek B: ASW-B1/B2 (2960) — hardening L2, porty data/voice, port serwerowy w VLAN 30
- Dodatkowo: WLC1, LWAP1, SRV1, hosty (PC/Laptop), telefony Cisco 7960

---

## VLAN & plan adresacji

### Budynek A (SVI na DSW-A1/DSW-A2)

| VLAN | Rola | Subnet | DSW-A1 SVI | DSW-A2 SVI |
|---:|---|---|---|---|
| 10 | A-PC (DATA) | 10.1.0.0/24 | 10.1.0.2 | 10.1.0.3 |
| 20 | A-Phone (VOICE) | 10.2.0.0/24 | 10.2.0.2 | 10.2.0.3 |
| 40 | Wi-Fi | 10.6.0.0/24 | 10.6.0.2 | 10.6.0.3 |
| 99 | A-MGMT | 10.0.0.0/28 | 10.0.0.2 | 10.0.0.3 |

### Budynek B (SVI na DSW-B1/DSW-B2)

| VLAN | Rola | Subnet | DSW-B1 SVI | DSW-B2 SVI |
|---:|---|---|---|---|
| 10 | B-PC (DATA) | 10.3.0.0/24 | 10.3.0.2 | 10.3.0.3 |
| 20 | B-Phone (VOICE) | 10.4.0.0/24 | 10.4.0.2 | 10.4.0.3 |
| 30 | SERVERS | 10.5.0.0/24 | 10.5.0.2 | 10.5.0.3 |
| 99 | B-MGMT | 10.0.0.16/28 | 10.0.0.18 | 10.0.0.19 |

---

## DHCP

Centralny DHCP działa na R1. Przełączniki Distribution wykonują DHCP relay przez ip helper-address.

- DNS: 10.5.0.4
- Domena: MyITProject
- Option 43 w puli A-MGMT (AP/WLC discovery)

---

## NAT + Internet

- Dwa łącza do ISP (primary/backup) + statyczne trasy domyślne (backup AD=2)
- PAT (overload) przez pulę publiczną 203.0.113.200–203.0.113.207/29
- Static NAT: 10.5.0.4 → 203.0.113.113

---

## STP & EtherChannel

### Rapid-PVST root split

Budynek A
- DSW-A1: root primary dla VLAN 10, 99
- DSW-A2: root primary dla VLAN 20, 40

Budynek B
- DSW-B1: root primary dla VLAN 10, 99
- DSW-B2: root primary dla VLAN 20, 30

### EtherChannel

- Budynek A: Po1 (PAgP desirable), trunk, native VLAN 1000, allowed 10/20/40/99
- Budynek B: Po1 (LACP active), trunk, native VLAN 1000, allowed 10/20/30/99

### Hardening (Access)

- PortFast + BPDU Guard
- Uplinki trunk: switchport nonegotiate
- Porty przykładowe:
  - ASW-A2/ASW-A3/ASW-B1: access VLAN 10 + voice VLAN 20 (PC + IP Phone)
  - ASW-B2: access VLAN 30 (port serwerowy)

---

## Ograniczenia (Packet Tracer)

- Brak użycia HSRP/VRRP ze względu na ograniczenia/niestabilność wsparcia w Packet Tracer. Redundancja: podwójny Core/Distribution, OSPF oraz STP root split.


---

## Autor

Antoni Gąsiorowski
