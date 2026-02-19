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

## Limitations (Packet Tracer)

- Brak użycia HSRP/VRRP ze względu na ograniczenia/niestabilność wsparcia w Packet Tracer. Redundancja: podwójny Core/Distribution, OSPF oraz STP root split.

---

## Autor

Antoni Gąsiorowski
