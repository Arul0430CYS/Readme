# Active Directory Penetration Testing & Reconnaissance

A practical collection of commands and workflows for **Active Directory reconnaissance and enumeration** during authorized security assessments and lab environments.

The focus is on command execution, enumeration techniques, important options, expected results, practical workflows, and troubleshooting.

> **Authorization:** Use these commands only against systems, networks, domains, or applications that you own or have explicit permission to assess.

---

## Overview

Active Directory (AD) is Microsoft's centralized directory service for identity and access management in enterprise environments.

During an authorized security assessment, AD reconnaissance can help identify:

* Domain Controllers
* Domain users and groups
* DNS and LDAP information
* SMB shares
* Service accounts and SPNs
* Kerberos configuration issues
* ACL and SYSVOL permissions
* Domain computers
* Trust relationships
* Potential attack paths

---

## Tools

* [Nmap](https://nmap.org/)
* DNSRecon
* LDAPSearch
* NetExec
* Responder
* PowerView
* PowerSploit
* SharpHound
* BloodHound
* ADRecon
* PowerShell

---

## Installation

### Kali Linux / Debian-based systems

```bash
sudo apt update && sudo apt install dnsrecon nmap ldap-utils responder netexec -y
```

Installs the main DNS, network, LDAP, SMB enumeration, and assessment tools available through the system repositories.

### PowerView

```powershell
Import-Module .\PowerView.ps1
```

Loads the PowerView module into the current PowerShell session.

### PowerSploit

```powershell
Import-Module .\PowerSploit.ps1
```

Loads the PowerSploit module.

### SharpHound

```powershell
. .\SharpHound.ps1
```

Loads the SharpHound PowerShell collector.

---

## Basic Syntax

### DNSRecon

```bash
dnsrecon -r <IP_RANGE> -n <NAMESERVER>
```

Performs reverse DNS lookups across the specified IP range using the specified DNS server.

### LDAPSearch

```bash
ldapsearch -x -H ldap://<TARGET_IP> -b "<BASE_DN>" "<FILTER>"
```

Queries an LDAP service for directory objects and attributes.

### NetExec

```bash
netexec smb <TARGET_IP_OR_RANGE> -u '<USER>' -p '<PASSWORD>' [options]
```

Performs SMB-based enumeration using supplied credentials or an appropriate unauthenticated configuration.

### PowerView

```powershell
Get-Domain[Object] -Identity <Target> [Flags]
```

PowerView cmdlets can query domain users, groups, computers, ACLs, and other Active Directory objects.

---

# Commands & Execution

## DNS & Network Reconnaissance

### Reverse DNS Enumeration

```bash
dnsrecon -r <IP_RANGE> -n <DNS_SERVER>
```

**Purpose:** Performs reverse DNS lookups to map IP addresses to hostnames.

**Important options:**

* `-r` — Specifies the target IP range.
* `-n` — Specifies the DNS server.

**Example:**

```bash
dnsrecon -r 10.10.10.0/24 -n 10.10.10.1
```

**Expected result:** A list of IP addresses and corresponding hostnames.

---

### DHCP Server Discovery

```bash
sudo nmap --script broadcast-dhcp-discover
```

**Purpose:** Uses the Nmap DHCP discovery script to identify DHCP servers and obtain network configuration information.

**Important option:**

* `--script broadcast-dhcp-discover` — Executes the DHCP discovery NSE script.

**Example:**

```bash
sudo nmap --script broadcast-dhcp-discover -e eth0
```

**Expected result:** DHCP-related information such as network configuration, gateway, and DNS information.

---

## LDAP Enumeration

### LDAP Service Enumeration

```bash
nmap -n -sV --script "ldap* and not brute" <DC_IP>
```

**Purpose:** Identifies LDAP services and performs LDAP-related enumeration without running brute-force scripts.

**Important options:**

* `-n` — Disables reverse DNS resolution.
* `-sV` — Enables service/version detection.
* `--script` — Selects the LDAP-related NSE scripts while excluding brute-force scripts.

**Example:**

```bash
nmap -n -sV --script "ldap* and not brute" 10.10.10.10
```

**Expected result:** LDAP service information, supported capabilities, and directory-related metadata.

---

### LDAP Root DSE Query

```bash
ldapsearch -H ldap://<DC_IP> -s base -b "" "objectClass=*" -LLL
```

**Purpose:** Queries the LDAP Root DSE for base-level directory service metadata.

**Important options:**

* `-H` — Specifies the LDAP server URI.
* `-s base` — Searches only the base object.
* `-b ""` — Uses an empty base DN to query Root DSE.
* `-LLL` — Produces concise LDIF output.

**Example:**

```bash
ldapsearch -H ldap://10.10.10.10 -s base -b "" "objectClass=*" -LLL
```

**Expected result:** Information such as naming contexts and supported LDAP capabilities.

---

### Authenticated LDAP User Enumeration

```bash
ldapsearch -LLL -x \
-H ldap://<DC_HOST> \
-D "<USER>@<DOMAIN>" \
-w "<PASSWORD>" \
-b "dc=<DOMAIN_PART>,dc=<DOMAIN_PART>" \
"(objectClass=user)" \
sAMAccountName userPrincipalName memberOf
```

**Purpose:** Authenticates to LDAP and retrieves domain user information.

**Important options:**

* `-x` — Uses simple authentication.
* `-H` — Specifies the LDAP server.
* `-D` — Specifies the authentication identity.
* `-w` — Supplies the password.
* `-b` — Defines the LDAP search base.

**Expected result:** User accounts, UPNs, and group membership information.

> Avoid storing real passwords directly in commands or Git repositories. Prefer secure credential handling or placeholders.

---

# SMB Enumeration

### SMB Share & User Enumeration

```bash
netexec smb <TARGET_RANGE> -u '<USER>' -p '<PASSWORD>' --shares --users
```

**Purpose:** Enumerates SMB shares and domain users using authorized credentials.

**Important options:**

* `-u` — Specifies the username.
* `-p` — Specifies the password.
* `--shares` — Enumerates SMB shares.
* `--users` — Enumerates domain users.

**Example:**

```bash
netexec smb 10.10.10.0/24 -u '<USER>' -p '<PASSWORD>' --shares --users
```

**Expected result:** SMB hosts, accessible shares, permissions, and user information.

---

### SMB Password Policy Enumeration

```bash
netexec smb <TARGET_IP> -u '' -p '' --pass-pol
```

**Purpose:** Attempts to retrieve domain password policy information through an unauthenticated SMB session where the environment permits it.

**Important option:**

* `--pass-pol` — Requests password and account lockout policy information.

**Example:**

```bash
netexec smb 10.10.10.10 -u '' -p '' --pass-pol
```

**Expected result:** Password complexity, minimum password length, and account lockout information when exposed.

---

# PowerView Enumeration

### Find Domain Computers by Operating System

```powershell
Get-DomainComputer -OperatingSystem "Windows Server 2019 Standard" |
Select-Object Name, DnsHostName
```

**Purpose:** Finds domain-joined computers matching a specified operating system.

**Important option:**

* `-OperatingSystem` — Filters computers according to their OS attribute.

**Example:**

```powershell
Get-DomainComputer -OperatingSystem "*Server 2022*"
```

**Expected result:** Computer names and DNS hostnames matching the filter.

---

### Find Kerberoastable Users

```powershell
Get-DomainUser -SPN
```

**Purpose:** Identifies domain users with Service Principal Names (SPNs).

**Important option:**

* `-SPN` — Filters users associated with SPNs.

**Example:**

```powershell
Get-DomainUser -SPN |
Select-Object sAMAccountName, servicePrincipalName
```

**Expected result:** Accounts associated with registered service principals.

---

### Extract TGS Tickets in Hashcat Format

```powershell
Get-DomainUser -SPN |
Get-DomainSPNTicket -Format Hashcat |
Select-Object -ExpandProperty Hash
```

**Purpose:** Requests service tickets for SPN-associated accounts and formats the resulting ticket material for Hashcat-compatible processing.

**Important option:**

* `-Format Hashcat` — Formats the ticket output for Hashcat.

**Example:**

```powershell
Get-DomainUser -Identity "<SERVICE_ACCOUNT>" |
Get-DomainSPNTicket -Format Hashcat
```

---

### Find AS-REP Roastable Users

```powershell
Get-DomainUser -PreauthNotRequired |
Select-Object SamAccountName, UserPrincipalName
```

**Purpose:** Identifies accounts configured without Kerberos pre-authentication.

**Important option:**

* `-PreauthNotRequired` — Filters accounts where pre-authentication is not required.

**Example:**

```powershell
Get-DomainUser -PreauthNotRequired
```

**Expected result:** Accounts configured with the relevant Kerberos pre-authentication setting.

---

### Find Computers with Unconstrained Delegation

```powershell
Get-DomainComputer -Unconstrained
```

**Purpose:** Identifies domain computers configured for unconstrained delegation.

---

# BloodHound & SharpHound

### SharpHound Data Collection

```powershell
.\SharpHound.exe --CollectionMethods All -d <DOMAIN> --ZipFileName loot.zip
```

**Purpose:** Collects Active Directory information for analysis in BloodHound.

**Important options:**

* `--CollectionMethods All` — Enables the available collection methods specified by SharpHound.
* `-d` — Specifies the domain.
* `--ZipFileName` — Defines the output ZIP filename.

**Example:**

```powershell
.\SharpHound.exe --CollectionMethods Default,LoggedOn -d <DOMAIN>
```

**Expected result:** A ZIP archive containing collected JSON data for BloodHound analysis.

---

# ADRecon

### Automated Active Directory Reconnaissance

```powershell
.\ADRecon.ps1 -Method LDAP -DomainController <DC_IP> -Credential (Get-Credential)
```

**Purpose:** Collects Active Directory information such as users, groups, computers, GPOs, DNS records, and password policies.

**Important options:**

* `-Method LDAP` — Uses LDAP-based collection.
* `-DomainController` — Specifies the Domain Controller.
* `-Credential` — Requests credentials securely through PowerShell.

**Expected result:** Structured reports containing collected Active Directory information.

---

# SYSVOL & ACL Auditing

### Audit SYSVOL Permissions

```powershell
Get-Acl -Path "\\<DC_IP>\sysvol\<DOMAIN>\Policies" |
Format-List
```

**Purpose:** Retrieves ACL information from the SYSVOL policy directory.

**Important options:**

* `-Path` — Specifies the SYSVOL path.
* `Format-List` — Displays ACL information in a readable property format.

**Example:**

```powershell
Get-Acl -Path "\\<DC_IP>\sysvol\<DOMAIN>\Policies" |
Select-Object -ExpandProperty Access
```

**Expected result:** File and directory permissions associated with the SYSVOL policy files.

---

# WMI Active Directory Query

### Query Domain Password & Lockout Properties

```powershell
Get-WmiObject -Namespace root\directory\ldap -Class ds_domain |
Select-Object ds_lockoutduration, ds_lockoutthreshold, ds_maxpwdage, ds_minpwdage
```

**Purpose:** Queries domain-related properties exposed through the specified WMI namespace.

**Important options:**

* `-Namespace` — Specifies the WMI namespace.
* `-Class` — Specifies the WMI class.

**Expected result:** Domain password and account lockout properties.

---

# Practical Workflow

## Reconnaissance

Start by identifying network information and potential Domain Controllers.

```bash
sudo nmap --script broadcast-dhcp-discover
```

```bash
dnsrecon -r <IP_RANGE> -n <DNS_SERVER>
```

Use the results to identify relevant network infrastructure and DNS information.

---

## LDAP Enumeration

Identify LDAP services and collect exposed directory metadata.

```bash
nmap -n -sV --script "ldap* and not brute" <DC_IP>
```

```bash
ldapsearch -H ldap://<DC_IP> -s base -b "" "objectClass=*" -LLL
```

If authorized credentials are available, perform authenticated directory enumeration:

```bash
ldapsearch -LLL -x \
-H ldap://<DC_IP> \
-D "<USER>@<DOMAIN>" \
-w "<PASSWORD>" \
-b "dc=<DOMAIN_PART>,dc=<DOMAIN_PART>" \
"(objectClass=user)" \
sAMAccountName userPrincipalName memberOf
```

---

## SMB Enumeration

```bash
netexec smb <TARGET_RANGE> -u '<USER>' -p '<PASSWORD>' --shares --users
```

Where appropriate in an authorized lab, test whether unauthenticated SMB policy information is exposed:

```bash
netexec smb <TARGET_IP> -u '' -p '' --pass-pol
```

---

## Active Directory Object Enumeration

Use PowerView to identify relevant domain objects.

```powershell
Get-DomainUser -SPN
```

```powershell
Get-DomainUser -PreauthNotRequired
```

```powershell
Get-DomainComputer -Unconstrained
```

```powershell
Get-DomainComputer -OperatingSystem "*Server*"
```

---

## Attack Path Mapping

Collect authorized Active Directory relationship data using SharpHound:

```powershell
.\SharpHound.exe --CollectionMethods All -d <DOMAIN> --ZipFileName full_ad_audit.zip
```

The resulting data can be imported into BloodHound for relationship and attack-path analysis.

---

# Common Options & Flags

| Option                | Tool       | Purpose                                                 |
| --------------------- | ---------- | ------------------------------------------------------- |
| `-r`                  | DNSRecon   | Specifies the target IP range                           |
| `-n`                  | DNSRecon   | Specifies the DNS server                                |
| `-n`                  | Nmap       | Disables reverse DNS resolution                         |
| `-sV`                 | Nmap       | Enables service/version detection                       |
| `--script`            | Nmap       | Selects NSE scripts                                     |
| `-H`                  | LDAPSearch | Specifies the LDAP server URI                           |
| `-x`                  | LDAPSearch | Enables simple authentication                           |
| `-D`                  | LDAPSearch | Specifies the authentication identity                   |
| `-w`                  | LDAPSearch | Supplies the authentication password                    |
| `-b`                  | LDAPSearch | Specifies the LDAP search base                          |
| `-s base`             | LDAPSearch | Limits the search scope to the base object              |
| `-LLL`                | LDAPSearch | Produces concise LDIF output                            |
| `-u`                  | NetExec    | Specifies the username                                  |
| `-p`                  | NetExec    | Specifies the password                                  |
| `--shares`            | NetExec    | Enumerates SMB shares                                   |
| `--users`             | NetExec    | Enumerates domain users                                 |
| `--pass-pol`          | NetExec    | Requests password policy information                    |
| `-SPN`                | PowerView  | Filters users associated with SPNs                      |
| `-PreauthNotRequired` | PowerView  | Filters accounts without Kerberos pre-authentication    |
| `-Unconstrained`      | PowerView  | Finds computers configured for unconstrained delegation |
| `--CollectionMethods` | SharpHound | Controls data collection methods                        |
| `-d`                  | SharpHound | Specifies the domain                                    |
| `--ZipFileName`       | SharpHound | Specifies the output ZIP filename                       |

---

# Lab Examples

> The following examples are intended for controlled environments such as CTFs, training labs, or authorized penetration-testing engagements.

### Enumerate SPN-Associated Accounts

```powershell
Get-DomainUser -SPN |
Select-Object sAMAccountName, servicePrincipalName |
Format-Table -AutoSize
```

**Purpose:** Identifies accounts associated with registered SPNs.

---

### Assess SMB Shares

```bash
netexec smb <TARGET_IP> -u 'Guest' -p '' --shares
```

**Purpose:** Tests SMB share accessibility using the Guest account where such testing is authorized.

---

# Output Interpretation

A typical LDAP result may contain fields such as:

```text
dn: CN=<SERVICE_ACCOUNT>,OU=ServiceAccounts,DC=<DOMAIN>,DC=<TLD>
objectClass: user
sAMAccountName: <SERVICE_ACCOUNT>
userPrincipalName: <SERVICE_ACCOUNT>@<DOMAIN>
servicePrincipalName: <SERVICE>
memberOf: <GROUP>
```

### Important Fields

**`sAMAccountName`**

The account logon name.

**`userPrincipalName`**

The account's UPN, commonly represented as:

```text
user@domain.example
```

**`servicePrincipalName`**

Identifies a service principal associated with the account and is relevant when assessing Kerberos service-account configurations.

**`memberOf`**

Shows the groups associated with the account and helps determine its effective privileges.

---

# Troubleshooting

## LDAP Bind Failure

### Error

```text
ldap_bind: Invalid credentials (49)
```

### Possible Cause

Incorrect credentials or incorrect domain/UPN formatting.

### Example

```bash
ldapsearch -x \
-H ldap://<DC_IP> \
-D "user@corp.local" \
-w "<PASSWORD>" \
-b "dc=corp,dc=local"
```

---

## PowerShell Execution Policy

### Error

```text
File C:\PowerView.ps1 cannot be loaded because running scripts is disabled on this system.
```

### Cause

The current PowerShell execution policy prevents script execution.

### Lab/Authorized Testing

For a temporary process-level policy change:

```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
```

This change applies only to the current PowerShell process.

---

# Quick Command Reference

## DNSRecon

```bash
dnsrecon -r <IP_RANGE> -n <DNS_SERVER>
```

## DHCP Discovery

```bash
sudo nmap --script broadcast-dhcp-discover
```

## LDAP Service Enumeration

```bash
nmap -n -sV --script "ldap* and not brute" <DC_IP>
```

## LDAP Root DSE

```bash
ldapsearch -H ldap://<DC_IP> -s base -b "" "objectClass=*" -LLL
```

## SMB Enumeration

```bash
netexec smb <TARGET_RANGE> -u '<USER>' -p '<PASSWORD>' --shares --users
```

## SMB Password Policy

```bash
netexec smb <TARGET_IP> -u '' -p '' --pass-pol
```

## Find SPN Accounts

```powershell
Get-DomainUser -SPN
```

## Find AS-REP Candidates

```powershell
Get-DomainUser -PreauthNotRequired
```

## Find Unconstrained Delegation

```powershell
Get-DomainComputer -Unconstrained
```

## Find Domain Computers

```powershell
Get-DomainComputer -OperatingSystem "*Server*"
```

## SharpHound Collection

```powershell
.\SharpHound.exe --CollectionMethods All -d <DOMAIN> --ZipFileName loot.zip
```

## ADRecon

```powershell
.\ADRecon.ps1 -Method LDAP -DomainController <DC_IP> -Credential (Get-Credential)
```

## SYSVOL ACL Audit

```powershell
Get-Acl -Path "\\<DC_IP>\sysvol\<DOMAIN>\Policies" | Format-List
```

---

# Safety & Authorization

These commands are intended for **educational purposes, authorized penetration testing, and controlled security labs**.

Only execute reconnaissance, enumeration, credential-testing, or security-assessment commands against systems for which you have explicit authorization.

Never commit real passwords, API keys, private credentials, captured authentication material, or sensitive client information to a public GitHub repository.
