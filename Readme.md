# 🏢 Dunder Mifflin Active Directory Lab

This project is a **simulated enterprise Active Directory environment** built in **VMware Workstation Pro 17** on a Windows 11 host.  
It is themed on *The Office (US)*, modeling Dunder Mifflin Paper Company with HQ in New York and a branch in Scranton, PA.  

The lab demonstrates:
- Installing and configuring **Active Directory Domain Services (AD DS)**.
- Deploying a **multi-DC environment** with a writable DC and a Read-Only DC (RODC).
- Creating **Organizational Units (OUs)**, **users**, and **security groups**.
- Implementing **file sharing with permissions** using security groups.
- Testing **domain join and login** from client machines.

---

## ⚙️ Lab Specifications

- **Host Machine**: Dell G7 7588 (Windows 11, 16 GB RAM)  
- **Virtualization Platform**: VMware Workstation Pro 17  
- **Network Mode**: NAT (192.168.100.0/24)  

### Domain
- **Forest Root Domain**: `dundermifflin.com`

### Domain Controllers
1. **DC1-NYHQ**  
   - Windows Server 2022  
   - Role: Writable Domain Controller (HQ)  
   - Specs: 2 vCPUs, 2 GB RAM, 60 GB HDD  
   - IP: `192.168.100.10`

2. **DC2-Scranton**  
   - Windows Server 2022  
   - Role: Read-Only Domain Controller (RODC) (Scranton Branch)  
   - Specs: 2 vCPUs, 2 GB RAM, 60 GB HDD  
   - IP: `192.168.100.20`

### Client
- **Client1-Scranton**  
  - Windows 10 Enterprise Evaluation  
  - Specs: 2 vCPUs, 2 GB RAM, 60 GB HDD  
  - IP: `192.168.100.50`  
  - Purpose: Domain join + login testing  

---

## 🗂 Organizational Units (OUs)

- **New York (HQ)** OU  
  - `Computers`  
  - `Users` → Departments: Finance, Executive  

- **Scranton (Branch)** OU  
  - `Computers`  
  - `Users` → Departments: Sales, HR, IT, Accounting, Admin, Customer Service, Warehouse, QA, Supplier Relations  

- **Groups** OU  
  - Departmental Security Groups (Sales, HR, IT, Accounting, etc.)  

---

## 👥 Users & Groups (The Office Characters)

Examples:
- **Executives**: David Wallace (CFO), Jan Levinson (CEO)  
- **Scranton Sales**: Jim Halpert, Dwight Schrute, Andy Bernard, Phyllis Lapin, Stanley Hudson  
- **HR**: Toby Flenderson, Holly Flax  
- **Admin**: Pam Beesly (Office Admin), Erin Hannon (Receptionist)  
- **Accounting**: Angela Martin, Oscar Martinez, Kevin Malone  
- **Warehouse**: Darryl Philbin  
- **IT Staff**: Sadiq, Nick  
- **Others**: Ryan Howard (Temp), Kelly Kapoor (Customer Service), Creed Bratton (QA), Meredith Palmer (Supplier Relations), Interns (Eric, Megan, Maurie)  

---

## 📂 File Sharing & Permissions

- Departmental folders created and shared.  
- Security groups assigned with NTFS & share permissions.  
- Example: **Jim Halpert (Sales)** logs in from Client1 and accesses Sales shared folder.  

---

## 🖼 Screenshots

Screenshots are included in the `/docs/screenshots/` directory, covering:
1. VM creation in VMware Workstation  
2. OS installation and static IP configuration  
3. AD DS role installation and promotion to DC  
4. OU and user/group creation  
5. Folder sharing and permission setup  
6. Domain join and login verification  

---

## 🎯 Learning Outcomes

- Demonstrates end-to-end **Active Directory deployment**.  
- Shows **enterprise-like topology design** with DCs and OUs.  
- Validates **group-based access control** using shared folders.  
- Strengthens skills for **System Administrator** career path.  

---

## 📎 References

- Microsoft Docs: [Active Directory Domain Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/ad-ds-overview)  
- VMware Docs: [Workstation Pro User Guide](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)  

---
