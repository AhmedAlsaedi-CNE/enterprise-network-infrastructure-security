# Enterprise Network Infrastructure & Security Topology (GNS3)

A comprehensive multi-tier enterprise network designed, implemented, and secured using GNS3.

---

## 🗺️ Network Topology

<img width="925" height="625" alt="Screenshot 2026-07-21 083211" src="https://github.com/user-attachments/assets/1e10fb31-6ac2-4261-873e-93d87a47cb07" />


---

## 📐 Network Architecture
* **Access Layer:** Layer 2 security enforcement and edge device connectivity.
* **Distribution Layer:** Inter-VLAN routing with EIGRP and security filtering via SVIs.
* **Core / WAN Layer:** Edge routing, perimeter defense, and Internet/WAN connectivity.

---

## 🔐 Implemented Security Features

### 1. Access Layer Security (SW1 & SW2)
* **DHCP Snooping:** Configured across VLANs with trusted uplinks and Option 82 disabled.
* **Dynamic ARP Inspection (DAI):** Validates ARP packets against DHCP snooping database to prevent ARP Spoofing/MitM.
* **Port Security (Protect Mode):** Enforces MAC address validation and drops unauthorized frame attempts.
* **BPDU Guard:** Instantly err-disables ports upon receiving unauthorized BPDUs.

### 2. Distribution Layer Security (MUL1 & MUL2)
* **VLAN Extended ACLs:** Strict inter-VLAN traffic filtering on SVIs with implicit deny.
* **DAI Trust Configuration:** Trusted uplink interfaces to allow legitimate ARP traffic to gateways.
* **EIGRP MD5 Authentication:** Encrypted routing updates (`EigrpPass2026`) to prevent rogue neighbor adjacencies.

### 3. Core / WAN Layer Security (R1 & SP)
* **Perimeter Extended ACLs:** Traffic filtering to drop malicious WAN packets.
* **Interface Management:** Administrative shutdown of all unused physical and logical ports.
* **PAT / NAT Overload:** Translates private IP space to hide internal enterprise architecture.
* **EIGRP MD5 Authentication:** Secured control plane communication between R1, Distribution switches, and SP.

### 4. Control & Management Plane Protection
* **SSH (Secure Shell):** Encrypted VTY access across all network devices.
* **AAA Framework:** Local authentication and authorization for administrative access.

---

## 🧪 Verification & Security Testing (PoC)

### 1. Layer 2 Security Verification
| Test Scenario | Command / Log Output | Result |
| :--- | :--- | :--- |
| **Port Security Enforcement** | `show port-security interface <int>` | Dropped unauthorized frames (`protect` mode active) |
| **Dynamic ARP Inspection** | `show ip arp inspection statistics` | Dropped invalid ARP packets from rogue hosts |
| **DHCP Snooping Status** | `show ip dhcp snooping binding` | Valid IP-to-MAC bindings stored correctly |

#### 📸 Proof Screenshots:
* **Port Security & DAI Validation:**  
  ![Port Security Drop](screenshots/port_security.png)

* **DHCP Snooping Table:**  
  ![DHCP Snooping Table](screenshots/dhcp_snooping.png)

---

### 2. Perimeter & Routing Verification
| Test Scenario | Command / Log Output | Result |
| :--- | :--- | :--- |
| **WAN Threat Mitigation** | `show access-lists BLOCK_ATTACK_PORT` | Matches/drops observed for unauthorized traffic |
| **EIGRP MD5 Authentication** | `show ip eigrp neighbors` | Secured neighbor adjacencies established |
| **Path Tracking** | `trace 8.8.8.8` | Traffic routed successfully to WAN destination |

#### 📸 Proof Screenshots:
* **ACL Match Counters:**  
  ![ACL Drop Verification](screenshots/acl_matches.png)

* **Traceroute Results:**  
  ![Traceroute Proof](screenshots/traceroute.png)
