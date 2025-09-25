# 01 – Lab Environment & Specifications

This document describes the **hardware, host configuration, virtual networking, and virtual machine specifications** used to build the **Dunder Mifflin Active Directory Lab**.  
It serves as a reference for replicating or troubleshooting the lab setup.

---

## 1. Host Machine Specifications

| Component           | Details                          |
|---------------------|----------------------------------|
| **Host OS**         | Windows 11 Pro (64-bit)          |
| **Processor (CPU)** | Intel Core i7 / AMD Ryzen 7 (8 cores, 16 threads) |
| **Memory (RAM)**    | 32 GB DDR4                       |
| **Storage**         | 1 TB NVMe SSD + 2 TB HDD         |
| **Virtualization**  | VMware Workstation Pro (v17.x)   |
| **Networking**      | Host-only + NAT adapters         |

> 💡 Note: You can adjust specs depending on your actual hardware, but ensure each VM has enough RAM & disk to run smoothly.

---

## 2. Virtual Network Configuration

The lab uses a **Host-Only network** for isolation and internal communication.  

| Network Type | Purpose                                      |
|--------------|----------------------------------------------|
| **Host-Only**| Isolates lab traffic from the internet; allows host ↔ VM connectivity. |
| **NAT**      | Used temporarily for Windows updates/VMware Tools installation. |

**Subnet (example):**
- Network: `192.168.10.0/24`
- DC01 (Domain Controller): `192.168.10.10`
- RODC01 (Read-Only DC): `192.168.10.11`
- Win10-Client01: `192.168.10.100`
- Default Gateway: N/A (Host-only network)

---

## 3. Virtual Machine Specifications

### 3.1 Domain Controller (DC01)
| Component         | Specification                     |
|-------------------|-----------------------------------|
| **OS**            | Windows Server 2019 Datacenter (GUI) |
| **CPU**           | 2 vCPUs                          |
| **RAM**           | 4 GB                             |
| **Disk**          | 60 GB                            |
| **Network**       | Host-only                        |
| **Role**          | Primary Domain Controller (PDC)  |
| **IP Address**    | 192.168.10.10                    |

### 3.2 Read-Only Domain Controller (RODC01)
| Component         | Specification                     |
|-------------------|-----------------------------------|
| **OS**            | Windows Server 2019 Datacenter (GUI) |
| **CPU**           | 2 vCPUs                          |
| **RAM**           | 4 GB                             |
| **Disk**          | 60 GB                            |
| **Network**       | Host-only                        |
| **Role**          | Read-Only Domain Controller       |
| **IP Address**    | 192.168.10.11                    |

### 3.3 Client Machine (Win10-Client01)
| Component         | Specification                     |
|-------------------|-----------------------------------|
| **OS**            | Windows 10 Enterprise (x64)       |
| **CPU**           | 2 vCPUs                          |
| **RAM**           | 4 GB                             |
| **Disk**          | 40 GB                            |
| **Network**       | Host-only                        |
| **Role**          | Domain-Joined Client              |
| **IP Address**    | 192.168.10.100 (DHCP or static)   |

---

## 4. Domain Information

| Attribute              | Value                       |
|------------------------|-----------------------------|
| **Domain Name**        | `dundermifflin.com`        |
| **Forest Functional Level** | Windows Server 2019    |
| **Root Domain**        | `dundermifflin.com`        |
| **Domain Controller**  | `DC01`                     |
| **RODC**               | `RODC01`                   |
| **Client Machine**     | `Win10-Client01`           |

---

## 5. Lab Topology Diagram 

```text
           [ Host Machine ]
                 |
        -----------------------
        |                     |
   [ DC01 – 192.168.10.10 ]   [ RODC01 – 192.168.10.11 ]
                 |
        -----------------------
                 |
       [ Win10-Client01 – 192.168.10.100 ]
