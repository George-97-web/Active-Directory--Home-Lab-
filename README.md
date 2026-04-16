# 🖥️ Active Directory Home Lab

A fully functional Active Directory home lab built using Oracle VirtualBox, Windows Server 2019, and Windows 10. This project simulates a real enterprise network environment including domain services, DHCP, DNS, NAT routing, and client-server communication.

---

## 🧰 Lab Environment

| Component | Details |
|---|---|
| Virtualization Platform | Oracle VirtualBox |
| Server OS | Windows Server 2019 |
| Client OS | Windows 10 |
| Domain Name | mydomain.com |
| Server Hostname | DC |

---

## 🌐 Network Topology

```
  [Internet]
      |
  [VirtualBox NAT]
      |
  NIC 1 - INTERNET (NAT)
  IP: 10.0.2.15 | Gateway: 10.0.2.2
      |
  [Windows Server 2019 - DC]
      |
  NIC 2 - INTERNAL (Internal Network)
  IP: 172.16.0.1
      |
  [Windows 10 Client]
  IP: 172.16.0.100 (via DHCP)
```

### Network Settings Summary

| Setting | Value |
|---|---|
| Internal NIC IP | 172.16.0.1 |
| DHCP Scope | 172.16.0.100 – 172.16.0.200 |
| Default Gateway (Client) | 172.16.0.1 |
| DNS Server (Client) | 172.16.0.1 |
| DNS Forwarder (Server) | 8.8.8.8 / 8.8.4.4 |

---

## ⚙️ Setup Steps

### 1. VirtualBox Configuration
- Created two VMs: one for Windows Server 2019 (DC) and one for Windows 10 (Client)
- Configured two network adapters on the Server:
  - **Adapter 1:** NAT (for internet access)
  - **Adapter 2:** Internal Network (for client communication)
- Configured one network adapter on the Client:
  - **Adapter 1:** Internal Network (same as server's Adapter 2)

### 2. Windows Server 2019 Installation
- Installed Windows Server 2019
- Renamed the server to **DC**
- Assigned a static IP to the internal NIC: `172.16.0.1`

### 3. Active Directory Domain Services (AD DS)
- Installed the **AD DS** role via Server Manager
- Promoted the server to a **Domain Controller**
- Created a new forest with domain name: `mydomain.com`

### 4. DHCP Configuration
- Installed the **DHCP Server** role
- Created a new IPv4 scope:
  - Range: `172.16.0.100 – 172.16.0.200`
  - Default Gateway (Option 003): `172.16.0.1`
  - DNS Server (Option 006): `172.16.0.1`
- Authorized the DHCP server in Active Directory

### 5. DNS Configuration
- DNS role installed automatically with AD DS
- Added DNS Forwarders:
  - `8.8.8.8` (Google Primary)
  - `8.8.4.4` (Google Secondary)

### 6. RRAS / NAT Configuration
- Installed the **Remote Access** role with the **Routing** service
- Configured **Routing and Remote Access (RRAS)** as a NAT router:
  - Public Interface: **INTERNET** (NIC 1 - NAT adapter)
  - Private Interface: **INTERNAL** (NIC 2 - Internal adapter)
- This allows the client to reach the internet through the server

### 7. Windows 10 Client Setup
- Installed Windows 10
- Set network adapter to **Internal Network**
- Confirmed DHCP lease obtained from server
- Joined the client to **mydomain.com**

---

## 🐛 Issues Encountered & Solutions

### Issue 1: Wrong Adapter Selected in RRAS NAT
**Problem:** During RRAS NAT configuration, the INTERNAL adapter was selected as the public interface instead of the INTERNET adapter.  
**Symptom:** Client had a valid IP and gateway but no internet access.  
**Fix:** Opened RRAS → NAT properties → corrected the public interface to the **INTERNET** adapter.

---

### Issue 2: DNS Forwarder Typo
**Problem:** DNS forwarder was set to `172.20.1.1` (a non-existent IP) instead of a public DNS server.  
**Symptom:** `ping 8.8.8.8` worked but `ping google.com` failed with "could not find host."  
**Fix:** Removed the incorrect forwarder and added `8.8.8.8` and `8.8.4.4`.

---

### Issue 3: Missing Default Gateway on Server's NAT NIC
**Problem:** The server's INTERNET NIC had no default gateway configured.  
**Symptom:** Server could not ping external IPs; `ipconfig` showed no default gateway.  
**Fix:** Set the gateway on the INTERNET NIC to `10.0.2.2` (VirtualBox's built-in NAT gateway).

---

### Issue 4: Windows 10 Firewall Blocking ICMP
**Problem:** The server could not ping the client by IP.  
**Symptom:** Ping requests timed out from server to `172.16.0.100`.  
**Fix:** Enabled the **File and Printer Sharing (Echo Request - ICMPv4-In)** inbound firewall rule on the Windows 10 client, or via CMD:
```
netsh advfirewall firewall add rule name="Allow ICMPv4" protocol=icmpv4:8,any dir=in action=allow
```

---

## ✅ Verification Checklist

| Test | Result |
|---|---|
| Client gets IP from DHCP | ✅ |
| Client can ping server (172.16.0.1) | ✅ |
| Client can ping internet by IP (8.8.8.8) | ✅ |
| Client can ping internet by name (google.com) | ✅ |
| Server can ping client (172.16.0.100) | ✅ |
| Client joined to domain (mydomain.com) | ✅ |

---

## 📚 References

- [Oracle VirtualBox Documentation](https://www.virtualbox.org/wiki/Documentation)
- [Microsoft AD DS Installation Guide](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services)
- [Microsoft DHCP Server Setup](https://learn.microsoft.com/en-us/windows-server/networking/technologies/dhcp/dhcp-deploy-wps)
- [Microsoft RRAS Documentation](https://learn.microsoft.com/en-us/windows-server/remote/remote-access/ras-gateway/ras-gateway)

---

## 👤 Author

Built as a home lab project to develop hands-on skills in Windows Server administration, Active Directory, networking, and troubleshooting.
