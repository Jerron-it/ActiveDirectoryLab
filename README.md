<h1>Active Directory Home Lab</h1>

 ### [YouTube Demonstration](https://youtu.be/7eJexJVCqJo)

<h2>Description</h2>
This project is a hands-on Active Directory lab environment built to develop and demonstrate practical skills in Windows Server administration, Active Directory Domain Services (AD DS), networking, and enterprise identity management.
The lab simulates a small business environment, including a Windows Server domain controller and Windows client machines joined to the domain. The goal is to gain real-world experience with deploying, configuring, securing, and managing an Active Directory environment.
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Microsoft Azure</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Program walk-through:</h2>

<p align="center">
I started off by opening Microsoft Azure and selected Resource group. <br/>
<img src="https://imgur.com/em6CNhi.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I created a Resource Group and made my resource group name.  <br/>
<img src="https://imgur.com/vJv2evj.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I selected Virtual Network and created a new vnet and placed it under the current resource group. I then finished by creating a name for my Vnet.  <br/>
<img src="https://imgur.com/lnGnCpi.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
I now created a new virtual machine. For this project I have named my VM DC-1 using Windows Server 2025 and selected a standard 16 GiB size.   <br/>
<img src="https://imgur.com/6Rxizk4.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Under networking tab for the virtual machine I made sure to put my virtual network in the active directory vent that I just created. I have now finished creating my virtual machine.  <br/>
<img src="https://imgur.com/ro8LQ3K.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Now that I have created my resource group, virtual network, subnet, and my domain controller VM, I am going to make another VM and name it Client-1. I will then attach it to the same region and Virtual Network as DC-1  <br/>
<img src="https://i.imgur.com/K71yaM2.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Observe the wiped disk:  <br/>
<img src="https://i.imgur.com/AeZkvFQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
