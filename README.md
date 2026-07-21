# Enterprise Network Infrastructure & Security Topology (GNS3)

## Project Overview & Introduction

Building a network topology is one thing, but making sure it can actually withstand real-world threats is a completely different story. 

In this project, I designed and built a multi-tier enterprise network inside GNS3 with a security-first mindset. Instead of just setting up connectivity and routing, I intentionally introduced multiple threat vectors into the infrastructure—placing dedicated attackers at every single layer (LAN access level, Distribution switches, and WAN perimeter). 

The main objective was to test and simulate common network attacks (like Rogue DHCP servers, ARP Spoofing, MAC Flooding, Unauthorized Routing Updates, and External WAN Probes), verify how the network behaves under threat, and then apply targeted Cisco hardening configurations to completely lock down the infrastructure.

---

## Network Topology

<img width="925" height="625" alt="Screenshot 2026-07-21 083211" src="https://github.com/user-attachments/assets/259255f7-4295-4dd1-92fa-c93fcbee4bb5" />


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
| **Rogue Host Connection** | Port Security (`protect`) | Unauthorized frames dropped; legitimate traffic unaffected. |
| **ARP Spoofing / MitM** | Dynamic ARP Inspection | Invalid ARP replies dropped via binding table mismatch. |
| **Rogue DHCP Server** | DHCP Snooping | DHCP Offer packets blocked on untrusted access ports. |

### 2. Perimeter & Control Plane Tests
| Test Scenario | Security Mechanism | Observed Behavior / Result |
| :--- | :--- | :--- |
| **External WAN Probe** | Extended ACLs | Drops logged on `BLOCK_ATTACK_PORT` rule. |
| **Untrusted Route Injection** | EIGRP MD5 Auth | Neighbor adjacency rejected without valid key match. |
| **WAN Packet Tracing** | PAT / NAT Overload | External nodes only see translated public interfaces. |

---

## Remote Access Credentials (Lab Environment)

```text
Domain Name: AhmedAlsaed.lab
Username:    Ahmed
Password:    cisco
Enable Sec:  class
