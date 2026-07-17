# My Home Lab Setup

A hands-on environment built to simulate enterprise IT environments, practice network administration, and test system deployments.

---

## Project Overview
This lab was built to practice Active Directory configuration, network segmentation, and basic virtualization management.

* **Goal:** Practice domain administration and secure network configuration.
* **Key Technologies:** Active Directory, pfSense, VirtualBox, ProxMox, Ubuntu Server, Windows 10/11, Windows Server 2022

* **WAN/LAN Gateway:** pfSense Virtual Firewall
* **Domain Controller:** Windows Server 2022 (IP: `192.168.10.10`)
* **Client Machine:** Windows 11 Enterprise (DHCP)
* **Linux Server:** Ubuntu 22.04 LTS (IP: `192.168.10.20`)

---

## 🛠️ Hardware & Software Stack

### Hardware Specs
| Component | Device / Specification | Purpose |
| :--- | :--- | :--- |
| **Host Machine** | AMD Ryzen 7 2700x 6 core, GTX 1080, 16gb (2 x 8) DDR4 Ram | Hypervisor Server |
| **Network** | Ubiquiti UniFi Switch & AP | Physical VLAN segmentation |

### Hypervisor & VMs
* **Hypervisor:** Proxmox VE 9.2 (built inside a VirtualBox VM) - 7 CPU cores, 8gb ram
  * **VM 1:** pfSense (Router/Firewall) - 2 Cores, 2GB RAM
  * **VM 2:** Windows Server 2022 (Active Directory) - 2 Cores, 4GB RAM
  * **VM 3:** Ubuntu Server (Docker Host) - 2 Cores, 4GB RAM

---

## 🚀 Configuration & Deployment Steps

Detail how you actually built this. Keep it clear enough that another IT tech could replicate your setup.

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

Show off your problem-solving skills here. Recruiters care more about *how you fix things* than a perfect run.

> 💡 **Issue:** Windows 11 client VM could not resolve the `lab.local` domain when attempting to join.
> * **Root Cause:** The client VM's network adapter was pointing to public DNS (`8.8.8.8`) instead of the Domain Controller (`192.168.10.10`).
> * **Resolution:** Configured the local DHCP server on pfSense to hand out the Domain Controller's IP as the primary DNS server. Resolved instantly.

---

## 🎯 Next Steps & Future Plans
* [ ] Implement a centralized logging server (SIEM) using ELK Stack or Splunk.
* [ ] Write a PowerShell script to automate user onboarding into Active Directory.
* [ ] Deploy an open-source ticket system (like osTicket) to practice helpdesk workflows.

