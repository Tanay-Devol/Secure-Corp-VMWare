# Organization Units, Users & Groups Design

> This document explains **how and why Active Directory objects are structured** in SecureCorp.

---

## Objective

Design a **realistic Active Directory structure** that:
- Separates users by role
- Uses groups for privilege management
- Avoids insecure default containers
- Mirrors how real enterprises deploy AD

This structure is intentionally simple but vulnerable, so attacks and detections can be demonstrated later.

---

## Why OU Design Matters (Important Concept)

In Active Directory:
- Organizational Units (OUs) control:
  - Group Policy application
  - Delegation of administration
  - Scope of security impact

Poor OU design leads to:
- Privilege sprawl
- Weak security boundaries
- Easy lateral movement for attackers

> Attackers love flat AD environments.

---

## Default Containers vs Custom OUs

Active Directory creates these by default:
- Builtin
- Users
- Computers
- Domain Controllers

⚠️ These are containers, not proper OUs (except Domain Controllers).

Best practice:
- ❌ Do NOT use default Users/Computers for real objects
- ✅ Create custom OUs for enterprise management

---

## SecureCorp OU Structure

### Parent OU

Create a single enterprise root:

```
SecureCorp
```

This acts as the **administrative boundary** for policies and permissions.

---

### Sub-OUs

Inside `SecureCorp`, create:

```
SecureCorp
 ├── IT
 ├── HR
 ├── Users
 ├── Servers
 └── Service_Accounts
```

### Purpose of Each OU

| OU | Purpose |
|----|---------|
| IT | IT staff and admin accounts |
| HR | HR users and resources |
| Users | Standard employee accounts |
| Servers | Domain-joined servers |
| Service_Accounts | Non-human accounts (future use) |

---

## Step 1 - Create Organizational Units

On the Domain Controller:

1. Open **Active Directory Users and Computers** (`dsa.msc`)
2. Right-click `securecorp.local`
3. Select New → Organizational Unit

Create:
```
SecureCorp
```

Then right-click `SecureCorp` and create all sub-OUs listed above.

> Leave "Protect from accidental deletion"enabled for all OUs.

---

## Step 2 - Create Domain Users

Navigate to:
```
SecureCorp → Users
```

Create the following users:

### Standard User
```
Name: John Doe
Username: jdoe
Password: Password@123
```

### HR User
```
Name: Alice HR
Username: ahr
Password: Password@123
```

### IT Administrator (Privileged User)
```
Name: IT Admin
Username: itadmin
Password: Password@123
```

### Password Options (IMPORTANT)

When creating users:

```
✔ User must change password at next logon
✘ User cannot change password
✘ Password never expires
✘ Account is disabled
```
---

## Step 3 - Create Security Groups

Navigate to:
```
SecureCorp → IT
```

Create:

### IT_Admins
- Group scope: Global
- Group type: Security

### HR_Users
- Group scope: Global
- Group type: Security

---

## Step 4 - Assign Group Membership

Add users to groups:

| User | Group |
|------|-------|
| itadmin | IT_Admins |
| ahr | HR_Users |
| jdoe | None (standard user) |

> Privileges are assigned to groups, not users.

---

## Step 5 - Why Group-Based Privileges Are Critical

Best practice:
- ❌ Do NOT make users admins directly
- ✅ Add groups to privileged roles

Benefits:
- Easier auditing
- Cleaner privilege management
- More realistic attack surface

This is also what attackers target:
- Group membership abuse
- Privilege escalation via groups

---

## Step 6 - Initial Privilege Assignment

At this stage:
- `itadmin` is NOT yet a local admin on workstations
- Privileges will be granted later using Group Policy

This separation is intentional and realistic.

---

## Verification

On the Windows 10 domain client:

- Login as `securecorp\jdoe`
- Confirm normal (non-admin) access

Then:
- Login as `securecorp\itadmin`
- Confirm domain login works

---

## Common Mistakes

- Creating users in default `Users` container
- Granting admin rights directly to user accounts
- Mixing servers and users in the same OU

---

## Result

SecureCorp Active Directory now has:
- Clean OU hierarchy
- Role-based users
- Group-driven privilege model
- A realistic baseline for attacks and defense

---

## Next Step

Proceed to Group Policy configuration for administrative privileges.

