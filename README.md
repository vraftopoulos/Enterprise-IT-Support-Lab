# Windows Server 2022 Active Directory Home Lab
## Project Overview
This project demonstrates the deployment of a secure Active Directory (AD) environment within a virtualized home lab using Oracle VirtualBox. It includes configuring a Windows Server 2022 Domain Controller (DC), setting up a dedicated private network, implementing DHCP/DNS services, creating automated user accounts via PowerShell, and joining a Windows 10 Client to the domain.

## Architecture & Network Design
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

## Step 1: VirtualBox Network Configuration
1. Created a dedicated **NAT Network** named `AD-Lab-Network`.
2. Configured the Network CIDR to `192.168.10.0/24`.
<img width="1159" height="80" alt="image" src="https://github.com/user-attachments/assets/a2dbdff1-b982-4478-b3c0-51b733a2f8e4" />

## Step 2: Windows Server 2022 Installation
1. Provisioned a new Virtual Machine (`DC-01`) with 2 vCPUs, 4GB RAM, and a 50GB VHD.
2. Attached the network adapter to `AD-Lab-Network`.
3. Installed **Windows Server 2022 Standard (Desktop Experience)**.
4. Configured the local Administrator account.

## Step 3: Network Configuration & Hostname Assignment
In an enterprise environment, Domain Controllers require predictable network identities. Dynamic IPs (DHCP) can cause critical failures in DNS resolution and authentication.

1. **Hostname Change:** Renamed the server from its default generated name to `DC-01`.
2. **Static IP Assignment:** Configured the IPv4 properties manually to isolate the DC and prepare it for the DNS role.
   * **IP Address:** `192.168.10.10`
   * **Subnet Mask:** `255.255.255.0`
   * **Default Gateway:** `192.168.10.1`
   * **Preferred DNS:** `127.0.0.1` (Loopback address, pointing to itself as the primary DNS server)

## Step 4: Active Directory Domain Services (AD DS) Installation & Promotion
To establish the enterprise identity paradigm, the server was promoted to a Domain Controller, establishing a new Active Directory Forest.

1. **Role Installation:** Installed the **Active Directory Domain Services** and **DNS Server** roles via Server Manager.
2. **Domain Controller Promotion:** Configured the server as the root Domain Controller for a new forest.
   * **Root Domain Name:** `mydomain.local`
   * **Functional Levels:** Windows Server 2016 (ensures maximum compatibility and modern AD features).
   * **DNS Delegation:** Automatically configured during the promotion process to allow `DC-01` to resolve local domain queries.

## Step 5: DHCP Server Installation & Scope Configuration
To automate IP distribution and network parameters across the enterprise environment, the DHCP Server role was deployed and authorized.

1. **Role Deployment:** Installed and authorized the **DHCP Server** role within the Active Directory infrastructure to prevent rogue DHCP server attacks.
2. **Scope Architecture (Client-Scope):**
   * **Address Pool:** `192.168.10.100` to `192.168.10.200` (Allows up to 101 concurrent client devices)
   * **Subnet Mask:** `255.255.255.0`
   * **Scope Options Distributed:**
     * **003 Router (Gateway):** `192.168.10.1`
     * **006 DNS Servers:** `192.168.10.10` (Directing all traffic to our Domain Controller for name resolution)
    
## Step 6: Client Provisioning & Active Directory Domain Join
To validate the infrastructure services (DHCP/DNS) and central management capability, a Windows 10 Client workstation was introduced to the environment.

1. **Network Alignment:** Attached the Windows 10 VM network adapter to `AD-Lab-Network`.
2. **Network Verification:** Executed `ipconfig /all` on the client to verify DHCP lease authorization.
   * Successfully received an IP from the pool (e.g., `192.168.10.100`).
   * Verified Primary DNS pointer was accurately targeted to `192.168.10.10` (DC-01).
3. **Domain Join Authentication:**
   * Navigated to System Properties -> Computer Name Changes.
   * Modified Hostname to `CLIENT-01`.
   * Authenticated against the Domain Controller using Domain Administrator credentials to complete the Secure Domain Join.
