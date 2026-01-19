# Group Policy: Granting Local Administrator Rights

> This document explains how and why Group Policy is used to grant local administrator privileges to IT staff.

---

## Objective

Grant local administrator privileges on domain-joined workstations to IT staff using Group Policy, not manual configuration.

This approach:
- Scales across the enterprise
- Is auditable
- Reflects real-world Active Directory administration

---

## Why Group Policy (Not Manual Admin Assignment)

❌ Bad practice:
- Adding users manually to local Administrators on each machine

Problems:
- Does not scale
- Hard to audit
- Easily misconfigured

✅ Best practice:
- Use Group Policy Objects (GPOs)
- Assign privileges to groups, not users

---

## Security Context

In SecureCorp:
- `itadmin` is a domain user
- `IT_Admins` is a security group
- Privileges will be granted to the group, not the user

This creates:
- Centralized control
- A realistic attack surface (group abuse)

---

## Step 1 - Open Group Policy Management

On the Domain Controller:

```
Server Manager → Tools → Group Policy Management
```

Expand:
```
Forest
 └── Domains
     └── securecorp.local
```

---

## Step 2 - Create a New GPO

1. Right-click:
```
SecureCorp → Users
```

2. Select:
```
Create a GPO in this domain, and Link it here
```

3. Name the GPO:
```
GPO_Local_Admins
```

> Linking the GPO here ensures it applies to domain-joined workstations.

---

## Step 3 - Edit the GPO

Right-click `GPO_Local_Admins` → Edit

Navigate to:

```
Computer Configuration
 └── Policies
     └── Windows Settings
         └── Security Settings
             └── Restricted Groups
```

---

## Step 4 - Configure Restricted Groups

1. Right-click **Restricted Groups** → **Add Group**
2. Browse and select:
```
IT_Admins
```

3. Under **This group is a member of**, click **Add**
4. Add:
```
Administrators
```

Click **OK**.

---

## What This Configuration Means

```
IT_Admins → Local Administrators (on target machines)
```

- Members of `IT_Admins` gain **local admin rights**
- Assignment is automatic via Group Policy
- No manual changes on endpoints

---

## Step 5 - Apply Group Policy

On the Windows 10 domain client (or reboot):

```cmd
gpupdate /force
```

Reboot the client system.

---

## Step 6 - Verification

### Check Local Administrators Group

On the Windows 10 client, logged in as `itadmin`:

```text
Win + R → lusrmgr.msc
Groups → Administrators
```

Expected entry:
```
SECURECORP\IT_Admins
```

---

### Verify via Elevated Command Prompt (UAC-aware)

Open Command Prompt as Administrator, then run:

```cmd
net session
```

Expected:
- No "Access is denied" error

> A non-elevated CMD will still show access denied due to UAC.

---

## Security Implications (Why Attackers Care)

This configuration is realistic but risky:
- Compromising one IT account = admin access on many machines
- Attackers often target group membership changes

Later phases will demonstrate:
- Privilege escalation via groups
- Detection of malicious GPO changes

---

## Result

SecureCorp now has:
- Centralized admin delegation
- Group-based privilege control
- A realistic enterprise configuration

---

## Next Step

Proceed to begin Active Directory attack scenarios.

