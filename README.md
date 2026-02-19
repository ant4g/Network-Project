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

## Test plan

L2 / VLAN / EtherChannel
- show vlan brief
- show interfaces trunk
- show etherchannel summary

STP
- show spanning-tree vlan 10
- show spanning-tree vlan 20
- show spanning-tree vlan 30
- show spanning-tree vlan 40

OSPF / routing
- show ip ospf neighbor (Core/Distribution; FULL)
- show ip route (check default route 0.0.0.0/0 advertised by R1)

DHCP
- Hosts in VLAN 10/30 receive addresses from correct pools + DNS 10.5.0.4

E2E
- Inter-VLAN: PC (VLAN10) ↔ SRV1 (VLAN30)
- A ↔ B: ICMP 10.1.0.0/24 → 10.3.0.0/24
- Internet: NAT + show ip nat translations on R1

---

## Limitations (Packet Tracer)

- HSRP/VRRP is not used due to Packet Tracer limitations/instability. Redundancy is provided by dual Core/Distribution, OSPF, and STP root split.

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
- Dodatkowo: Option 43 w puli A-MGMT (AP/WLC discovery)

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

## Plan testów

L2 / VLAN / EtherChannel
- show vlan brief
- show interfaces trunk
- show etherchannel summary

STP
- show spanning-tree vlan 10
- show spanning-tree vlan 20
- show spanning-tree vlan 30
- show spanning-tree vlan 40

OSPF / routing
- show ip ospf neighbor (Core/Distribution; stan FULL)
- show ip route (trasa domyślna 0.0.0.0/0 rozgłaszana z R1)

DHCP
- Hosty w VLAN 10/30 dostają adresy z właściwych pul + DNS 10.5.0.4

E2E
- Inter-VLAN: PC (VLAN10) ↔ SRV1 (VLAN30)
- A ↔ B: ICMP 10.1.0.0/24 → 10.3.0.0/24
- Internet: NAT + show ip nat translations na R1

---

## Ograniczenia (Packet Tracer)

- Brak użycia HSRP/VRRP ze względu na ograniczenia/niestabilność wsparcia w Packet Tracer. Redundancja: podwójny Core/Distribution, OSPF oraz STP root split.

---

## Autor

Antoni Gąsiorowski
