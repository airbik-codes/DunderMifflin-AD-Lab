DC2-Scranton — Overview

VM name: DC2-Scranton

Purpose: Read-Only Domain Controller (RODC) for Scranton site

OS: Windows Server 2022 (Desktop Experience)

vCPU / RAM / Disk: 2 vCPU / 2–4 GB RAM (I recommend 4 GB if host can spare it) / 60 GB HDD

Network: VMware NAT network (lab subnet 192.168.100.0/24)

Static IP: 192.168.100.20

Hostname: DC2-Scranton

Domain: dundermifflin.com (writable DC exists at 192.168.100.10 / DC1-NYHQ)

Snapshots: I take snapshots at the same milestones as DC1: (1) VM created & OS installed, (2) after hostname & IP set, (3) after AD DS role installed & RODC promotion.

1 — Create the VM in VMware Workstation Pro 17

Open VMware Workstation Pro 17 → File → New Virtual Machine.

Select Typical (recommended) → Next.

Choose Installer disc image file (ISO) → point to your Windows Server 2022 ISO → Next.

Guest OS: Microsoft Windows → select Windows Server 2022 (x64) → Next.

Give the VM a name: DC2-Scranton and choose storage location → Next.

Disk capacity: 60 GB (single file is fine).

Click Customize Hardware:

Processors: 2 vCPUs

Memory: 2048 MB (2 GB) — I prefer 4096 MB (4 GB) when possible.

Network Adapter: Set to NAT (same NAT network as DC1 so they can reach each other)

CD/DVD (SATA): attach the Windows Server 2022 ISO.

Finish → Power on the VM.

Screenshot placeholder: ../images/dc2/dc2-vm-settings.png

2 — Install Windows Server 2022

Boot the VM; the Windows installer should start from the ISO.

Set language/time/keyboard → Next → Install now.

Select Windows Server 2022 Standard (Desktop Experience).

Accept license → Custom: Install Windows only (advanced).

Select the 60 GB disk → create partition(s) if desired → Next.

Wait for installation & automatic reboots.

On first login, set the local Administrator password. Use a secure temporary password; do not upload real production passwords to the repo.

Install VMware Tools (VM → Install VMware Tools) to enable better networking and clipboard support.

Screenshot placeholder: ../images/dc2/dc2-install-windows.png
Snapshot: post-install.

3 — Rename the server (computer name) & reboot

I rename the server before making network/DNS changes.

GUI method:

Start → Settings → System → About → Rename this PC.

Enter DC2-Scranton → Restart when prompted.

PowerShell (elevated):

Rename-Computer -NewName "DC2-Scranton" -Restart


Verify after reboot:

hostname


Screenshot placeholder: ../images/dc2/dc2-rename.png
Snapshot: post-rename.

4 — Configure static IP & DNS (point to DC1)

Key point: Before promoting an RODC, the new server’s DNS must be able to resolve and contact the writable DC (DC1). For this reason I set DC2’s preferred DNS server to DC1’s IP (192.168.100.10) during the promotion.

4.1 GUI steps

Settings → Network & Internet → Ethernet (or Control Panel → Network Connections).

Right-click the adapter → Properties → select Internet Protocol Version 4 (TCP/IPv4) → Properties.

Choose Use the following IP address and enter:

IP address: 192.168.100.20

Subnet mask: 255.255.255.0

Default gateway: 192.168.100.1 (VMware NAT gateway)

Preferred DNS server: 192.168.100.10 (DC1-NYHQ)

Alternate DNS: (leave blank or also 192.168.100.10)

OK → Close.

4.2 PowerShell (alternative)
$iface = Get-NetAdapter | Where-Object {$_.Status -eq 'Up'}
New-NetIPAddress -InterfaceIndex $iface.ifIndex -IPAddress 192.168.100.20 -PrefixLength 24 -DefaultGateway 192.168.100.1
Set-DnsClientServerAddress -InterfaceIndex $iface.ifIndex -ServerAddresses 192.168.100.10


Verify:

ipconfig /all
nslookup dc._msdcs.dundermifflin.com 192.168.100.10


Screenshot placeholder: ../images/dc2/dc2-static-ip.png
Snapshot: post-ip-config.

5 — Pre-promotion checks (ensure DC1 is reachable)

Before installing AD DS on DC2, I verify connectivity and that DC1 is healthy:

# From DC2 (or the host), test reachability
Test-Connection -ComputerName 192.168.100.10 -Count 4

# Query DC locator via DNS (from DC2)
nltest /dsgetdc:dundermifflin.com

# From DC1 ensure DC1's AD and DNS are healthy (run on DC1)
dcdiag /v
repadmin /replsummary


If any DNS or connectivity issues show up, I fix them before proceeding.

6 — Install AD DS Role on DC2
6.1 Using Server Manager (GUI)

Open Server Manager → Manage → Add Roles and Features.

Role-based → Next → Select DC2-Scranton → Next.

Under Server Roles, check Active Directory Domain Services → click Add Features when prompted (includes management tools).

Next → Install. Wait until the installation finishes.

Screenshot placeholder: ../images/dc2/dc2-add-roles.png

7 — Promote DC2 as a Read-Only Domain Controller (RODC)

Important: I must use Domain Admin credentials (or a delegated account) to add a domain controller to an existing domain. The wizard will offer to make this server an RODC.

7.1 GUI Promotion Steps

After AD DS role installation completes → click the Notifications flag in Server Manager and select Promote this server to a domain controller.

Choose Add a domain controller to an existing domain → enter domain: dundermifflin.com.

Credentials: provide a Domain Admin account (e.g., dundermifflin\Administrator) when prompted.

Under Domain Controller Options:

Select: Read-only domain controller (RODC).

Global Catalog (GC): leave unchecked or checked depending on design (I usually leave GC unchecked for RODC unless you need local GC).

DNS: you can choose to install DNS on the RODC or not. I typically do not install DNS on the RODC if the writable DC is local and reachable. If you expect clients at Scranton to rely on local name resolution when WAN is down, install DNS.

Choose Replication source: select DC1-NYHQ (the writable DC) as the source.

Password Replication Policy (PRP): you can choose to preconfigure which account credentials are allowed to be cached on this RODC. I leave it default (no accounts cached) and configure PRP after installation using ADUC.

Review the summary, confirm prerequisites pass, then click Install. The server will reboot when promotion completes.

Screenshot placeholder: ../images/dc2/dc2-promote-rodc.png

7.2 PowerShell Promotion (automated)

I sometimes use PowerShell to automate promotions. Run elevated PowerShell on DC2:

# Install AD DS role (if not already)
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Secure DSRM password (do NOT hardcode in public repos)
$dsrm = Read-Host -AsSecureString "Enter DSRM password for RODC"

# Promote as RODC - point to the replication source if desired
Install-ADDSDomainController `
  -DomainName "dundermifflin.com" `
  -ReadOnlyReplica `
  -SafeModeAdministratorPassword $dsrm `
  -ReplicationSourceDC "DC1-NYHQ.dundermifflin.com" `
  -Credential (Get-Credential)


The command prompts for Domain Admin credentials and restarts the server after promotion.

8 — Post-promotion verification

After reboot and signing back in (use Administrator or a domain admin account), I verify the RODC’s status and replication.

8.1 Confirm RODC is created and read-only:
Import-Module ActiveDirectory
Get-ADDomainController -Filter * | Format-Table Name,HostName,IsReadOnly,IPv4Address

# Check specifically
(Get-ADDomainController -Identity "DC2-Scranton").IsReadOnly


Expected: IsReadOnly = True for DC2-Scranton.

8.2 Replication status
# Show replication summary from the new RODC
repadmin /replsummary

# Show inbound replication for the RODC
repadmin /showrepl DC2-Scranton


Check for any replication errors and resolve them via event logs or network/firewall checks.

8.3 DNS & SRV records

If DNS was installed on RODC, verify the zone exists and SRV records are present. If not, ensure DC1 has SRV records for DC2 in DNS.

# On DC1 (or from DC2 pointing to DC1)
nslookup -type=SRV _ldap._tcp.dc._msdcs.dundermifflin.com 192.168.100.10

8.4 Check services

Make sure the NTDS and DNS (if installed) services are running:

Get-Service ntds,dns


Screenshot placeholder: ../images/dc2/dc2-verification.png

Snapshot: post-RODC-promotion.

9 — Password Replication Policy (PRP) — quick notes

An RODC does not cache passwords by default. If you want certain accounts (service accounts or selected users) to be cached on the RODC, configure the Password Replication Policy:

GUI (ADUC method):

On a writable DC, open Active Directory Users and Computers (ADUC).

Navigate to Domain Controllers → right-click the computer DC2-Scranton → Properties → Password Replication Policy tab.

Add accounts/groups to the Allowed RODC Password Replication Group if you want them cached locally. Use caution — caching passwords increases risk if the RODC is compromised.

Important: For a lab, I usually leave PRP unchanged (no cached passwords) to demonstrate security best practice.

10 — Troubleshooting (common RODC issues & fixes)

RODC cannot contact replication source

Check DNS: nslookup DC1-NYHQ 192.168.100.10 from DC2.

Verify network connectivity: Test-Connection 192.168.100.10.

Ensure firewall is not blocking AD ports (TCP 389, 636, 3268, 3269, 445, RPC dynamic ports). Temporarily allow or disable firewall for lab testing.

Replication failures

On DC2 run repadmin /showrepl and review errors.

Check System and Directory Service event logs in Event Viewer.

Time sync problems

Verify time: w32tm /query /status. Sync time with DC1 if needed to avoid Kerberos issues.

DNS resolution problems

Confirm DC2 is using DC1 as preferred DNS at promotion time. If you installed DNS locally on DC2, ensure zones replicated correctly.
11 — Quick command cheat-sheet (copyable)
# Rename machine (run elevated)
Rename-Computer -NewName "DC2-Scranton" -Restart

# Set static IP and DNS (run elevated)
$iface = Get-NetAdapter | Where-Object {$_.Status -eq 'Up'}
New-NetIPAddress -InterfaceIndex $iface.ifIndex -IPAddress 192.168.100.20 -PrefixLength 24 -DefaultGateway 192.168.100.1
Set-DnsClientServerAddress -InterfaceIndex $iface.ifIndex -ServerAddresses 192.168.100.10

# Install AD DS role
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promote to RODC (example)
$dsrm = Read-Host -AsSecureString "Enter DSRM password"
Install-ADDSDomainController -DomainName "dundermifflin.com" -ReadOnlyReplica -SafeModeAdministratorPassword $dsrm -ReplicationSourceDC "DC1-NYHQ.dundermifflin.com" -Credential (Get-Credential)

# Verify RODC status
Import-Module ActiveDirectory
Get-ADDomainController -Filter * | Format-Table Name,HostName,IsReadOnly,IPv4Address

# Replication checks
repadmin /replsummary
repadmin /showrepl DC2-Scranton
dcdiag /v

12 — Final notes & best practices

Keep the writable DC (DC1-NYHQ) online and healthy while adding RODCs and during initial testing.

Use snapshots generously in the lab so you can roll back misconfigurations.

Do not cache more passwords on the RODC than necessary — follow least privilege for PRP.

Document which accounts/groups you allow to replicate to the RODC and why.

After promotion, periodically review Event Viewer on the RODC for replication and service issues.
