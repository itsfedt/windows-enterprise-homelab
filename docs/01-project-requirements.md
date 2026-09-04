# Asteria Digital GmbH -- Windows Enterprise Homelab

## 1. Project Overview

This project implements a virtualized Windows enterprise infrastructure for asteria Digital Gmbh, a fictional German IT consulting and software development company based in Düsseldorf.

Asteria Digital Gmbh has approximately 60 employees distributed across Management, human resources, Finances, IT, Engineering and Sales.

The purpose of the project is to develop practical experience designing, deploying, administrating, securing and troubleshooting common Microsoft enterprise infrastructure technologies.

The environment will initially focus on traditional on-premises Windows infrastructure before being extended with Microsft Entra ID and Microsoft Intune.

## 2. Business Scenario

Asteria Digital GmbH has grown from a small startup into a company with approx. 60 employees. Its Windows computers are currently administered individually and rely primarily on local user accounts. As teh company grows, this approach has becaome difficult to manage. There company therefore requires centralized identity management, network services, access control, workstation configuration and security policy enforcement.

## 3. Business Requirements

### BR-01 —— Centralized Identtiy Management

Employees must be able to authenticate using centrally managed company accounts.

### BR-02 —— Organizational Administration

Employees and computers must be logically organized according to company departments and administrative requirements.

### BR-03 —— Centralized Name Resolution

Internal servers and workstations must be able to resolve corporate hostnames through a centrally managed DNS service.

### BR-04 —— Automatic Network Configuration

Employee workstations should automatically obtain appropriate network settings rather than requiring manual IP configuration.

### BR-05 —— Centralized Workstation Configuration

IT administrators must be able to centrally enforce selected security and workstation configuration policies.

### BR-06 —— Department-Based Resource Access

Access to corporate resources must follow the principles of least privilege. For example,HR employees should be ablöe to access HR resources while employees outside HR should not automatically recieve the same access.

### BR-07 —— File Services

Departments require centrally managed shared folders for storing and accessing internal files.

### BR-08 —— User Lifecycle Management 

The IT department requires repeatable processes for employee onboarding, role changes, password administration and employee offboarding.

### BR-09 —— Automation
Repetitive administrative tasks should be automated where practical using PowerShell. 

### BR-10 —— Endpoint Management

The infrastructure should later be extended with Microsoft Entra Id and Microsoft Intune to explore modern clud-based Windows endpoint management.

## 4. Planned Infrastructure 

| System            | Hostname | IP Address  | Role                 |
| :------           | :-----   | :-------    | :--------            |
| Domain Controller | DC01     | 10.10.10.10 | AD DS, DNS, DHCP     |
| File Server       | FS01     | 10.10.10.20 | File Services        |
| Windows Client    | CLIENT01 | DHCP        | Domain workstation   |
| Windows Client    | CLIENT02 | DHCP        | intune/Entra testing |

## 5. Network

Network: 
 > 10.10.10.0/24 

Planned DHCP range: 
> 10.10.10.100 - 10.10.10.200

Primary DNS server: 
> 10.10.10.10


## 6. Active Directory

Lab domain:
> corp.asteria.local

NetBIOS domain:
> ASTERIA

Planned organizational departments:
- Management
- Human Resources
- Finances
- IT
- Engineering
- Sales


## 7. Planned Technologies

The project will explore:

- Windows Server
- Windows 11
- Active Directory Domain Services
- DNS
- DHCP
- Group Policy
- NTFS permissions
- Windows File Services
- Powershell
- Git/Github
- Microsoft Entra ID
- Microsoft intune


## 8. Security Objectives

The environment will demonstrate several fundamental enterprise security concepts, including:
- least privilege
- role-based acess
- separation of privileged accounts
- centralized password policies
- account lockout policies
- endpoint firewall configuration
- Microsoft Defender configuration
- controlled resource access
- employee onboarding and offboarding procedures

## 9. Automation Objectives

PowerShell will be used progressively to automate administrative taks such as:
- creating Organizational units
- creating Active Directory users
- creating securty groups
- assigning group memberships
- disabling users
- resetting passwords
- exporting Active Directory information
- performing basic infrastructure health checks

## 10. Project Phases

### Phase 1 —— On-Premises Infrastructure

1. Virtualization
2. Network configuration
3. Windows Server installation
4. Active Directory 
5. DNS
6. DHCP
7. Windows domain clients
8. Group Policy 
9. File services
10. Access control 
11. PowerShell automation
12. Security hardening
13. Testing and troubleshooting

### Phase 2 —— Modern Endpoint Management

1. Microsoft Entra ID
2. Microsoft Intune
3. Windows device enrollment
4. Configuration profiles
5. Compliance policies  
6. Application development 
7. Comparison between traditional GPO-based and modern Intune-based management

## 11. Learning Objectives

By completing this project, I aim to develop hands-on knowledge of Windows enterprise administration and better understand how identity, networking, authorization, security poilicies and endpoint management interact in a real organizational environment. The project will also emphasize infrastructure documentation, reproducibility, troubleshooting and administrative automation rather than focusing only on succesful installation.