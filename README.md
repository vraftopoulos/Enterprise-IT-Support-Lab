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

## Step 7: Automated User Provisioning via PowerShell (Infrastructure as Code)
Manually creating user accounts is inefficient and prone to human error. To align with modern DevOps and SysAdmin workflows, I developed and executed a PowerShell automation script to structure the Directory Environment.

### PowerShell Automation Script
The script programmatically performs the following enterprise tasks:
1. Provisions standard **Organizational Units (OUs)** based on department structures (`IT`, `HR`, `Finance`).
2. Generates user accounts using standard naming conventions (`firstinitial + lastname`).
3. Establishes secure default passwords (`UserPassword123!`) and places them directly into their respective OUs.

## Step 8: Production Validation & Enterprise Group Policy (GPO) Implementation
To finalize the deployment, identity verification was conducted, and centralized administrative controls were enforced via Group Policy Objects (GPOs).

### 1. Client-Side Authentication Success
* Successfully authenticated onto `CLIENT-01` using the programmatically generated domain account: `mydomain.local\jdoe`.
* Verified that Active Directory successfully provisioned a roaming local profile and enforced organizational domain restrictions upon workstation initialization.

### 2. Group Policy Object (GPO) Security Hardening
To demonstrate centralized system security management, a strict compliance policy was established:
* **Policy Name:** `Restrict_Users_GPO`
* **Objective:** Prevent non-administrative personnel from executing arbitrary scripts or commands via the Command Prompt to mitigate lateral movement risks.
* **Path:** `User Configuration -> Policies -> Administrative Templates -> System -> Prevent access to the command prompt -> Enabled`.


---

## Key Takeaways & Skills Demonstrated
By architecting and completing this hands-on Home Lab, I have successfully simulated and managed an enterprise-level Active Directory infrastructure, proving competencies in:
* **Network Infrastructure & Administration:** Managing isolated NAT Networks, configuring static routing, and executing DHCP/DNS role authorization.
* **Directory Services Architecture:** Domain Controller promotion, Active Directory design layout, and structural OU hierarchy.
* **Automation & Scripting:** Utilizing **PowerShell** for automated batch-user provisioning (Infrastructure as Code mentality).
* **Enterprise Security Policy Enforcement:** Mitigating insider threats and securing client endpoints utilizing **Group Policy Objects (GPOs)**.

* ## Step 9: Enterprise Password Policy & Brute-Force Protection (Account Lockout)
To satisfy strict regulatory compliance and protect corporate identities from automated brute-force or dictionary attacks, severe account security baselines were enforced.

### 1. Configuration Parameters
Modifications were applied directly to the **Default Domain Policy** under `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Account Policies`:

* **Password Complexity & Length:** Enforced a minimum password length of **12 characters** and enabled cryptographic complexity rules (requiring uppercase, lowercase, digits, and special characters).
* **Account Lockout Threshold:** Accounts are programmatically locked after **5 consecutive failed logon attempts**.
* **Account Lockout Duration:** Implemented a **30-minute cooling period** before the system auto-unlocks the account, mitigating automated persistence attempts.

### 2. Administrative Verification
* Tested the policy on the client workstation (`CLIENT-01`) by simulating a brute-force scenario on user account `jdoe`.
* Verified that the domain account successfully locked down after the 5th attempt.
* Demonstrated administrative remediation by navigating to *Active Directory Users and Computers* on `DC-01` and manually unlocking the staff token.

* ## Step 10: Peripheral Security & Data Loss Prevention (DLP) - USB Blocking
To mitigate the risk of corporate data exfiltration and insider threats via unauthorized physical media (USB flash drives, external hard drives), a strict hardware restriction policy was enforced.

### 1. Configuration Parameters
A dedicated GPO named `Block_USB_GPO` was created and linked to the root domain. The policy targets machine-level baselines under `Computer Configuration -> Policies -> Administrative Templates -> System -> Removable Storage Access`:

* **Setting enforced:** `All Removable Storage classes: Deny all access` -> **Enabled**.
* **Impact:** This configuration completely isolates the operating system's kernel from mounting external storage volumes, preventing both data copying (Read/Write) and execution of untrusted payloads (e.g., malicious `.exe` or BadUSB scripts).

### 2. Administrative Verification
* Enforced immediate policy distribution on `CLIENT-01` via `gpupdate /force`.
* Simulated a hardware insertion attack by mounting a physical USB drive through the host hypervisor integration.
* Confirmed endpoint enforcement; the system accurately generated an **"Access is Denied"** dialogue box when attempting file system traversal.

* ## Step 11: Enterprise Branding & Desktop Compliance Deployment
To ensure visual uniformity across corporate assets and enforce branding compliance guidelines, a centralized desktop background policy was deployed.

### 1. Infrastructure Preparation (Network Share)
* Created a secure, read-only directory network share at `C:\Branding` on `DC-01`.
* Stored the enterprise background image (`company_wallpaper.jpg`) and distributed it via the Universal Naming Convention (UNC) path: `\\dc-01\Branding\company_wallpaper.jpg`.

### 2. GPO Configuration Parameters
A dedicated GPO named `Company_Wallpaper_GPO` was created and linked to the root domain. Configurations were mapped under `User Configuration -> Policies -> Administrative Templates -> Desktop -> Desktop`:

* **Desktop Wallpaper:** Set to **Enabled** and pointed to the UNC network path with a style parameter of **Fill**.
* **Prohibit Changing Wallpaper:** Set to **Enabled** to strip end-users of local customization privileges over desktop environments.

### 3. Administrative Verification
* Triggered immediate compliance on `CLIENT-01` utilizing `gpupdate /force` and performed a user session refresh.
* Confirmed that user `jdoe` successfully inherited the corporate wallpaper.
* Verified security enforcement; the Windows Settings layout successfully locked the background modification options, flashing the standard *"Some settings are managed by your organization"* restriction banner.

* ## Step 12: Centralized Storage Architecture & Automated Drive Mapping
To eliminate localized storage dependencies and safeguard institutional data assets, an automated, server-side distributed storage network was established.

### 1. File Server Infrastructure Configuration
* Provisioned a centralized storage repository on `DC-01` located at `C:\CompanyShares\IT_Share`.
* Implemented SMB (Server Message Block) file-sharing permissions, granting network execution capabilities (`Read/Write/Change`) to targeted domain profiles.

### 2. GPO Drive Maps Redirection
A dedicated GPO named `Drive_Mapping_GPO` was leveraged to dynamically push infrastructure attachments. Configured under `User Configuration -> Preferences -> Windows Settings -> Drive Maps`:

* **Action type:** `Update`
* **Remote Path Targeting (UNC):** `\\dc-01\CompanyShares\IT_Share`
* **Device Letter Mapping:** Bound uniformly to environmental mount point **`S:`**.
* **Display Label:** Identified globally across endpoints as `IT Department Share`.

### 3. Administrative Verification
* Triggered network synchronization on the workstation endpoint using the operational command pipeline execution via `Windows + R -> gpupdate /force`.
* Navigated to *This PC* on `CLIENT-01` under user context `jdoe` and verified the seamless operational mounting of the `S:` network volume.
* Confirmed bidirectional stability by executing folder read/write stress-tests mirroring active production deployments.

* ## Step 13: Automated Software Deployment via Group Policy
To streamline endpoint provisioning and eliminate manual software configuration workflows, centralized software deployment was orchestrated utilizing Windows Server's native deployment engine.

### 1. Repository Configuration
* Downloaded the enterprise-grade Google Chrome MSI package (`googlechrome.msi`).
* Established a secure distribution repository at `C:\Software` on `DC-01`, configured as a read-only network SMB share accessible globally via UNC: `\\dc-01\Software\googlechrome.msi`.

### 2. GPO Package Assignment
A dedicated GPO named `Software_Deployment_GPO` was linked to the environment. The software distribution properties were defined under `Computer Configuration -> Policies -> Software Settings -> Software installation`:

* **Deployment Method:** Set to **Assigned**.
* **Targeting:** Configured using the strict network UNC path to ensure the endpoint system contexts can retrieve the binaries during network bootstrapping.

### 3. Verification & Compliance Delivery
* Executed a complete system restart on `CLIENT-01` to trigger early-stage computer boot execution blocks.
* Monitored the initial boot cycle; the Windows subsystem successfully captured the GPO directive, displaying the standard installation message banner.
* Verified compliance post-authentication; Google Chrome was deployed natively across the operating system layer, visible on user `jdoe`'s desktop without manual administrative intervention.

* ## Step 14: Disaster Recovery & User Data Resilience (Folder Redirection)
To prevent corporate data loss from physical endpoint failures and streamline workstation migrations, a centralized Folder Redirection architecture was deployed.

### 1. File Server Infrastructure
* Provisioned a baseline storage repository on `DC-01` at `C:\UserProfiles`.
* Configured advanced SMB sharing permissions enabling `Everyone: Full Control`, allowing the Active Directory subsystem to programmatically provision and sandbox isolated user directories with strict NTFS inherited ownership.

### 2. GPO Technical Specifications
A dedicated GPO named `Folder_Redirection_GPO` was mapped under `User Configuration -> Policies -> Windows Settings -> Folder Redirection -> Desktop`:

* **Setting applied:** `Basic - Redirect everyone's folder to the same location`.
* **Placement Option:** `Create a folder for each user under the root path`.
* **Root UNC Path:** `\\dc-01\UserProfiles`.
* **Result:** The local user shell environment dynamically targets the network repository, seamlessly migrating the local Desktop profile data to the Server over SMB.

### 3. Administrative Verification
* Executed environmental policy evaluation on `CLIENT-01` using `gpupdate /force` and authorized the requested user session logoff.
* Authenticated as user `jdoe` and generated a dummy file (`jdoe_important_file.txt`) directly onto the client desktop surface.
* Inspected `DC-01` under `C:\UserProfiles\jdoe\Desktop` and verified that the file systems synchronized seamlessly, ensuring high-availability data compliance.

* ## Step 16: Centralized Network Security & Firewall Hardening
To establish an internal network security perimeter and prevent lateral threat proliferation (e.g., Ransomware/Worm propagation), Endpoint Firewall management was operationalized and locked via GPO.

### 1. Global Policy Enforcement
Constructed a baseline security GPO named `Firewall_Hardening_GPO` targeting all active managed assets. Configured under `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Defender Firewall with Advanced Security`:
* **State Synchronization:** Mandated the Firewall state to **ON** across all functional profiles: *Domain*, *Private*, and *Public*.
* **UI Hardening:** Programmatically restricted local structural overrides, preventing users or malicious software from deactivating network protections.

### 2. Controlled Exception Provisioning (ICMPv4/Ping)
To maintain administrative diagnostics without widening the endpoint attack surface, a strict inbound exception rule was pushed:
* **Rule Logic:** Leveraged the predefined `File and Printer Sharing` framework.
* **Scope Isolation:** Enforced a single, isolated inbound exception for **`Echo Request - ICMPv4-In`**, discarding generic file-sharing protocols.
* **Execution:** Enabled secure bidirectional packet routing exclusively for network heartbeat mapping (`ping`).

### 3. Verification & Compliance Logging
* Forced environmental distribution using `gpupdate /force` on `CLIENT-01`.
* Inspected the client-side Firewall settings interface; verified the interface was successfully locked with the administrative banner warning.
* Executed an ICMP diagnostics evaluation from `DC-01` to `CLIENT-01` and validated stable, successful packet response streams over the secure channel.

* ## 🛑 Step 17: Application Whitelisting & Ransomware Mitigation (AppLocker)
To neutralize the execution of unauthorized software, shadow IT applications, and untrusted binaries (e.g., Ransomware payloads) within user profiles, an AppLocker application control baseline was enforced.

### 1. Service Orchestration
Created a dedicated GPO named `AppLocker_Policy_GPO`. To ensure client-side compliance engine availability, the endpoint subsystem was configured under `Computer Configuration -> Policies -> Windows Settings -> Security Settings -> System Services`:
* **Application Identity Service:** Set to **Automatic** initialization.

### 2. Rule Blueprint & Enforcement Settings
Configured the application execution constraints under `Security Settings -> Application Control Policies -> AppLocker -> Executable Rules`:
* **Baseline Authorization:** Generated the standard Windows **Default Rules** matrix. This workflow authorizes binary operations strictly bounded to secure system roots (`%systemroot%\*` and `%programfiles%\*`).
* **Enforcement State:** Armed the structural architecture to **Enforce Rules** state, successfully blacklisting execution capabilities on user-writable paths such as `C:\Users\*`.

### 3. Verification Posture
* Deployed policies to `CLIENT-01` via execution pipeline redirection and recycling cycles (`gpupdate /force` & system reboot).
* Logged in under user context `jdoe` and attempted to execute an unapproved standalone binary wrapper downloaded into the local profile environment.
* Verified compliance enforcement; the Windows kernel intercepted the execution thread, dropping the operation and delivering the administrative blockage notification banner.

## 📜 Step 20: Centralized Logging via Windows Event Forwarding (WEF)
To establish operational visibility and secure audit trails against anti-forensics techniques (e.g., local log clearing by malicious actors), enterprise Windows Event Forwarding (WEF) was implemented.

### 1. Collector Initialization
* Configured `DC-01` as the central Log Collector utilizing the native Windows Event Collector utility (`wecutil qc`), provisioning high-performance HTTP listeners for event ingestion.

### 2. GPO Target Orchestration
Created a dedicated asset delivery GPO named `Event_Forwarding_GPO` linked strictly to the Workstations OU. Configured under `Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Event Forwarding`:
* **Configure target Subscription Manager:** Programmatically bound managed endpoints to look up the secure management URI: `Server=http://<DC_IP>:5985/wsman/SubscriptionManager/WEC,Refresh=60`.

### 3. Subscription Ruleset & Verification
* Created a *Source-Initiated Subscription* within the Event Viewer database on `DC-01` targeting the `Domain Computers` global identifier.
* Scoped data ingestion explicitly to **Security** and **System** channels filtering for Critical, Warning, and Audit Failure structural IDs.
* Triggered synchronization via `gpupdate /force` on `CLIENT-01`. Generated purposeful login evaluation failures to trigger audit tracking, and verified real-time event pipeline redirection inside the **Forwarded Events** repository on the Domain Controller.

