# Centralized Enterprise Network Edge & Security Fabric

![Network Topology](Images/Topology.jpg)

## 📌 Project Overview
This project demonstrates the design, implementation, and centralized management of a secure, future-proof enterprise network edge using the **Fortinet Ecosystem**. Managed entirely via **FortiManager**, the architecture transitions a traditional static-routed multi-site branch topology into an abstracted, scalable **SD-WAN** fabric while enforcing strict Layer 7 application control policies.

This environment was deployed within a virtualized sandbox hypervisor (**PNetLab**), mimicking real-world production data center and branch deployment workflows.

---

## 1. Network Segmentation & Layer 3 IP Schema

The enterprise infrastructure segregates user traffic, secure server environments, and branch routing lines through distinct 802.1Q virtual boundaries and deterministic sub-interfaces.

### VLAN Allocations
* **VLAN 10 (HQ_Data Plane)**: Assigned for internal corporate client and local compute resources at Headquarters.
* **VLAN 20 (HQ_Servers Plane)**: Dedicated data center segment hosting critical core server assets, including the FortiManager appliance.
* **VLAN 30 (BR_Data Plane)**: Segment dedicated to remote branch office client subnets.
* **VLAN 99 (Native Infrastructure)**: Configured across all internal trunk links to enforce proper tag encapsulation and network sanitation.

### Device IP Address Assignments

| Node Name | Physical/Logical Interface | IP Address Assignment | Purpose / Description |
| :--- | :--- | :--- | :--- |
| **Fortinet_FMG** | Port 1 | `192.168.192.131/24` | Centralized Policy & Object Manager (NSE 5 Platform) |
| **HQ_FGT** | Port 1 (Management) | `192.168.192.137/24` | Primary Hub Firewall Management Gateway |
| **HQ_FGT** | Port 2 (WAN) | Dynamic `10.0.137.117/24` | Hub ISP Gateway Transport Interface |
| **HQ_FGT** | Port 3.10 (HQ_Data) | `10.10.10.254/24` | Default Gateway for Local Corporate Network |
| **HQ_FGT** | Port 3.20 (HQ_Servers) | `10.20.20.254/24` | Default Gateway for High-Value Server Domain |
| **HQ_Core_SW** | VLAN 20 SVI | `10.20.20.1/24` | Core Distribution Switching Management Layer IP |
| **Branch_FGT** | Port 1 (Management) | `192.168.192.136/24` | Remote Spoke Firewall Management Gateway |
| **Branch_FGT** | Port 2 (WAN) | Dynamic `10.0.137.15/24` | Spoke ISP Gateway Transport Interface |
| **Branch_FGT** | Port 3.30 (BR_Data) | `10.10.20.254/24` | Default Gateway for Remote Spoke Clients |

---

## 2. Spanning Tree Protocol (STP) Domain Design

To ensure predictable path redundancy and protect against devastating switching loops, **Rapid Per-VLAN Spanning Tree Plus (Rapid-PVST+)** was explicitly configured across the Cisco switching tier.

* **Primary Root Bridge (`HQ_Core_SW`)**: Enforced with a priority of `24576` for VLANs 10, 20, and 99, ensuring internal server and data traffic gravitates predictably toward the core distribution layer.
* **Secondary Bridges (`HQ_Access_SW`, `BR_Core_SW`)**: Left at default priorities (`32768`) to seamlessly inherit downstream alternate/designated pathing blocks.

### Infrastructure Boot Initialization Matrix (PNetLab Optimization)
Due to virtualized CPU resource constraints and device startup behavior in sandboxed environments, specific execution states must be met to avoid DHCP lease timing issues or FGFM tunnel dropouts. Nodes must be activated sequentially:

1. **Phase I: Cisco Switching Fabric (`HQ_Core_SW`, `HQ_Access_SW`, `BR_Core_SW`)** *Objective:* Establish 802.1Q trunk connections and allow Rapid-PVST+ to fully converge into a stable forwarding state before frames are injected.
2. **Phase II: Security Gateways & Orchestration (`HQ_FGT`, `Fortinet_FMG`, `BR_FGT`)** *Objective:* Bring local firewall interfaces online and ensure the FortiManager's FGFM daemon is listening on TCP port 541 for device registration requests.
3. **Phase III: Client Endpoint Tier (`HQ_PC_1`, Windows VMs)** *Objective:* Power down endpoints until upstream DHCP daemons on the FortiGates are fully awake, preventing client operating systems from timing out or generating APIPA addresses (`169.254.x.x`).

---

## 3. Site-to-Site IPsec VPN Tunnel Architecture

A robust overlay tunnel was engineered between **HQ_FGT (Hub)** and **Branch_FGT (Spoke)** to guarantee confidential, cross-premises encryption over unsecure WAN boundaries.

### Cryptographic Profiles
* **IKE Version:** IKEv2
* **Phase 1 (ISAKMP Negotiation):** AES-256 Encryption | SHA-256 Hashing | Diffie-Hellman Group 14 (2048-bit)
* **Phase 2 (ESP Data Payload):** AES-256 Encryption | SHA-256 Hashing | Perfect Forward Secrecy (PFS) Enabled via DH Group 14
* **Proactive Lab Optimization:** Configured Phase 2 settings with `set auto-negotiate enable` and `set keepalive enable`. This commands the FortiGate kernel to proactively forge and keep the cryptographic tunnel up 24/7 without requiring manual, host-generated interesting traffic to trigger the path.

---

## 4. Centralized Management (FortiManager Orchestration)
* Abstracted the firewalls' local configurations by separating management planes into independent **Device Settings** databases and shared **Policy Packages**.
* Utilized **Normalized Interfaces (Interface Mapping)** on FortiManager to bind generic logical zones (`LAN_Zone`, `WAN_Zone`) to distinct, site-specific physical ports across both the HQ and Branch devices.
* Standardized deployment workflows, allowing a single, centralized policy package to safely govern multiple structural boundaries via automated deployment wizards.

---

## 5. High-Performance WAN Edge (SD-WAN Fabric)
* Abstracted the physical internet gateway interface (`port2`) into a virtualized **`sdwan` interface zone**, providing a scalable infrastructure that permits seamless multi-homed ISP expansion without breaking downstream firewall rule dependencies.
* Implemented symmetric traffic-shaping configurations scaled to $100\text{ Mbps}$ bandwidth baselines to ensure interface optimization.
* Developed active **Performance SLAs** tracking critical performance indicators (latency, jitter, and packet loss) via ping probes to public infrastructure nodes (`8.8.8.8`) for dynamic, automated link quality monitoring.

---

## 6. Layer 7 Next-Generation Security Profiles
* **Application Control Profiles:** Built and applied a strict corporate application block policy targeting bandwidth-intensive, unapproved, and social media software categories (e.g., explicit drop execution on `facebook.com`).
* **SSL Inspection Integration:** Pushed customized, flow-based certificate inspection structures to achieve deep packet inspection visibility over encrypted HTTPS handshakes without interrupting underlying application stability.
* **Security & Routing Synchronicity:** Re-engineered traditional static default gateway rules, migrating routing pathways directly into the SD-WAN virtual framework. Firewall traffic policies are bound cleanly to the SD-WAN zone profiles, ensuring unified security enforcement across all transport vectors.
