# Enterprise Network Infrastructure & Security Topology (GNS3)

## Project Overview & Introduction

Building a network topology is one thing, but making sure it can actually withstand real-world threats is a completely different story. 

In this project, I designed and built a multi-tier enterprise network inside GNS3 with a security-first mindset. Instead of just setting up connectivity and routing, I intentionally introduced multiple threat vectors into the infrastructure—placing dedicated attackers at every single layer (LAN access level, Distribution switches, and WAN perimeter). 

The main objective was to test and simulate common network attacks (like Rogue DHCP servers, ARP Spoofing, MAC Flooding, Unauthorized Routing Updates, and External WAN Probes), verify how the network behaves under threat, and then apply targeted Cisco hardening configurations to completely lock down the infrastructure.

---

## Network Topology

<img width="907" height="620" alt="Screenshot 2026-07-21 105841" src="https://github.com/user-attachments/assets/7b010bd0-1251-44bd-8c87-1259e7b7ce58" />



---

## Network Architecture
* **Access Layer (SW1 & SW2):** Enforces Layer 2 security controls for end-user ports (PC1-PC6) and handles threat mitigation for internal attackers (Attacker3 & Attacker4).
* **Distribution Layer (MUL1 & MUL2):** Manages Inter-VLAN routing, applies security policies per SVI, and authenticates dynamic routing peers.
* **Core / WAN Layer (R1, R2, SP):** Provides WAN routing, edge filtering against external threats (Attacker1 & Attacker2), and IP hiding via NAT/PAT.

---

## Implemented Security Controls

### 1. Access Layer Protection (SW1 & SW2)
* **DHCP Snooping:** Enables trusted path filtering across VLANs (Option 82 disabled) to block unauthorized rogue DHCP servers.
* **Dynamic ARP Inspection (DAI):** Cross-references ARP packets with the DHCP Snooping database to prevent ARP Spoofing / Man-in-the-Middle attacks.
* **Port Security (Protect Mode):** Enforces MAC address validation on host ports to instantly drop frames from unauthorized devices or MAC-spoofing attempts.
* **BPDU Guard:** Automatically err-disables switchports if rogue switches or unauthorized BPDUs are detected.

### 2. Distribution Layer Defense (MUL1 & MUL2)
* **VLAN Extended ACLs:** Filters and controls lateral traffic movement between VLANs via SVIs with an implicit deny posture.
* **DAI Trust Configuration:** Configures trusted uplink interfaces to ensure legitimate ARP traffic reaches default gateways without inspection drops.
* **EIGRP MD5 Authentication:** Encrypts dynamic routing updates using pre-shared MD5 keys (`EigrpPass2026`) to block untrusted neighbors or route injection.

### 3. Core / WAN Layer Defense (R1, R2 & SP)
* **Perimeter Extended ACLs (`BLOCK_ATTACK_PORT`):** Filters incoming WAN traffic to block external attacks originating from untrusted networks (Attacker1 & Attacker2).
* **Interface Management:** Administratively shuts down all unused physical and logical interfaces across all devices.
* **PAT / NAT Overload:** Translates internal IP spaces (`10.1.0.0/16` and `10.2.0.0/16`) to hide internal host architectures from the outside world.
* **EIGRP Control Plane Security:** Enforces MD5 authentication between Core Routers, Distribution Switches, and the SP edge.

### 4. Management & Administrative Protection
* **SSH (Secure Shell):** Encrypted VTY access across all network devices.
* **AAA Framework:** Configures local authentication and authorization rules for device administration.

---

## Security Testing & Attack Mitigation (PoC)

### 1. Access Layer Security Tests
| Test Scenario | Security Mechanism | Observed Behavior / Result |
| :--- | :--- | :--- |
| **Rogue Host Connection** | Port Security (`restrict`) | Unauthorized frames dropped; legitimate traffic unaffected. |
| **ARP Spoofing / MitM** | Dynamic ARP Inspection | Invalid ARP replies dropped via binding table mismatch. |
| **Rogue DHCP Server** | DHCP Snooping | DHCP Offer packets blocked on untrusted access ports. |

### 2. Perimeter & Control Plane Tests
| Test Scenario | Security Mechanism | Observed Behavior / Result |
| :--- | :--- | :--- |
| **External WAN Probe** | Extended ACLs | Drops logged on `BLOCK_ATTACK_PORT` rule. |
| **Untrusted Route Injection** | EIGRP MD5 Auth | Neighbor adjacency rejected without valid key match. |
| **WAN Packet Tracing** | PAT / NAT Overload | External nodes only see translated public interfaces. |

---

##  Real-World Attack Simulation & CLI Verification (PoC)

### 1. End-to-End Connectivity & NAT Verification
#### A. Ping & Trace from PC6 to Server1 (8.8.8.8):
<img width="1265" height="627" alt="image" src="https://github.com/user-attachments/assets/bbbe5467-dba8-430e-a095-d4fbc863fd30" />

#### B. Ping & Trace from PC1 at VLAN 10 LAN1 to PC4 at VLAN 20 LAN2:
<img width="1292" height="685" alt="image" src="https://github.com/user-attachments/assets/c868e655-cba1-422f-aa3b-e0da0c200a69" />

#### C. PAT / NAT Overload Translation Verification
#### Verifying that internal host IP addresses (VLAN 10 & VLAN 20) are successfully dynamically translated to the Public WAN interface IP when accessing external resources:
<img width="1175" height="641" alt="image" src="https://github.com/user-attachments/assets/c055fba0-27f2-4148-85ec-61e8a142ac53" />


### 2.Layer 2 Access Security (DHCP Snooping, Port Security)
#### A. Active DHCP Snooping Binding Table 
#### Verifying legitimate clients assigned via DHCP on trusted switchports:
<img width="1472" height="318" alt="image" src="https://github.com/user-attachments/assets/034eaecf-25c5-4f91-8414-ded2347345d5" />

#### B. Port Security Enforcement (Attacker3 Mitigation)
#### Verifying frame drops when an unauthorized device/MAC tries to flood or access Gi2/0 (Gi8) on Switch-1:
<img width="1441" height="551" alt="image" src="https://github.com/user-attachments/assets/74632657-2c01-4a94-a032-a42787ccc041" />

### 3. Dynamic Routing Security (EIGRP MD5 Authentication)
#### Verifying MD5 authenticated EIGRP neighbor adjacencies across Distribution switches and Core routers:
<img width="832" height="611" alt="image" src="https://github.com/user-attachments/assets/4b258678-10e7-4c81-8213-b2da7b9767be" />

### 4. Perimeter ACL & External Threat Drops
#### Verifying blocked probe attempts from external WAN nodes (Attacker1 & Attacker2) at router R1:
<img width="1113" height="357" alt="image" src="https://github.com/user-attachments/assets/4853fbdd-930d-4ad3-83e6-58ab12357a06" />








