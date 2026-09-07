# Network Design

## 1. Overview

The Asteria Digital GmbH homelab uses an isolated virtual IPv4 network to simulate the internal network of a small enterprise.

The network must support Active Directory Domain Services, DNS, DHCP, Windows domain clients, file services and future infrastructure components while remaining isolated from the physical home or university network.

The lab will use network address translation to provide outbound Internet access without directly exposing the virtual enterprise network to the physical LAN.

---

## 2. Network Address

The internal network is:

`10.10.10.0/24`

The corresponding subnet mask is:

`255.255.255.0`

This provides 256 total IPv4 addresses, of which 254 are usable by hosts.

| Property | Value |
|---|---|
| Network Address | `10.10.10.0` |
| First Usable Address | `10.10.10.1` |
| Last Usable Address | `10.10.10.254` |
| Broadcast Address | `10.10.10.255` |
| Subnet Mask | `255.255.255.0` |
| CIDR Prefix | `/24` |

---

## 3. Address Allocation

| Address / Range | Purpose |
|---|---|
| `10.10.10.1` | Planned virtual gateway |
| `10.10.10.10` | DC01 |
| `10.10.10.11` | Reserved for future DC02 |
| `10.10.10.20` | FS01 |
| `10.10.10.21–99` | Static infrastructure addresses |
| `10.10.10.100–200` | DHCP client range |
| `10.10.10.201–254` | Reserved for future use |

Servers use static addresses because infrastructure services such as DNS, Active Directory and file services require predictable network locations.

Employee workstations will obtain their addresses dynamically through DHCP.

---

## 4. Domain Controller

The primary domain controller will use:

**Hostname:** `DC01`

**IPv4 address:** `10.10.10.10`

**Subnet mask:** `255.255.255.0`

**DNS server:** `10.10.10.10`

DC01 will eventually provide:

- Active Directory Domain Services
- DNS
- DHCP

The domain controller uses itself as its primary DNS server because the Active Directory domain relies on DNS records hosted by the Windows DNS service.

---

## 5. DNS Design

The Active Directory DNS namespace will be:

`corp.asteria.local`

Domain clients will use:

`10.10.10.10`

as their DNS server.

This allows internal resources such as:

`dc01.corp.asteria.local`

and:

`fs01.corp.asteria.local`

to be resolved correctly.

External DNS requests can later be forwarded by DC01 to an upstream DNS resolver.

Domain clients should therefore not bypass the Active Directory DNS server by using public DNS servers directly.

---

## 6. DHCP Design

The planned DHCP scope is:

**Scope name:** `ASTERIA-LAN`

**Start:** `10.10.10.100`

**End:** `10.10.10.200`

**Subnet mask:** `255.255.255.0`

The following DHCP options will eventually be configured:

| DHCP Option | Value |
|---|---|
| Router / Gateway | `10.10.10.1` |
| DNS Server | `10.10.10.10` |
| DNS Domain | `corp.asteria.local` |

This will allow Windows workstations to obtain consistent network configuration automatically.

---

## 7. Network Isolation

The enterprise lab should not be directly bridged to the physical network.

Instead, the virtual machines will operate inside an isolated virtual network with NAT-based Internet access.

This design is important because the project will eventually deploy a Windows DHCP server.

Allowing the lab DHCP server to communicate directly with the physical LAN could result in DHCP conflicts with the real router or institutional network.

The target architecture is therefore:

Host Computer → Virtual NAT → Asteria Lab Network → Virtual Machines

---

## 8. DHCP Isolation Requirement

Only one DHCP server should provide addresses inside the Asteria lab network.

When Windows DHCP is introduced, any DHCP service provided automatically by the virtualization platform must either be disabled or separated from the Windows DHCP network.

This ensures that CLIENT01 and other workstations receive their network configuration from DC01.

---

## 9. Planned Systems

| Hostname | Address | Configuration |
|---|---|---|
| DC01 | `10.10.10.10` | Static |
| FS01 | `10.10.10.20` | Static |
| CLIENT01 | `10.10.10.100–200` | DHCP |
| CLIENT02 | `10.10.10.100–200` | DHCP |

---

## 10. Traffic Flow

Internal domain communication remains inside the lab network.

For example:

`CLIENT01 → DC01`

will be used for DNS queries, authentication and Group Policy processing.

Communication with external networks follows:

`CLIENT01 → Gateway → NAT → Internet`

External DNS queries follow:

`CLIENT01 → DC01 → Upstream DNS`

while internal Active Directory DNS queries are resolved directly by DC01.

---

## 11. Design Goals

The network design should provide:

- predictable server addressing
- automatic workstation addressing
- centralized DNS
- Active Directory compatibility
- Internet access
- isolation from the physical LAN
- protection against external DHCP conflicts
- room for additional servers and clients
- a simple topology suitable for troubleshooting