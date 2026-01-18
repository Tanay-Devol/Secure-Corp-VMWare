# Ubuntu Firewall Setup (SecureCorp - Step 1)

> This document is written for starting this project.
> Follow the steps in order. Skipping the verifications in between will lead to consequences + errors in later stages so recommended not to skip them.

---

## Objective

Set up an Ubuntu Linux machine as a multi-interface enterprise firewall that will:
- Act as a **gateway** between internal machines and the internet
- Provide **LAN and DMZ segmentation**
- Perform **NAT (Network Address Translation)**
- Forward **traffic securely**
- **Persist configuration** after reboot

---

## Lab Design Overview

The firewall has three network interfaces:

| Interface | Purpose | Network |
|-----------|---------|---------|
| `enp0s3` | WAN | Internet (NAT) |
| `enp0s8` | LAN | 192.168.175.0/24 |
| `enp0s9` | DMZ | 172.16.50.0/24 |

The firewall IPs will be:
- LAN Gateway: `192.168.175.1`
- DMZ Gateway: `172.16.50.1`

---

## Prerequisites

- Oracle VirtualBox installed
- Ubuntu Server or Ubuntu Desktop ISO
- Basic Linux terminal knowledge

---

## Step 1 - Create the Firewall VM

Create a new VM in VirtualBox:

Name: SecureCorp-FW
Type: Linux
Version: Ubuntu (64-bit)
RAM: 4 GB or more
CPU: 3 cores or more
Disk: 30 GB (VDI, dynamic)

Install Ubuntu.

---

## Step 2 - Configure Network Adapters (VirtualBox)

Power OFF the firewall VM.

Configure **three adapters**:

### Adapter 1 (WAN)
Attached to: NAT
Cable connected: ✔

### Adapter 2 (LAN)
Attached to: Host-only Adapter
Name: VirtualBox Host-Only Ethernet Adapter (The host-only network which has 192.168.175.0/24 ipv4 prefix)
Cable connected: ✔

### Adapter 3 (DMZ)
Attached to: Host-only Adapter
Name: VirtualBox Host-Only Ethernet Adapter #2 (The host-only network which has 172.16.50.0/24 ipv4 prefix)
Cable connected: ✔

Power ON the VM after this.

---

## Step 3 - Identify Network Interfaces

Inside Ubuntu, open a terminal and run:

```bash
ip a
```

Identify:
- WAN interface → usually `enp0s3`
- LAN interface → usually `enp0s8`
- DMZ interface → usually `enp0s9`

> Interface names may differ slightly depending on system.

---

## Step 4 - Assign Static IPs (LAN & DMZ)

The firewall must have static IPs on internal networks.

### Configure LAN

```bash
sudo nmcli con add type ethernet ifname enp0s8 con-name LAN ipv4.method manual ipv4.addresses 192.168.175.1/24
sudo nmcli con up LAN
```

### Configure DMZ

```bash
sudo nmcli con add type ethernet ifname enp0s9 con-name DMZ ipv4.method manual ipv4.addresses 172.16.50.1/24
sudo nmcli con up DMZ
```

Verify:

```bash
ip a
```

Expected:
- `enp0s8` → 192.168.175.1
- `enp0s9` → 172.16.50.1

---

## Step 5 - Enable IP Forwarding

Linux does not route traffic by default.

Edit sysctl config:

```bash
sudo nano /etc/sysctl.conf
```

Add or uncomment:

```
net.ipv4.ip_forward=1
```

Apply:

```bash
sudo sysctl -p
```

Verify:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

Expected output:
```
1
```

---

## Step 6 - Configure Firewall & NAT Rules

### Enable NAT (LAN + DMZ → Internet)

```bash
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```

### Allow Forwarding

```bash
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
sudo iptables -A FORWARD -i enp0s9 -o enp0s3 -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s9 -m state --state ESTABLISHED,RELATED -j ACCEPT
```

---

## Step 7 - Verify Firewall Rules

```bash
sudo iptables -t nat -L -n -v
sudo iptables -L FORWARD -n -v
```

You should see:
- `MASQUERADE` on WAN interface
- FORWARD rules for LAN and DMZ

---

## Step 8 - Persist Firewall Rules

Install persistence:

```bash
sudo apt update
sudo apt install iptables-persistent -y
```

When prompted:
```
Save current IPv4 rules? → YES
Save IPv6 rules? → NO
```

Rules are saved to:
```
/etc/iptables/rules.v4
```

---

## Step 9 - Reboot Test

```bash
sudo reboot
```

After reboot:

```bash
ip a
sudo iptables -t nat -L -n -v
```

If rules and IPs persist, the firewall is complete.

---

## Result

Now we have:
- A functional firewall
- NAT and routing enabled
- Persistent configuration
- A stable base for Active Directory and security testing

---

## IMPORTANT:-
1) NO MODIFICATIONS SHALL BE DONE IN THIS FIREWALL FURTHER IN THE PROJECT.
2) FIREWALL SHOULD BE ALWAYS TURNED ON IN THE BACKGROUND FURTHER IN THE PROJECT.

---

## Next Step

Proceed to **Windows Server + Active Directory setup**.
