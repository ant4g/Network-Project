# Network Security Lab (Packet Tracer): IPsec S2S + OSPF (HMAC) + VLAN Segmentation

Author: Antoni Gąsiorowski

## Overview
This repository contains a Packet Tracer lab demonstrating practical network security controls in a small routed environment:
- Site-to-Site IPsec VPN protecting traffic between Site A (192.168.1.0/24) and Site B VLAN 3 (172.16.3.0/24) using AES-256, ESP-AES-256/ESP-SHA-HMAC, and PFS group5
- OSPF Area 0 for dynamic routing across WAN links
- OSPF adjacency authentication (key-chain with HMAC-SHA-256) on the transit link
- VLAN segmentation at Site B using router-on-a-stick on R3:
  - VLAN 3: 172.16.3.0/24
  - VLAN 33: 172.16.33.0/24
- Access-layer hardening (where configured): PortFast + BPDU Guard, port-security (sticky MAC), trunk hardening (native VLAN 99, DTP disabled)

## Topology
![Packet Tracer topology](Topology_PK.png)

High-level structure:
- R1 (Site A) ↔ R2 (Transit) ↔ R3 (Site B)
- Site A LAN behind R1: 192.168.1.0/24
- Site B VLANs behind R3: 172.16.3.0/24 (VLAN 3) and 172.16.33.0/24 (VLAN 33)

## Key Security Controls
### IPsec Site-to-Site VPN
- IKE policy: AES-256, pre-shared key, DH group 5
- IPsec transform-set: ESP-AES-256 + ESP-SHA-HMAC
- PFS: group5
- Interesting traffic is scoped with extended ACLs:
  - 192.168.1.0/24 ↔ 172.16.3.0/24

### OSPF Security (Control Plane)
- OSPF is enabled across WAN links (Area 0)
- Key-chain authentication (HMAC-SHA-256) is applied on the R2–R3 adjacency to reduce the risk of routing protocol spoofing/poisoning

### Segmentation (Data Plane)
- Two L2 segments at Site B (VLAN 3 and VLAN 33) terminate on R3 802.1Q subinterfaces
- Switch management SVIs are placed in VLAN 3 (172.16.3.0/24)

## Limitations (Packet Tracer)
Packet Tracer does not fully match feature parity of real IOS/CSR images (e.g., some Zone-Based Firewall behaviors on 802.1Q subinterfaces). This lab focuses on controls that are consistently demonstrable in PT: IPsec, OSPF authentication, VLAN segmentation, and access-layer hardening.

---

# Laboratorium Network Security (Packet Tracer): IPsec S2S + OSPF (HMAC) + Segmentacja VLAN

Autor: Antoni Gąsiorowski

## Opis
Repozytorium zawiera laboratorium Packet Tracer pokazujące praktyczne mechanizmy bezpieczeństwa w małej sieci routowanej:
- Tunel Site-to-Site IPsec chroniący ruch między Site A (192.168.1.0/24) a Site B VLAN 3 (172.16.3.0/24) z użyciem AES-256, ESP-AES-256/ESP-SHA-HMAC oraz PFS group5
- OSPF Area 0 do routingu dynamicznego na łączach WAN
- Uwierzytelnianie sąsiedztwa OSPF (key-chain z HMAC-SHA-256) na łączu tranzytowym
- Segmentacja VLAN w Site B z router-on-a-stick na R3:
  - VLAN 3: 172.16.3.0/24
  - VLAN 33: 172.16.33.0/24
- Utwardzenie warstwy dostępowej (tam gdzie skonfigurowane): PortFast + BPDU Guard, port-security (sticky MAC), hardening trunków (native VLAN 99, wyłączone DTP)

## Topologia
![Topologia Packet Tracer](Topology_PK.png)

Struktura (wysoki poziom):
- R1 (Site A) ↔ R2 (Tranzyt) ↔ R3 (Site B)
- LAN w Site A za R1: 192.168.1.0/24
- VLANy w Site B za R3: 172.16.3.0/24 (VLAN 3) oraz 172.16.33.0/24 (VLAN 33)

## Kluczowe mechanizmy bezpieczeństwa
### IPsec Site-to-Site
- Polityka IKE: AES-256, pre-shared key, DH group 5
- Transform-set IPsec: ESP-AES-256 + ESP-SHA-HMAC
- PFS: group5
- Zakres szyfrowanego ruchu (interesting traffic) ograniczony ACL:
  - 192.168.1.0/24 ↔ 172.16.3.0/24

### Bezpieczeństwo OSPF (control plane)
- OSPF działa na łączach WAN (Area 0)
- Key-chain authentication (HMAC-SHA-256) na sąsiedztwie R2–R3 ogranicza ryzyko spoofingu/poisoningu protokołu routingu

### Segmentacja (data plane)
- Dwa segmenty L2 w Site B (VLAN 3 i VLAN 33) terminują na subinterfejsach 802.1Q na R3
- Zarządzanie switchami (SVI) jest w VLAN 3 (172.16.3.0/24)

## Ograniczenia (Packet Tracer)
Packet Tracer nie ma pełnej zgodności funkcji z realnymi obrazami IOS/CSR (np. ograniczenia Zone-Based Firewall na subinterfejsach 802.1Q). Laboratorium koncentruje się na mechanizmach, które da się konsekwentnie pokazać w PT: IPsec, uwierzytelnianie OSPF, segmentacja VLAN i hardening warstwy dostępowej.
