# Active Directory Home Lab in Microsoft Azure

## Overview

This project documents the deployment and administration of a small Microsoft Active Directory environment in Azure using Windows Server 2022. I built a domain controller, created a branch-based OU structure, provisioned users and security groups, joined a second server to the domain as a client, verified DNS/Kerberos authentication, configured domain password policy, deployed a logon script through SYSVOL/NETLOGON, and delegated password-reset permissions to a Help Desk group.

The project also includes real troubleshooting scenarios involving Azure virtual networks, DNS, Windows file extensions, and Remote Desktop/NLA behavior.

> **Security note:** Passwords, public IP addresses, and other sensitive account details are intentionally excluded from this repository.

## Lab Environment

| Component | Configuration |
|---|---|
| Cloud platform | Microsoft Azure |
| Domain Controller | `DC01` |
| Client VM | `CLIENT01` |
| Server OS | Windows Server 2022 Datacenter |
| AD DNS domain | `jerron.lab` |
| NetBIOS domain | `JERRON` |
| DC01 private IP | `172.16.0.4` |
| CLIENT01 private IP | `172.16.0.5` |
| Azure VNet | `vnet-eastus2-1` |
| Subnet | `snet-eastus2-1` |
| Resource group | `ad-lab-rg` |

## Architecture

```mermaid
flowchart LR
    A[Azure VNet\nvnet-eastus2-1] --> B[DC01\n172.16.0.4\nAD DS + DNS + Kerberos]
    A --> C[CLIENT01\n172.16.0.5\nDomain Joined]
    C -->|DNS| B
    C -->|Authentication / Kerberos| B
    B --> D[jerron.lab]
```

## Active Directory Structure

```text
jerron.lab
├── OU_Branches
│   └── Houston
│       ├── Users
│       ├── Workstations
│       └── Laptops
└── OU_Groups
    ├── Accounting
    ├── Help Desk
    └── IT Support
```

---

# Step 1 — Create the Windows Server VM in Azure

I created a Windows Server 2022 virtual machine named `DC01` in Azure. The VM was deployed in the `ad-lab-rg` resource group and placed on the virtual network that would later also be used by CLIENT01.

For cost control, I stopped/deallocated lab VMs when not in use. Azure pricing varies by region, VM size, storage, and subscription, so the environment should be shut down when the lab is paused and deleted when no longer needed.

## Static Private IP

Because the domain controller also provides DNS, it needs a stable private IP. I changed the NIC's private IP assignment from dynamic to static.

![DC01 static private IP](images/01-dc01-static-private-ip.png)

**Checkpoint:** DC01 deployed successfully, RDP connectivity worked, Windows Server loaded, and the private IP was configured as static.

---

# Step 2 — Install Active Directory Domain Services

On DC01, I used Server Manager to install the **Active Directory Domain Services (AD DS)** role and the required management tools.

![AD DS role selected](images/02-ad-ds-role-selected.png)

The role installation completed successfully.

![AD DS installation success](images/03-ad-ds-install-success.png)

At this point, the AD DS binaries were installed, but the server had not yet been promoted to a domain controller.

---

# Step 3 — Promote DC01 to a Domain Controller

I promoted DC01 and created a new Active Directory forest with the root domain:

```text
jerron.lab
```

![New forest configuration](images/04-new-forest-configuration.png)

The prerequisite check completed successfully before promotion.

![AD DS prerequisite check](images/05-prerequisites-check.png)

After the server rebooted, I verified that I was authenticated to the new domain.

![Domain administrator login verification](images/06-domain-admin-login.png)

**Result:** DC01 became the first domain controller for `jerron.lab`, with DNS and Global Catalog functionality available.

---

# Step 4 — Create Organizational Units

I created a branch-based OU design instead of placing users and workstations directly in default containers.

The top-level branch OU was created with accidental-deletion protection enabled.

![Create OU_Branches](images/07-create-branches-ou.png)

The completed Houston branch structure contains separate OUs for users, workstations, and laptops.

![Completed OU structure](images/08-completed-ou-structure.png)

This structure supports targeted Group Policy, easier troubleshooting, and cleaner delegation.

---

# Step 5 — Create Domain Users

I created users inside `OU_Branches → Houston → Users` instead of the default Users container.

![Create Active Directory user](images/09-create-ad-user.png)

The lab users were:

- Dak Prescott — `dakprescott`
- CeeDee Lamb — `cdlamb`
- George Pickens — `Gpickens`

![Houston Users OU](images/10-houston-users-ou.png)

---

# Step 6 — Create Security Groups

I created a centralized `OU_Groups` OU containing role-based global security groups:

- Accounting
- Help Desk
- IT Support

![Security groups OU](images/11-security-groups-ou.png)

I added the lab users to the **Help Desk** security group to demonstrate access through group membership rather than direct user permissions.

![Help Desk group membership](images/12-helpdesk-group-membership.png)

---

# Step 7 — Create CLIENT01 and Join the Domain

I created a second Windows Server 2022 VM named `CLIENT01` and used it as a domain-joined client.

## DNS Configuration and Verification

CLIENT01 was configured to use DC01 (`172.16.0.4`) as its DNS server. Active Directory depends on DNS to locate domain controllers and domain services.

From CLIENT01, `nslookup jerron.lab` successfully resolved the domain through DC01.

![CLIENT01 DNS verification](images/13-client01-dns-verification.png)

## Domain Join

CLIENT01 was joined to `jerron.lab`.

![Successful domain join](images/14-successful-domain-join.png)

After reboot, I verified the computer's domain membership with PowerShell.

![CLIENT01 domain membership verification](images/15-client01-domain-membership-verification.png)

Active Directory automatically created the CLIENT01 computer account in the default Computers container.

![CLIENT01 computer object](images/16-client01-ad-computer-object.png)

I then moved CLIENT01 into the Houston Workstations OU.

![CLIENT01 in Workstations OU](images/17-client01-workstations-ou.png)

---

# Step 8 — Test Domain Authentication

Because CLIENT01 is Windows Server and the lab is accessed through RDP, standard domain users needed permission to sign in through Remote Desktop.

## Verify User Group Membership

Dak Prescott was confirmed as a member of the Help Desk security group.

![Dak Prescott group membership](images/18-dakprescott-group-membership.png)

## Grant RDP Through Group Membership

Instead of granting RDP permission directly to Dak, I added the **JERRON\Help Desk** domain group to CLIENT01's local **Remote Desktop Users** group.

![Remote Desktop Users group](images/19-remote-desktop-users-group.png)

## Verify Domain Login

I signed into CLIENT01 as Dak and confirmed the session was using his domain identity.

![Domain user login verification](images/20-domain-user-login-verification.png)

## Verify Security Token

`whoami /groups` confirmed that the `JERRON\Help Desk` group was present in Dak's Windows security token.

![Domain user security token](images/21-domain-user-security-token.png)

## Verify Kerberos

`klist` showed Kerberos tickets for the `JERRON.LAB` realm and DC01 as the KDC.

![Kerberos ticket verification](images/22-kerberos-ticket-verification.png)

This validated domain authentication, group membership evaluation, and Kerberos operation end-to-end.

---

# Step 9 — Bonus Administration Tasks

## 9.1 Configure Domain Password Policy

Using Group Policy Management, I configured the domain password policy with settings including:

- Minimum password length: 10 characters
- Maximum password age: 90 days
- Password complexity: Enabled
- Password history: 24 passwords remembered

![Domain password policy](images/23-domain-password-policy.png)

I forced a policy refresh and verified the effective domain policy with PowerShell.

![Password policy verification](images/24-password-policy-verification.png)

## 9.2 Create and Assign a Logon Script

I created `logon.bat` in the domain's SYSVOL/NETLOGON scripts location and assigned it to Dak Prescott through the user's Profile settings.

![User logon script assignment](images/25-user-logon-script-assignment.png)

The script created `Domain-Login.txt` on Dak's desktop and wrote the logged-in domain username to the file.

![Logon script verification](images/26-logon-script-verification.png)

## 9.3 Delegate Password Reset Permissions to Help Desk

I used the Delegation of Control Wizard on the Houston Users OU to grant the `JERRON\Help Desk` group permission to:

- Reset user passwords
- Force password changes at next logon

![Help Desk password reset delegation](images/27-help-desk-password-reset-delegation.png)

## 9.4 Test Delegated Password Reset

While logged in as `JERRON\dakprescott`, I reset CeeDee Lamb's password using the delegated Help Desk permissions.

![Delegated password reset](images/28-help-desk-delegated-password-reset.png)

The first reset required a password change at next logon. RDP/NLA blocked the connection because the password had to be changed before the remote session could be established.

![Forced password change behavior](images/29-user-forced-password-change.png)

For final verification, the test account was reset again without the forced-change flag and successfully authenticated to CLIENT01.

![Password reset verification](images/30-password-reset-verification.png)

---

# Troubleshooting and Lessons Learned

## CLIENT01 Was Created on the Wrong Azure VNet

The original CLIENT01 was accidentally created on `vnet-eastus2-2`, while DC01 was on `vnet-eastus2-1`. Both VNets used the same `172.16.0.0/24` address space, so both machines received `172.16.0.4` even though they were isolated from each other.

### Symptoms

- CLIENT01 could not resolve `jerron.lab`
- DNS port 53 testing failed
- CLIENT01 appeared to be contacting `172.16.0.4`, but that address was actually its own NIC on the separate VNet

### Resolution

I recreated CLIENT01 on:

```text
vnet-eastus2-1 / snet-eastus2-1
```

Azure then assigned CLIENT01 `172.16.0.5`. I configured its preferred DNS server as DC01 (`172.16.0.4`), verified DNS resolution, and successfully joined the domain.

## Security Group Name Mismatch

An RDP configuration command initially referenced `JERRON\Helpdesk`, but the real group name was `JERRON\Help Desk` with a space. After verifying the exact AD group name, the group was successfully added to CLIENT01's local Remote Desktop Users group.

## Hidden `.txt` Extension on the Logon Script

The logon script initially appeared to be named `logon.bat`, but Windows had actually saved it as:

```text
logon.bat.txt
```

The file was visible in NETLOGON but could not execute as a batch file. I renamed it to `logon.bat`, verified it through `\\jerron.lab\NETLOGON`, and successfully executed the script.

## RDP / NLA and Forced Password Changes

After a Help Desk password reset with **User must change password at next logon** enabled, the RDP client returned error `0x1207`. Network Level Authentication requires authentication before the full Windows session starts, while the account required an interactive password change first.

The behavior confirmed the forced-password-change state was being enforced. For final remote-login verification, I reset the lab user's password again without that flag.

---

# Skills Demonstrated

- Microsoft Active Directory Domain Services
- Windows Server 2022 administration
- Azure virtual machines and virtual networking
- DNS configuration and troubleshooting
- Domain controller deployment
- Organizational Unit design
- User and group administration
- Role-based access through security groups
- Domain joining
- Group Policy Management
- Windows password policy
- Kerberos authentication and ticket inspection
- Windows access tokens and `whoami`
- Remote Desktop authorization
- SYSVOL / NETLOGON
- Windows logon scripts
- Delegation of Control
- Help Desk password resets
- PowerShell administration
- Network and authentication troubleshooting

---

# Key Takeaways

This lab demonstrated the full lifecycle of a small Active Directory environment: deploying infrastructure, creating a domain, organizing directory objects, joining endpoints, authenticating users, applying policy, delegating administrative permissions, and troubleshooting real configuration problems.

The most valuable takeaway was seeing how dependent Active Directory is on correct DNS and network design. The wrong-VNet issue prevented domain communication even though the IP addressing initially looked correct, reinforcing the importance of validating network placement, name resolution, and service connectivity before troubleshooting higher-level authentication problems.

---

## Repository Notes

- All credentials have been excluded.
- Public IP addresses and sensitive Azure account information are not intentionally published.
- This environment was created for hands-on learning and should not be treated as a production AD design.
