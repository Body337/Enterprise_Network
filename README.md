# Centralized Enterprise Network
![Network Topology](Images/Topology.jpg)
## 📌 Project Overview
This project demonstrates the design, implementation, and centralized management of a secure, future-proof enterprise network edge using the **Fortinet Ecosystem**. Managed entirely via **FortiManager**, the architecture transitions a traditional static-routed topology into an abstracted, scalable **SD-WAN** fabric while enforcing strict application control policies.

This environment was deployed  within a virtualized sandbox hypervisor, mimicking real-world data center and branch deployment workflows.

---
### VLAN Allocations
* **VLAN 10 (Data Plane)**: Assigned for internal corporate client and desktop compute resources.
* **VLAN 20 (Servers Plane)**: Dedicated out-of-band management tier hosting critical core assets including the FortiManager appliance.
* [cite_start]**VLAN 99 (Native VLAN)**: Used on standard trunk boundaries across internal network switches to maintain proper frame tagging practices[cite: 132].

### Device IP Address Assignments

| Node Name | Physical/Logical Interface | IP Address Assignment | Purpose / Description |
| :--- | :--- | :--- | :--- |
| **Fortinet_FMG** | Port 1 | `192.168.192.131/24` | [cite_start]Centralized Policy & Object Manager (NSE 5 Platform) [cite: 335, 340] |
| **HQ_FGT** | Port 1 (Managment) | `192.168.192.137/24` | [cite_start]Primary Hub Firewall (External Transport Gate) [cite: 335] |
| **HQ_FGT** | Port 2 (wan) | `dynamic` | Remote Spoke Site-02 Firewall Edge |
| **HQ_FGT** | Port 3.10(HQ-Data) | `10.10.10.254/24` | [cite_start]Default Gateway for VLAN 10 Data Networks [cite: 152] |
| **HQ_FGT** | Port 3.20(GQ-Servers) | `10.20.20.254/24` | [cite_start]Default Gateway for VLAN 20 Management Networks [cite: 152, 188] |
| **HQ_Core_SW** | VLAN 20 SVI | `10.20.20.1/24` | Core Infrastructure Switching Management IP |
| **Branch_FGT** | Port 1 (Management) | `192.168.136/24` | Remote Spoke Site-01 Firewall Edge |
| **Branch_FGT** | Port 2 (wan) | `dynamic` | Remote Spoke Site-02 Firewall Edge |
| **Branch_LAN** | Port 3.30 (BR-Data) | `10.10.20.0/24` range | [cite_start]Internal Spoke Subnets Managed via Dynamic Mapping [cite: 44] |

## 🛠️ Tech Stack & Architecture Components
* **Central Management:** FortiManager (v6.2)
* **Next-Generation Firewall (NGFW):** FortiGate Virtual Appliance (FortiOS v6.2)
* **Hypervisor/Lab Environment:** PNETLab / VMware Workstation
* **Core Mechanisms:** SD-WAN, Performance SLAs, Application Control, Flow-based SSL Inspection, Static Routing.

---

## 📐 Network Topology & Design Features

### 1. Centralized Management (FortiManager)
* Separated configuration database management from live execution by maintaining separate **Device Settings** and **Policy Packages**.
* Utilized the consolidated deployment wizard to maintain a single source of truth for the edge environment.

### 2. High-Performance WAN Edge (SD-WAN)
* Abstracted the physical internet interface (`port2`) into a virtual **`sdwan` interface zone** to ensure future-proof multi-homed ISP expansion without firewall policy disruption.
* Maintained traffic shaping benchmarks with configured traffic metrics ($100\text{ Mbps}$ Symmetric bandwidth baselines).
* Configured automated **Performance SLAs** tracking latency and packet loss against public infrastructure (`8.8.8.8`) to govern automated path monitoring.

### 3. Layer 7 Next-Generation Security
* **Application Control:** Designed and implemented a strict application policy blocking bandwidth-wasting, high-risk, and social media platforms (e.g., explicit blocking of `facebook.com`).
* **SSL Inspection Integration:** Applied customized flow-based inspection profiles to ensure deep packet inspection over encrypted HTTPS handshakes without breaking underlying application traffic.
* **Security & Routing Synchronicity:** Re-engineered traditional default gateway routing rules to map directly to the `sdwan` interface while binding granular firewall policies to the SD-WAN zone structure.

---
