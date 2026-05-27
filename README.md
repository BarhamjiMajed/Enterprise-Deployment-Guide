# Enterprise-Deployment-Guide
# 🚀 Deployment Guide: Windows Server 2019 Domain Controller

### Core Configuration Data

* **Domain Name:** `Globe.com`
* **Static IP:** `192.168.10.10/24`
* **Gateway:** `192.168.10.1`
* **DNS Server:** `127.0.0.1`

---

## Phase 1: Initial System Setup (Networking & Time)

Before installing roles, ensure the system is properly configured with a static IP and the correct time zone.

* **Option A: Graphical User Interface (GUI)**
1. **Time Zone:** Right-click the clock in the taskbar > **Adjust date/time** > Set your correct time zone.
![description](images/set-time-zone.png)


2. **Network IP:** Open `ncpa.cpl` > Right-click **Ethernet** > **Properties** > **IPv4 Properties**. Enter:
* IP: `192.168.10.10` | Subnet: `255.255.255.0` | Gateway: `192.168.10.1`
* DNS: `127.0.0.1`

![description](images/set-ip-address.png)



3. **Rename Server:** In **Server Manager** > **Local Server** > Click **Computer name** > **Change**. Rename to `SVR-PDC` and restart.


* **Option B: PowerShell (Automation)**
```powershell
# Update Time Zone
Set-TimeZone -Id "Arab Standard Time"

# Set Static IP and DNS
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1

# Rename and Restart
Rename-Computer -NewName "SVR-PDC" -Restart

```



---

## Phase 2: Installing Active Directory Domain Services (AD DS)

* **Option A: GUI**
1. In **Server Manager**, click **Manage** > **Add roles and features**.
2. Select **Active Directory Domain Services** from the server roles list.
3. Click **Add Features** in the pop-up, then proceed to finish the installation.


![description](images/Add-ADDS-Role.png)


* **Option B: PowerShell**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

```



---

## Phase 3: Promoting Server to Domain Controller (Forest Creation)

* **Option A: GUI**
1. In **Server Manager**, click the **Notification flag** (Yellow triangle) > **Promote this server to a domain controller**.
2. Select **Add a new forest** and enter `Globe.com` as the root domain name.
3. On the **Domain Controller Options** screen, set a strong **DSRM password**.
4. Proceed through the following screens (DNS Options, NetBIOS, Paths) clicking **Next**.
5. Once the "Prerequisites Check" passes, click **Install**.


![description](images/Review-ADDS-Configiuration.png)


* **Option B: PowerShell**
```powershell
# Promotion command (Prompts for DSRM password)
Install-ADDSForest -DomainName "Globe.com" -InstallDns -Force

```



---

## Phase 4: Verification

Once the system restarts, verify the environment:

1. **Check DNS:** Open CMD and run `nslookup Globe.com`. It should return `192.168.10.10`.
2. **Verify AD:** Open **Active Directory Users and Computers** and ensure the `Globe.com` domain structure is visible.

---
