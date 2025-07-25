# Nickel

**Nickel** is a Windows Server 2022 Datacenter VM that acts as the **primary Domain Controller** (DC) for the `three.local` Active Directory forest. It provides core identity services, DNS resolution, and Group Policy enforcement for the entire homelab.

---

## 🧩 System Overview

- **Hostname**: `nickel`
- **Role**: Domain Controller
- **Domain**: `three.local` (NetBIOS: `THREE`)
- **Static IP**: `192.168.0.250`
- **Hosted on**: Proxmox (Tungsten)
- **Time Zone**: America/Denver (UTC−07:00)

---

## 🔧 Installed Roles & Services

### Active Directory Domain Services (AD DS)
- Forest: `three.local`
- Functional Level: Windows Server 2022
- Admin Account: `Administrator@three.local`
- Delegated Admins: `afugal`, members of `Super-Admins` group

### DNS
- Type: AD-integrated zone (`three.local`)
- Forwarders: `8.8.8.8` (Google), `1.1.1.1` (Cloudflare)
- Dynamic Updates: Secure only
- Clients use `192.168.0.250` as **Primary DNS**

### Time Sync (NTP)
- Uses Windows Time Service synced to internet time servers
- Nickel is the authoritative time source for all domain clients

---

## 🏛️ Active Directory Structure

```plaintext
three.local
├── Computers
│   ├── Windows       ← gold, platinum
│   ├── Linux         ← silicon
│   └── Mac
├── Identities
│   ├── Users
│   │   ├── Admins       ← afugal, salf7200, eli (Junior Dev)
│   │   └── Non-Admins   ← roommates
│   └── Groups          ← Super-Admins, Roommates, LocalAdmin_* groups
```

---

## 🛡️ Group Policy Objects (GPOs)

### 1. **Enable RDP**
- **Link**: Windows OU
- **Purpose**: Allow Remote Desktop on domain-joined Windows machines
- **Settings**:  
  `Computer Configuration → Policies → Admin Templates → Remote Desktop Services → Allow users to connect remotely`

### 2. **Auto Updates**
- **Link**: Windows OU
- **Purpose**: Enforce automatic Windows Updates at 3 AM daily  
- **Settings**:  
  `Windows Update → Configure Automatic Updates: Auto download and schedule install`

### 3. **Push Shared Folder Shortcut**
- **Link**: Users OU
- **Purpose**: Place a desktop shortcut to `\\nickel\Roommates` for all domain users  
- **Method**:  
  `Preferences → Windows Settings → Shortcuts → Location: Desktop`

### 4. **Local Admin Assignment**
- **Link**: Windows OU  
- **Purpose**: Grant select users local admin rights per machine  
- **Method**:  
  `Preferences → Control Panel Settings → Local Users and Groups`  
  - Group: `Administrators (built-in)`
  - Member: `Plat_LocalAdmins`, `Gold_LocalAdmins` (targeted with item-level filtering)

### 5. **Software Restriction Policy (Steam SRP)**
- **Link**: Windows OU  
- **Purpose**: Allow Steam to run without UAC prompt  
- **Status**: **Disabled**  
- **Details (if enabled)**:  
  Path rule: `%ProgramFiles(x86)%\Steam\steam.exe → Unrestricted`

---

## 📂 File Sharing

### `Roommates` Shared Folder
- **Share Name**: `\\nickel\Roommates`
- **Path**: `C:\Roommates`

**Permissions**:
- **Share**:
  - `Everyone`: Read
- **NTFS**:
  - `Roommates` (group): Read & Execute
  - `afugal`: Full Control
  - `Administrators`: Full Control

---

## 🔍 Security & Maintenance

- **System State Backups**: Performed weekly via **Windows Server Backup**
- **Audit Policies**: Enabled for logon/logoff events and directory service changes
- **Monitoring**:
  - `Event Viewer → Directory Service`
  - `DNS Server`
  - `Group Policy`
- **Group Policy Refresh**:  
  Run `gpupdate /force` after GPO changes on client machines

---

Nickel is the backbone of identity and access management in this homelab environment.
