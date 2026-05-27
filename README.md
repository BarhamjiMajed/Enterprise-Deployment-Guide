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
To fully configure the DNS server to serve internal devices, resolve local domain names (like `Globe.com`), and allow all client machines to access the internet seamlessly, we must set up two main components: **DNS Forwarders** and a **Reverse Lookup Zone**.

---

## Phase 5: Configuring DNS Forwarders (For Internet Access)

DNS Forwarders are public DNS servers (such as Google or Cloudflare) that your local Domain Controller queries when an internal client requests an external internet address that the local server does not recognize.

### 🔹 Option A: Using the Graphical User Interface (GUI)

1. Open **Server Manager**, click on **Tools** in the top right corner, and select **DNS**.
2. In the left pane, right-click on your server's name (e.g., `SVR-PDC`) and select **Properties**.
3. Navigate to the **Forwarders** tab and click the **Edit** button.
4. Add the following highly reliable global DNS server IP addresses:
* Google Primary DNS: `8.8.8.8`
* Cloudflare Primary DNS: `1.1.1.1`


5. Click **OK**, then **Apply**, and **OK** again.

![description](images/set-Forwarders-DNS.PNG)

### 🔹 Option B: Using PowerShell (Automation)

Open PowerShell as Administrator and run the following command to set the forwarders instantly:

```powershell
# Set Google and Cloudflare as DNS Forwarders for the local server
Set-DnsServerForwarder -IPAddress "8.8.8.8","1.1.1.1" -PassThru

```

---

## Phase 6: Creating a Reverse Lookup Zone

A Reverse Lookup Zone translates IP addresses back into hostnames (the exact opposite of a forward lookup). It is highly recommended in an Active Directory environment to ensure network stability and allow various services to function without errors.

### 🔹 Option A: Using the Graphical User Interface (GUI)

1. In the **DNS Manager**, expand your server name in the left pane.
2. Right-click on **Reverse Lookup Zones** and select **New Zone**.
3. In the New Zone Wizard, click **Next**. Choose **Primary zone** and ensure **Store the zone in Active Directory** is checked, then click **Next**.
4. For the Replication Scope, select the second option: `To all DNS servers running on domain controllers in this domain` and click **Next**.
5. Select **IPv4 Reverse Lookup Zone** and click **Next**.
6. In the **Network ID** field, type the first three octets of your network: `192.168.10` and click **Next**.
7. On the Dynamic Update screen, leave the secure default option: `Allow only secure dynamic updates`. Click **Next**, then **Finish**.


![description](images/Set-Reverse-Lookup-Zones.PNG)

### 🔹 Option B: Using PowerShell (Automation)

Run this command to create an AD-integrated Reverse Lookup Zone immediately:

```powershell
# Create a Reverse Lookup Zone for the 192.168.10.0 network and replicate it across the domain
Add-DnsServerPrimaryZone -NetworkId "192.168.10.0/24" -ReplicationScope "Domain"

```

---

## Phase 7: Client Configuration and Final Verification

For client devices to successfully access the internet and resolve local domain resources, their network settings must be strictly pointed to your new DNS server.

**1. Client Network Adapter Setup:**

* Ensure that the **Preferred DNS server** on every client machine in your network is set strictly to the Domain Controller's IP address: `192.168.10.10`.
* *(Note: Never set 8.8.8.8 as a primary or secondary DNS on a client's network card in an AD environment, otherwise, they will lose connection to the local domain).*

**2. Testing and Verification from a Client Machine:**
Open Command Prompt (CMD) on any client PC and run the following tests:

```cmd
# Test 1: Verify local domain resolution
nslookup Globe.com

# Test 2: Verify external internet resolution via DNS Forwarders
nslookup google.com

```

*If both commands return the correct IP addresses, your DNS infrastructure is professionally deployed and fully operational.*
