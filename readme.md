# Centralized Enterprise Edge Security & SD-WAN Deployment

## 📌 Project Overview
This project demonstrates the design, implementation, and centralized management of a secure, future-proof enterprise network edge using the **Fortinet Ecosystem**. Managed entirely via **FortiManager**, the architecture transitions a traditional static-routed topology into an abstracted, scalable **SD-WAN** fabric while enforcing strict application control policies.

This environment was deployed  within a virtualized sandbox hypervisor, mimicking real-world data center and branch deployment workflows.

---

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