# 03 — Virtualisation Environment Setup

## 1. Introduction

The purpose of this phase was to create the virtualisation layer on which the Asteria Digital GmbH enterprise homelab will operate.

Before deploying services such as Active Directory Domain Services, DNS or DHCP, I first needed a controlled and isolated virtual environment in which the servers and client machines could communicate with each other without interfering with the physical network of the host computer.

For this reason, I treated the virtualisation layer as part of the infrastructure design rather than simply as a mechanism for running virtual machines.

The main objectives of this phase were therefore to:

- configure the virtualisation platform,
- create an isolated internal network for the Asteria environment,
- provide controlled Internet connectivity through NAT,
- create the first Windows Server virtual machine,
- install Windows Server,
- prepare the server for its future role as the first domain controller,
- and establish a clean baseline before introducing Active Directory.

At the end of this phase, the environment contains the first server, `DC01`, but no Active Directory, DNS or DHCP roles have yet been installed.

---

## 2. Choice of Virtualisation Platform

I selected **Microsoft Hyper-V** as the virtualisation platform for this project.

Since the homelab is based primarily on Microsoft enterprise technologies, I considered Hyper-V to be a suitable choice because it integrates directly with Windows and provides native support for virtual machines, virtual switches, PowerShell-based management and checkpointing.

Another reason for this decision was that I wanted the project to include not only Windows Server administration, but also some exposure to infrastructure automation through PowerShell.

Although Hyper-V Manager provides a graphical interface for creating and administering virtual machines, I intentionally chose to perform part of the configuration using PowerShell.

This allowed me to better understand the underlying objects being created and made the environment easier to reproduce later.

---

## 3. Target Virtual Network Architecture

The Asteria Digital GmbH lab was designed around the following internal IPv4 network:

`10.10.10.0/24`

The network is isolated from the physical LAN of the host machine.

The planned architecture is:

```text
                         Internet
                            |
                            |
                       Windows Host
                            |
                      Windows NAT
                            |
                       10.10.10.1
                            |
                     ASTERIA-LAB
                  Hyper-V Internal Switch
                            |
          +-----------------+-----------------+
          |                 |                 |
        DC01               FS01            CLIENT01
    10.10.10.10         10.10.10.20          DHCP
```

At this stage, only `DC01` has been deployed.

The remaining systems will be introduced incrementally during later phases of the project.

---

## 4. Choice of an Internal Hyper-V Switch

Hyper-V supports different types of virtual switches, including External, Internal and Private switches.

I deliberately chose an **Internal switch** for the Asteria environment.

An External switch would have connected the virtual machines directly to the physical network. While this could simplify Internet connectivity, it would not provide the level of isolation I wanted for this lab.

This was particularly important because I plan to deploy a Windows DHCP server later in the project. Connecting that DHCP service directly to the physical LAN could potentially create conflicts with an existing home, university or corporate DHCP server.

A Private switch would have provided even stronger isolation, but it would have prevented direct communication between the host system and the virtual machines.

An Internal switch therefore represented the most appropriate compromise because it allows communication between:

- the host machine,
- DC01,
- future servers,
- future client machines,

while keeping the Asteria network separated from the physical LAN.

I created the switch using PowerShell:

```powershell
New-VMSwitch `
    -SwitchName "ASTERIA-LAB" `
    -SwitchType Internal
```

I then verified the configuration with:

```powershell
Get-VMSwitch
```

The expected switch type is:

```text
Name           SwitchType
----           ----------
ASTERIA-LAB    Internal
```

---

## 5. Host-Side Network Interface

When an Internal Hyper-V switch is created, Windows also creates a corresponding virtual network adapter on the host.

In this environment, the adapter is:

```text
vEthernet (ASTERIA-LAB)
```

I assigned the address:

`10.10.10.1/24`

to this interface.

This address acts as the host-side interface of the Asteria virtual network and later serves as the default gateway used by the virtual machines.

The configuration was performed using:

```powershell
New-NetIPAddress `
    -InterfaceAlias "vEthernet (ASTERIA-LAB)" `
    -IPAddress 10.10.10.1 `
    -PrefixLength 24
```

I verified the address afterwards with:

```powershell
Get-NetIPAddress `
    -InterfaceAlias "vEthernet (ASTERIA-LAB)"
```

The important result is that the host now has a direct presence on the lab network at:

```text
10.10.10.1
```

---

## 6. NAT Configuration

The Internal switch alone provides connectivity between systems inside the virtual network, but it does not automatically provide access to the Internet.

To allow the virtual machines to access external resources while remaining isolated from the physical LAN, I configured Network Address Translation on the Windows host.

I created the NAT configuration using:

```powershell
New-NetNat `
    -Name "ASTERIA-NAT" `
    -InternalIPInterfaceAddressPrefix "10.10.10.0/24"
```

I verified the NAT configuration with:

```powershell
Get-NetNat
```

The relevant configuration is:

```text
Name: ASTERIA-NAT
InternalIPInterfaceAddressPrefix: 10.10.10.0/24
```

This means that systems inside the Asteria network can later send traffic through the host computer toward external networks without being directly attached to the physical network.

The resulting traffic path is therefore:

```text
Virtual Machine
      |
      v
ASTERIA-LAB
      |
      v
10.10.10.1
      |
      v
Windows NAT
      |
      v
Host physical interface
      |
      v
Internet
```

---

## 7. DHCP Design Consideration

An important design decision during this phase was to avoid using the virtualisation platform as the DHCP provider.

The project requires DC01 to become the authoritative DHCP server for the Asteria lab in a later phase.

Therefore, I intentionally designed the environment so that client addressing will eventually follow this path:

```text
CLIENT01
    |
    | DHCP request
    v
DC01
    |
    v
Windows DHCP Server
```

This avoids having two competing DHCP services inside the same virtual network.

It also makes the environment more realistic because DHCP configuration will become part of the Windows Server infrastructure rather than being hidden inside the hypervisor.

---

## 8. VM Storage Organisation

I decided to keep virtual machine files separate from the Git repository.

The GitHub repository is intended to contain documentation, scripts, configuration examples and screenshots, while virtual disks and installation media remain local to the host computer.

The VM storage structure is organised approximately as follows:

```text
C:\VMs\
└── Asteria\
    ├── DC01\
    ├── FS01\
    └── CLIENT01\
```

For DC01, the virtual disk is stored under:

```text
C:\VMs\Asteria\DC01\
```

This separation is useful for several reasons:

- virtual disks are too large for normal Git version control,
- operating system images should not be stored in the repository,
- infrastructure documentation remains independent from the actual local VM files,
- the repository can later be cloned without requiring the virtual machines themselves.

---

## 9. DC01 Virtual Machine Design

`DC01` is intended to become the first domain controller of the Asteria Digital GmbH environment.

Before installing any server roles, I created the VM with enough resources to support Windows Server and the planned infrastructure services.

The virtual machine configuration is:

| Property | Configuration |
|---|---|
| Hostname | `DC01` |
| Hyper-V Generation | Generation 2 |
| Virtual CPUs | 2 |
| Startup Memory | 4 GB |
| Virtual Disk | 60 GB |
| Network | `ASTERIA-LAB` |
| Operating System | Windows Server 2025 |
| Firmware | UEFI |

I selected a **Generation 2** virtual machine because it uses modern virtual hardware and supports UEFI-based booting.

For this project, the allocated resources are sufficient because the server will initially host only a small lab environment.

The configuration may be adjusted later if additional services require more resources.

---

## 10. Creation of DC01 Using PowerShell

Rather than creating the VM through Hyper-V Manager, I created `DC01` using PowerShell.

I made this decision intentionally because I wanted the deployment process to be more transparent and reproducible.

The VM was created with:

```powershell
New-VM `
    -Name "DC01" `
    -Generation 2 `
    -MemoryStartupBytes 4GB `
    -NewVHDPath "C:\VMs\Asteria\DC01\DC01.vhdx" `
    -NewVHDSizeBytes 60GB `
    -SwitchName "ASTERIA-LAB"
```

The number of virtual processors was then configured with:

```powershell
Set-VMProcessor `
    -VMName "DC01" `
    -Count 2
```

The Windows Server installation ISO was attached with:

```powershell
Add-VMDvdDrive `
    -VMName "DC01" `
    -Path "C:\ISO\WindowsServer2025.iso"
```

I verified that the virtual machine had been created successfully using:

```powershell
Get-VM -Name "DC01"
```

I also verified the network connection with:

```powershell
Get-VMNetworkAdapter -VMName "DC01"
```

The VM network adapter must be connected to:

```text
ASTERIA-LAB
```

Using PowerShell for this step was particularly useful because the commands can later be reused or converted into a more complete automation script.

---

## 11. Windows Server Installation

After creating the virtual machine, I started `DC01` and booted from the Windows Server installation ISO.

For this initial domain controller, I selected a Windows Server edition with **Desktop Experience**.

Although Server Core is commonly used in enterprise environments, I chose Desktop Experience because this project is primarily intended for learning.

The graphical management tools will make it easier to explore services such as:

- Server Manager,
- Active Directory Users and Computers,
- DNS Manager,
- DHCP Manager,
- Group Policy Management.

Once I am comfortable with these components, Server Core could be introduced as an additional learning exercise.

The operating system was installed on the 60 GB virtual disk.

After installation, I configured a local Administrator password specifically for the lab environment.

No credentials are stored in the Git repository.

---

## 12. Initial Server Configuration

After the Windows Server installation completed, I performed several initial configuration tasks before installing any infrastructure roles.

### 12.1 Hostname

The server initially had an automatically generated Windows computer name.

Since its intended role had already been defined in the architecture, I renamed it to:

```text
DC01
```

using:

```powershell
Rename-Computer `
    -NewName "DC01" `
    -Restart
```

After the restart, I verified the name with:

```powershell
hostname
```

The expected result was:

```text
DC01
```

---

### 12.2 Time Zone

Because Asteria Digital GmbH is located in Germany, I verified the Windows time zone.

The desired configuration is:

```text
W. Europe Standard Time
```

The configuration can be checked with:

```powershell
Get-TimeZone
```

and, when required, set using:

```powershell
Set-TimeZone `
    -Id "W. Europe Standard Time"
```

Correct system time will become particularly important after Active Directory is deployed because Kerberos authentication depends on synchronized clocks.

---

### 12.3 Windows Updates

Before promoting the server to a domain controller, I installed available Windows updates.

I preferred to perform this step while the server was still a normal standalone Windows Server system.

This reduces the probability of introducing unnecessary operating system changes immediately after the Active Directory deployment.

---

## 13. Network Configuration Status

At the end of this phase, the logical addressing plan for DC01 is already defined:

```text
Hostname:       DC01
IPv4 address:   10.10.10.10
Prefix:         /24
Gateway:        10.10.10.1
```

However, I deliberately left the full DC01 network configuration and validation for the following phase.

This separation allows me to verify routing, DNS configuration, Internet connectivity and static addressing before installing Active Directory.

The next stage will therefore focus specifically on configuring:

```text
10.10.10.10/24
```

as the static address of DC01.

---

## 14. Baseline Checkpoint

Before introducing Active Directory or other infrastructure services, I created a Hyper-V checkpoint.

The checkpoint represents a clean Windows Server installation that can be used as a recovery point during experimentation.

I created it using:

```powershell
Checkpoint-VM `
    -Name "DC01" `
    -SnapshotName "Baseline-Windows-Installed"
```

My planned checkpoint strategy is:

```text
01 - Baseline Windows Installed
02 - Network Configured
03 - Before AD Promotion
04 - Active Directory Operational
```

For this homelab, checkpoints provide a convenient way to recover from configuration mistakes.

However, I do not consider checkpoints to be a replacement for a proper enterprise backup strategy.

This distinction is important because checkpoints are primarily a virtualisation management feature rather than a full disaster recovery solution.

---

## 15. Repository Protection

Because the project is being documented publicly, I added common virtual machine and credential-related file types to `.gitignore`.

The repository excludes:

```gitignore
# Virtual machine files
*.vhd
*.vhdx
*.avhdx
*.vmcx
*.vmrs

# Installation media
*.iso

# Credentials and certificates
*.key
*.pfx
*.pem
.env
```

This helps prevent large binary files, operating system images or sensitive information from accidentally being committed to GitHub.

---

## 16. Validation

After completing the virtualisation setup, I performed a number of checks to confirm that the environment had been created as intended.

### Hyper-V Switch

```powershell
Get-VMSwitch
```

Expected:

```text
ASTERIA-LAB    Internal
```

### Host Virtual Interface

```powershell
Get-NetIPAddress `
    -InterfaceAlias "vEthernet (ASTERIA-LAB)"
```

Expected IPv4 address:

```text
10.10.10.1/24
```

### NAT

```powershell
Get-NetNat
```

Expected NAT network:

```text
10.10.10.0/24
```

### Virtual Machine

```powershell
Get-VM -Name "DC01"
```

Expected result:

```text
DC01
```

### VM Network

```powershell
Get-VMNetworkAdapter `
    -VMName "DC01"
```

Expected switch:

```text
ASTERIA-LAB
```

### Server Hostname

Inside DC01:

```powershell
hostname
```

Expected:

```text
DC01
```

---

## 17. Validation Summary

| Validation Check | Expected Result | Status |
|---|---|---|
| Hyper-V available | Operational | PASS |
| `ASTERIA-LAB` switch | Internal | PASS |
| Host lab interface | `10.10.10.1/24` | PASS |
| NAT configuration | `10.10.10.0/24` | PASS |
| DC01 VM | Created | PASS |
| VM generation | Generation 2 | PASS |
| CPU allocation | 2 vCPU | PASS |
| Memory allocation | 4 GB | PASS |
| Virtual disk | 60 GB | PASS |
| Network adapter | `ASTERIA-LAB` | PASS |
| Windows Server | Installed | PASS |
| Hostname | `DC01` | PASS |
| Baseline checkpoint | Created | PASS |

---

## 18. Current Infrastructure State

At the end of this phase, the implemented environment is:

```text
                       Internet
                           |
                           |
                    Windows Host
                           |
                    ASTERIA-NAT
                           |
                     10.10.10.1
                           |
                    ASTERIA-LAB
                   10.10.10.0/24
                           |
                           |
                     +-----------+
                     |   DC01    |
                     |-----------|
                     | Windows   |
                     | Server    |
                     | 2025      |
                     |-----------|
                     | AD DS: No |
                     | DNS:   No |
                     | DHCP:  No |
                     +-----------+
```

The absence of Active Directory, DNS and DHCP at this stage is intentional.

The virtualisation and operating system layers are being validated before infrastructure roles are introduced.

---

## 19. Lessons Learned

This phase helped me better understand that virtualisation design is directly connected to network and infrastructure design.

One of the most important decisions was choosing an Internal Hyper-V switch instead of attaching the virtual machines directly to the physical network.

This allows the lab to remain isolated and will later make it possible to safely run my own DHCP service without affecting other devices outside the lab.

I also gained practical experience using Hyper-V through PowerShell.

Creating DC01 through PowerShell made the configuration more explicit than relying only on the graphical wizard. Each major property, including memory, virtual disk size, VM generation, processor count and virtual switch assignment, was deliberately specified.

Another important lesson was the distinction between internal communication and Internet connectivity.

The Hyper-V switch provides communication inside the Asteria environment, while NAT provides a controlled path toward external networks.

Finally, I established a habit that I intend to maintain throughout the project:

```text
Design
   ↓
Configure
   ↓
Validate
   ↓
Document
```

Rather than assuming that a configuration works because no error was displayed, I will use PowerShell commands, client tests and screenshots to verify each infrastructure component.

---

## 20. Next Step

The next phase will focus on the network configuration of DC01.

The server will be configured with the planned static address:

```text
10.10.10.10/24
```

using:

```text
Gateway: 10.10.10.1
```

The configuration will then be validated using tools such as:

```powershell
ipconfig
Get-NetIPAddress
Get-NetRoute
ping
Test-NetConnection
Resolve-DnsName
```

Only after the server network has been verified will DC01 be promoted to the first Active Directory domain controller of the Asteria Digital GmbH environment.