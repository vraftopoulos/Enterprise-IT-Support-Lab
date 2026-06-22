# Windows Server 2022 Active Directory Home Lab
## 📌 Project Overview
This project demonstrates the deployment of a secure Active Directory (AD) environment within a virtualized home lab using Oracle VirtualBox. It includes configuring a Windows Server 2022 Domain Controller (DC), setting up a dedicated private network, implementing DHCP/DNS services, creating automated user accounts via PowerShell, and joining a Windows 10 Client to the domain.

## 🏗️ Architecture & Network Design
To simulate a real-world enterprise environment, all virtual machines are isolated within a dedicated NAT Network.

* **Subnet:** `192.168.10.0/24`
* **Domain Controller (DC):**
    * **OS:** Windows Server 2022
    * **Hostname:** `DC-01`
    * **IP Address:** `192.168.10.10` (Static)
    * **Domain Name:** `mydomain.local`
* **Client PC:**
    * **OS:** Windows 10 Pro
    * **Hostname:** `CLIENT-01`
    * **IP Address:** Assigned via DHCP (`192.168.10.100` onwards)

---

## 🚀 Step 1: VirtualBox Network Configuration
1. Created a dedicated **NAT Network** named `AD-Lab-Network`.
2. Configured the Network CIDR to `192.168.10.0/24`.
<img width="1159" height="80" alt="image" src="https://github.com/user-attachments/assets/a2dbdff1-b982-4478-b3c0-51b733a2f8e4" />

