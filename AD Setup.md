# Windows Server & Active Directory Setup (SecureCorp)

> This document walks through **installing Windows Server and deploying Active Directory**.
> Explains **how** and **why** each step is performed.

---

## Objective

Deploy a **Domain Controller (DC)** that will:
- Provide centralized authentication
- Run DNS for the domain
- Manage users, groups, and computers

The domain created here is:
```
securecorp.local
```

---

## Lab Context

This Windows Server VM will be live on the LAN behind the Ubuntu firewall.

| Component | Value |
|-----------|-------|
| Server name | SecureCorp-DC |
| IP address | 192.168.175.5 |
| Gateway | 192.168.175.1 (Ubuntu Firewall) |
| DNS | 192.168.175.5 (self) |

---

## Step 1 - Create the Windows Server VM

Create a new VM in VirtualBox:

```
Name: SecureCorp-DC
Type: Microsoft Windows
Version: Windows Server (64-bit)
RAM: 4 GB (minimum)
CPU: 2 cores (suggested atleast 4 cores)
Disk: 50 GB (VDI, dynamic)
```

Attach a Windows Server 2022 ISO.

---

## Step 2 - Network Adapter Configuration

Power OFF the VM and configure networking.

### Adapter 1 (ONLY adapter)
```
Attached to: Host-only Adapter
Name: VirtualBox Host-Only Ethernet Adapter
Cable connected: ✔
```

❌ Do NOT add NAT
❌ Do NOT add multiple adapters

The firewall already handles internet access.

---

## Step 3 - Install Windows Server

Boot the VM and begin installation.

During setup:
- Select:
  ```
  Windows Server Standard (Desktop Experience)
  ```
- Set an Administrator password

After login, "Server Manager" will open automatically.

---

## Step 4 - Set Static IP Address (MANDATORY)

A Domain Controller must use a static IP.

On SecureCorp-DC:
In Control Panel → Network & Internet → Network Connections
Ethernet → Properties → IPv4

Set:
```
IP address:      192.168.175.5
Subnet mask:     255.255.255.0
Default gateway: 192.168.175.1
Preferred DNS:   192.168.175.5
```

> DNS points to itself - this is required for AD.

Reboot the server.

---

## Step 5 - Verify Network Connectivity

In Command Prompt:

```cmd
ping 192.168.175.1
ping 8.8.8.8
```

Both should succeed.

---

## Step 6 - Install Active Directory Domain Services (AD DS)

Open:
```
Server Manager → Manage → Add Roles and Features
```

Choose:
- Role-based installation
- Select current server

Check:
```
☑ Active Directory Domain Services
```

When prompted, click Add Features.

Install and wait for completion.

---

## Step 7 - Promote Server to Domain Controller

After installation:

1. Click the warning flag in Server Manager
2. Select promote this server to a domain controller

Choose:
```
Add a new forest
Root domain name: securecorp.local
```

---

## Step 8 - Domain Controller Options

Set:
```
Forest functional level: Windows Server 2016 or higher
Domain functional level: Windows Server 2016 or higher
☑ DNS Server
☑ Global Catalog
```

Set a DSRM password.

Ignore DNS delegation warning and continue.

---

## Step 9 - Complete Promotion

Leave default paths for:
- Database
- Logs
- SYSVOL

Click Install.

The server will reboot automatically.

---

## Step 10 - Post-Installation Verification

After reboot, log in as:
```
SECURECORP\Administrator
```

Verify identity:

```cmd
whoami
hostname
```

Expected:
```
securecorp\administrator
SecureCorp-DC
```

---

## Step 11 - Verify DNS & Domain

Run:

```cmd
ipconfig /all
```

Confirm:
- IP: 192.168.175.5
- DNS: 127.0.0.1 / ::1
- DNS suffix: securecorp.local

Test name resolution:

```cmd
nslookup securecorp.local
```

Expected:
```
Address: 192.168.175.5
```

---

## Common Pitfalls

- ❌ Using DHCP on a Domain Controller
- ❌ Pointing DNS to 8.8.8.8
- ❌ Promoting server before setting static IP
- ❌ Powering off DC while testing domain DNS

---

## Result

At this stage, SecureCorp has:
- A functioning Domain Controller
- Active Directory Domain Services
- Internal DNS for the domain

---

## Next Step

Proceed to OU design, users, and groups creation.

