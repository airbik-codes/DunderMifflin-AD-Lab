# Active Directory Lab Setup – Dunder Mifflin AD Environment

This document provides a detailed step-by-step guide to setting up a simulated **Active Directory (AD) domain environment** using VMware Workstation.  
The lab demonstrates skills in **Windows Server administration, domain controller promotion, OU structuring, group management, and file sharing** — key tasks for a System Administrator.

---

## 1. Lab Specifications

| Component                  | Details                                                                 |
|-----------------------------|-------------------------------------------------------------------------|
| **Virtualization Platform** | VMware Workstation Pro (Host: Windows 11)                              |
| **Domain Name**             | `dundermifflin.com`                                                    |
| **Server OS**               | Windows Server 2019 Datacenter (x64)                                   |
| **Client OS**               | Windows 10 Enterprise (x64)                                            |
| **Network Type**            | Host-Only Network (isolated lab environment)                           |
| **VMs Created**             | 1 Domain Controller (DC01), 1 Read-Only Domain Controller (RODC01), 1 Client (Win10-Client01) |

---

## 2. Virtual Machine Setup

### 2.1 Create a New Virtual Machine
1. Open VMware Workstation → Click **Create a New Virtual Machine**.
2. Select **Typical (Recommended)** → Next.
3. Choose **Installer ISO Image** → point to `Windows Server 2019 ISO`.
4. Configure VM settings:
   - **Name**: `DC01`
   - **Location**: Custom lab folder
   - **RAM**: 4 GB
   - **CPU**: 2 cores
   - **Disk**: 60 GB (single file)
   - **Network**: Host-only adapter
5. Finish and boot the VM.

Repeat similar steps for:
- `RODC01` (another Windows Server 2019 VM)
- `Win10-Client01` (Windows 10 Enterprise ISO)

---

## 3. Windows Server Installation (DC01 & RODC01)

1. Boot VM with Windows Server 2019 ISO.
2. Select **Language, Time, and Keyboard Layout** → Next.
3. Click **Install Now**.
4. Choose **Windows Server 2019 Datacenter (Desktop Experience)**.
5. Accept license terms.
6. Select **Custom Installation** → create new partition → install.
7. After reboot:
   - Set local Administrator password.
   - Log in and install VMware Tools.

Repeat same steps for `RODC01`.

---

## 4. Windows 10 Client Installation (Win10-Client01)

1. Boot with Windows 10 Enterprise ISO.
2. Go through setup wizard (language, region, keyboard).
3. Choose **Custom Install**.
4. Partition and install.
5. Create local admin user (e.g., `labadmin`).
6. Install VMware Tools after login.

---

## 5. Active Directory Domain Services (AD DS) Setup

### 5.1 Install AD DS Role on DC01
1. Open **Server Manager** → **Add Roles and Features**.
2. Role-based → Select **Active Directory Domain Services**.
3. Add required features → Next → Install.

### 5.2 Promote DC01 to Domain Controller
1. After installation → Click **Promote this server to a domain controller**.
2. Choose **Add a new forest** → Root domain: `dundermifflin.com`.
3. Set **DSRM password**.
4. Complete prerequisites check → Install → Reboot.

---

## 6. Read-Only Domain Controller (RODC01)

1. On `RODC01`, install **AD DS role**.
2. In **Server Manager**, choose **Promote to Domain Controller**.
3. Select **Add a domain controller to an existing domain** → `dundermifflin.com`.
4. Choose **Read-only domain controller (RODC)**.
5. Complete wizard → Reboot.

---

## 7. Organizational Unit (OU) Structure

Log into `DC01` → open **Active Directory Users and Computers (ADUC)**.  
Create OUs to mimic corporate structure:

DunderMifflin.com
├── Accounting
│ ├── Users
│ └── Groups
├── Sales
│ ├── Users
│ └── Groups
├── HR
│ ├── Users
│ └── Groups
├── IT
│ ├── Users
│ └── Groups
└── Shared Resources


### Example Users
- `Jim Halpert` → Sales
- `Pam Beesly` → HR
- `Angela Martin` → Accounting
- `Dwight Schrute` → Sales
- `Oscar Martinez` → Accounting
- `Michael Scott` → IT (Domain Admin)

---

## 8. Group Creation & Permissions

1. Inside each department OU, create security groups:
   - `Sales_Users`
   - `HR_Users`
   - `Accounting_Users`
   - `IT_Admins`
2. Add users to corresponding groups.

---

## 9. File Shares Setup

### 9.1 Create Department Folders
On `DC01`:
1. Create `D:\Shares\`.
2. Inside, create folders:
   - `Sales`
   - `HR`
   - `Accounting`
   - `Public`

### 9.2 Configure Share Permissions
1. Right-click folder → **Properties → Sharing → Advanced Sharing**.
2. Check **Share this folder**.
3. Set permissions:
   - `Sales` → Only `Sales_Users`
   - `HR` → Only `HR_Users`
   - `Accounting` → Only `Accounting_Users`
   - `Public` → Authenticated Users (Read/Write)

### 9.3 Configure NTFS Permissions
1. On **Security tab**, remove `Everyone`.
2. Add only respective department groups with **Modify** permission.
3. `Public` folder remains accessible to all.

---

## 10. Domain Join – Client Machine

1. On `Win10-Client01`, open **System Properties** → Change settings.
2. Under **Computer Name → Change**, select **Domain** → enter `dundermifflin.com`.
3. Provide domain admin credentials.
4. Restart client.
5. Login with domain user credentials (e.g., `dundermifflin\jhalpert`).

---

## 11. Verification & Testing

- **ADUC** shows all OUs, groups, and users.
- Clients can log in with domain accounts.
- File shares enforce department-based permissions.
- RODC synchronizes AD data but does not hold writable database.

---

## 12. Skills Demonstrated

- VMware workstation VM deployment  
- Windows Server installation and configuration  
- Active Directory Domain Services installation  
- Domain Controller promotion (Writable + RODC)  
- OU and user management  
- Group-based access control  
- File sharing with NTFS and Share permissions  
- Domain join and authentication testing  

---

## 13. Next Steps (Optional Enhancements)

- Group Policy Objects (GPOs) for security and drive mapping.  
- DNS and DHCP role installation.  
- Implementing login scripts.  
- PowerShell automation for bulk user creation.  

---

📌 **This lab provides a real-world simulation of enterprise AD administration and showcases practical sysadmin skills for portfolio demonstration.**
