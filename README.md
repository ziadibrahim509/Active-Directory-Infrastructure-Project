# 🖥️ Active Directory Infrastructure Implementation

> **MCSA Project** — A fully functional enterprise-grade Windows Server environment built from scratch, featuring multi-site Active Directory, centralized management, and automated deployment.

**Supervised by:** Eng. Mohamed Aboshely | **Date:** January 2026  
**Team:** Aesha Abdelwahid · Marten Wafik · Mohamed Mostafa · Ziad Ibrahim · Mohamed Elsayed

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Network Topology](#network-topology)
- [Infrastructure Components](#infrastructure-components)
- [Services Implemented](#services-implemented)
- [Group Policy Configuration](#group-policy-configuration)
- [User Mobility & Remote Access](#user-mobility--remote-access)
- [Automated Deployment (WDS)](#automated-deployment-wds)

---

## Project Overview

This project demonstrates a complete **Windows Server 2022** infrastructure simulating a real enterprise environment with multiple branches and centralized administration. It covers domain architecture, enterprise service integration, and scalable deployment strategies.

**Key Design Goals:**

| Goal | Implementation |
|---|---|
| High Availability | Primary + Additional Domain Controllers |
| Branch Separation | Child domains for Alexandria & Ismailia |
| Security | Read-Only Domain Controller (RODC) at branch sites |
| Scalability | AD Forest structure with DNS delegation |
| Centralized Control | Group Policy Objects (GPOs) |
| Automation | Windows Deployment Services (WDS) for 50 clients |

---

## Network Topology

```
                        ITI.Local (Forest Root)
                       ┌─────────────────────┐
                       │  DC1 - Primary DC    │  10.182.21.12
                       │  DC2 - Additional DC │  10.182.21.191
                       │  DC3 - RODC          │  10.182.21.249
                       │  DC6 - WDS Server    │  10.182.21.212
                       └─────────────────────┘
                              /           \
               Alex.ITI.Local             Ism.ITI.Local
              ┌────────────┐             ┌────────────┐
              │ DC4        │ 10.182.21.166│ DC5        │ 10.182.21.80
              │ PC4-FIN    │ 10.182.21.166│ PC5-HR     │ 10.182.21.141
              └────────────┘             └────────────┘

  Web Server: 10.182.21.127  |  DNS+DHCP: 10.182.21.73
```

**Domain Structure:** `ITI.Local` (root) → `Alex.ITI.Local` · `Ism.ITI.Local` (children)

---

## Infrastructure Components

### Domain Controllers

| Server | Role | IP Address | OS |
|--------|------|------------|----|
| DC1 | Primary Domain Controller | 10.182.21.12 | Windows Server 2022 |
| DC2 (ADDC) | Additional Domain Controller | 10.182.21.191 | Windows Server 2022 |
| DC3 | Read-Only Domain Controller (RODC) | 10.182.21.249 | Windows Server 2022 |
| DC4 | Child DC — Alexandria Branch | 10.182.21.166 | Windows Server 2022 |
| DC5 | Child DC — Ismailia Branch | 10.182.21.80 | Windows Server 2022 |
| DC6 | WDS Server | 10.182.21.212 | Windows Server 2022 |

### Primary Domain Controller (DC1)
- Hosts the `ITI.Local` forest root domain
- Provides authentication, authorization, and DNS
- Holds the initial Active Directory database
- Replicates directory data to all additional controllers

### Additional Domain Controller (DC2)
- Ensures **high availability** — services remain online if DC1 fails
- Performs **load balancing** for login and directory queries
- Maintains a live replicated copy for **disaster recovery**

### Read-Only Domain Controller (DC3)
- Deployed for secure branch-office authentication
- AD database is **read-only** — local changes cannot replicate back
- **Password Replication Policy (PRP):** `help@ITI.local` can authenticate locally even when the link to HQ is down

### Child Domains
**Alexandria Branch (DC4 — `Alex.ITI.Local`)**
- Dedicated OU structure for the FIN department
- PC4 joined to `Alex.ITI.Local`
- Local admins delegated limited rights

**Ismailia Branch (DC5 — `Ism.ITI.Local`)**
- Two-way transitive trust with parent domain
- DNS delegation configured
- OU structure for Ismailia users and computers

---

## Services Implemented

### 🌐 IIS — Web Hosting
- IIS role installed and configured via Server Manager
- Hosts two websites: `www.web1.com` and `www.web2.com`
- Web server IP: `10.182.21.127`

### 📁 FTP
- FTP Server role added to IIS
- Basic Authentication integrated with Active Directory credentials
- Supports centralized file sharing and remote upload/download

### 🔍 DNS
- Forward Lookup Zones configured for `web1.com` and `web2.com`
- DNS delegation set up for child domains
- Integrated with Active Directory for reliable service discovery

### 📡 DHCP
- Configured on the DNS+DHCP server (`10.182.21.73`)
- Automatically assigns IPs, DNS server, and gateway to clients
- Scope configured for the `10.0.0.0` network
- WDS server also configured with DHCP for PXE boot support

---

## Group Policy Configuration

GPOs are applied across the domain to enforce security and standardize the environment:

| Policy | Target | Effect |
|--------|--------|--------|
| Logon Hours + Workstation Restriction | User A (`A@ITI.local`) | Can only log on to PC1; blocked on Fridays |
| Removable Storage Deny | User C (`c@ITI.local`) | No access to flash/USB drives |
| Control Panel Block | User C | Cannot open Control Panel |
| Desktop Wallpaper | User C | Forced to ITI logo wallpaper |
| Software Deployment | PC2 | WinRAR deployed silently via `.msi` package |
| RDP Delegation | User B (`B@iti.local`) | Can log in remotely to DC1 without being a domain admin |

---

## User Mobility & Remote Access

### Roaming Profiles
User `A@Ism.ITI.Local` has a roaming profile configured so their desktop, documents, and settings follow them across machines:
- Profile path set to `\\Server\Profiles\%username%` in Active Directory
- Verified working on **PC1** (Primary Domain), **PC4**, and **PC5** (Child Domain)

### Remote Desktop Protocol (RDP)
- RDS configured for secure remote access to workstations
- User **D** (local user on PC6) can manage the web server remotely with administrative privileges
- Allows IT helpdesk to troubleshoot without physical access

---

## Automated Deployment (WDS)

Windows Deployment Services implemented on **DC6** (`10.182.21.212`):

- Clients boot over the network via **PXE (Preboot Execution Environment)** — no USB or DVD needed
- DHCP role co-located on WDS server to assign IPs during the boot process
- **50 Windows client machines** deployed automatically
- **50 user accounts** bulk-created in Active Directory via PowerShell
- Deployed machines automatically join the `ITI.Local` domain and inherit all GPOs and security settings

---

## 🔧 Technologies Used

- Windows Server 2022 Standard Evaluation
- Active Directory Domain Services (AD DS)
- DNS, DHCP, IIS, FTP
- Group Policy Management (GPO)
- Remote Desktop Services (RDS)
- Windows Deployment Services (WDS) + PXE
- VMware Workstation / ESXi 6.x (lab virtualization)
- PowerShell (bulk user/computer creation)
