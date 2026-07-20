# My Home Lab Setup

A hands-on environment built to simulate enterprise IT environments, practice network administration, and test system deployments.

---

## Project Overview
This lab was built to practice Active Directory configuration, network segmentation, and basic virtualization management.

* **Goal:** Practice domain administration and secure network configuration.
* **Key Technologies:** Proxmox VE (Nested), Oracle VirtualBox, Windows Server 2022, Windows 11 Enterprise, pfSense, Ubuntu Server 22.04 LTS
* **Hypervisor Layer:** Type 1 Proxmox VE hypervisor running nested inside a Type 2 VirtualBox sandbox environment.

* **WAN/LAN Gateway:** pfSense Virtual Firewall
* **Domain Controller:** Windows Server 2022 (IP: `192.168.10.10`)
* **Client Machine:** Windows 11 Enterprise (DHCP)
* **Linux Server:** Ubuntu 22.04 LTS (IP: `192.168.10.20`)

---

## 🛠️ Hardware & Software Stack

### Hardware Specs
| Component | Device / Specification | Purpose |
| :--- | :--- | :--- |
| Component | Device / Specification | Purpose |
| :--- | :--- | :--- |
| **Host CPU** | AMD Ryzen 7 2700X (8C / 16T) | Allocating 4 Cores to the virtualized Proxmox environment |
| **Host RAM** | 16 GB DDR4 | Allocating 8 GB to Proxmox, leaving 8 GB for the daily host OS |
| **Storage** | 200GB Virtual Disk (`.vdi`) | Dedicated virtual block storage for nested infrastructure |
### Hypervisor & VMs
* **Hypervisor:** Proxmox VE 9.2 (built inside a VirtualBox VM) - 4 CPU cores, 8gb ram
  * **VM 1:** pfSense (Router/Firewall) - 1 Cores, 2GB RAM
  * **VM 2:** Windows Server 2022 (Active Directory) - 2 Cores, 4GB RAM
  * **VM 3:** Ubuntu Server (Docker Host) - 1 Cores, 2GB RAM

---

## 🚀 Configuration & Deployment Steps


### Step 1: Hypervisor & Virtual Networking Setup
1. Installed **Proxmox VE** on the host hardware inside a VM using VirtualBox.
2. Created two virtual bridges:
   * `vmbr0` (WAN/External - bridged to physical network)
   * `vmbr1` (LAN/Internal - isolated network for VMs)

### Step 2: Active Directory Installation & Domain Setup
1. Installed Windows Server 2022 on VM 2.
2. Configured a static IP (`192.168.10.10`) and pointed DNS to loopback (`127.0.0.1`).
3. Promoted the server to a Domain Controller for the domain `lab.local`.
4. Created an Organizational Unit (OU) structure for `Employees` and `Computers`.

---

## 🔍 Troubleshooting & Lessons Learned
> 🔍 **Issue:** Proxmox VE web interface timed out or failed to connect when switching VirtualBox from a Bridged Adapter to a NAT Network.
> 
> * **Root Cause:** Proxmox assigned a static IP address to its virtual bridge (`vmbr0`) during the initial installation. When the VirtualBox network type was changed to a NAT Network to bypass Wi-Fi hardware limitations, the virtual machine was assigned a new private subnet IP (10.0.2.15). However, Proxmox was still listening internally on the old static IP, causing a total disconnect and timeout.
> * **Resolution:** Resolved the issue in two phases:
>   1. **Updated Proxmox Network Configurations:** Logged directly into the Proxmox text console via VirtualBox, opened `/etc/network/interfaces` and `/etc/hosts` using `nano`, and updated the static IP definitions to match the new NAT subnet gateway (10.0.2.15 / 10.0.2.1).
>   2. **Configured VirtualBox Port Forwarding:** Poked a secure hole through the VirtualBox NAT firewall. Created a port forwarding rule mapping the host's loopback address (`127.0.0.1:8006`) directly to the Proxmox guest VM (`10.0.2.15:8006`). This allowed seamless access via the local host's web browser using `https://127.0.0.1:8006`.


> 💡 **Issue:** Proxmox VE kernel panic / continuous reboot loop right after reaching the login console screen inside VirtualBox.
> 
> * **Root Cause:** Resource contention and CPU scheduling synchronization delays. Allocating too many CPU cores (6) to the nested Type 2 VirtualBox environment on an 8-core host machine starved the host OS of processing threads, causing timing mismatches that forced the Proxmox kernel to panic and reset.
> * **Resolution:** Reduced the VirtualBox VM CPU allocation down to **4 Cores** and ensured that hardware-enforced security features (**PAE/NX** and **Nested VT-x/AMD-V**) were explicitly enabled. This instantly stabilized the kernel execution environment.
---

## 🎯 Next Steps & Future Plans
* [ ] Implement a centralized logging server (SIEM) using ELK Stack or Splunk.
* [ ] Write a PowerShell script to automate user onboarding into Active Directory.
* [ ] Deploy an open-source ticket system (like osTicket) to practice helpdesk workflows.

