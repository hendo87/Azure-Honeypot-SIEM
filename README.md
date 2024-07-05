# Azure Honeypot + SIEM
![Project 1](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/2d43475d-b02f-4025-a0be-13c62ccb0f63)

# Introduction
<br>
In this project I build a honeypot and SIEM using Microsoft Azure. I start by building 2 virtual machines , 1 Windows VM with a SQL server installed and 1 Linux VM. I leave these machines open to the public to create real traffic and alerts from bad actors trying to hack into the VM's. This creates incidents that i walkthrough how to respond to using NIST 800-53 Access Control and NIST 800-61 Incident response in Micorsoft Sentinel . In this project I also configure Active Directory , Azure Activity logs , and Azure Key Vault storage to create logs that get ingested through Log Analytic Workspace. I configure Microsoft Sentinel to read the log files from Log Analytic Workspace then use KQL scripts to format the logs. At the end I take the metrics from the unsecured environment and compare them to the metrics of the hardend environment after the NIST controls are applied.

# Azure Resources Deployed, Technologies, and Regulations used
<br>
<ul>
  
<li><a href="https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview">Azure Virtual Network</a> (VNET)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview">Network Security Group</a> (NSG)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/virtual-machines/overview">Virtual Machines</a> (1x Windows 10 Pro, 1x Linux Server)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview">Log Analytic Workspace</a> (with Kusto Query Language KQL Queries)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/key-vault/general/basic-concepts">Azure Key Vault</a> (For Secure Secrets Management)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview">Azure Storage Account</a> (For Data Storage)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/sentinel/overview?tabs=azure-portal">Microsoft Sentinel</a> (For Security Information and Event Management (SIEM)</li>
<li><a href="https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction">Microsoft Defender</a> (for Cloud to Protect Cloud Resources)</li>
<li><a href="https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final">NIST SP 800-53 </a> (Security Controls)</li>
<li><a href="https://www.nist.gov/privacy-framework/nist-sp-800-61">NIST SP 800-61</a> (Incident Handling)</li>
</ul>

# Step 1. Create Windows and Linux Virtual Machines
![create_azure_vm](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/f0f9b5bc-aa29-4b7b-a8f1-ee43a3d73a66)

## Basics
<ul><li>Go to portal.azure.com</li>
<li>Search for "virtual machines"</li>
<li>Create > Azure virtual machine</li>
</ul>


### Project details
<ul><li>Create new resource group and give it a unique name </li>
<br>
<i>A resource group is a collection of resources that share the same lifecycle, permissions, and policies.</i>
</ul>

### Instance details
<ul><li>Name your VM </li>
<li>Select a recommended region ((US) East US 2)</li>
<li>Availability options: No infrastructure redundancy required</li>
<li>Security type: Standard</li>
<li>Image: Windows 10 Pro, version 21H2 - x62 Gen2</li>
<li>VM Architecture: x64</li>
<li>Size: Default is okay (Standard_D2s_v3 – 2vcpus, 8 GiB memory)</li>
</ul>

### Administrator account
<ul><li>Create a username and password for virtual machine</li>
<br>
  <i>IMPORTANT NOTE: These credentials will be used to log into the virtual machine (Keep them handy)</i>
</ul>

### Inbound port rules
<li>Public inbound ports: Allow RDP (3389)</li>

### Licensing
<li>Confirm licensing</li>
<li>Select Next : Disks ></li>

![vm1](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/fe7bfe28-f96e-47cb-8596-a03231b01eb5)

### Disks
<ul><li>Leave all defaults</li>
<li>Select Next : Networking ></li>
</ul>

## Networking
### Network interface
<ul>
<li>NIC network security group: Advanced > Create new</li>
  <br>
<i>A network security group contains security rules that allow or deny inbound network traffic to, or outbound network traffic from, the virtual machine. In other words, security rules management.</i>
</ul>
  <br>
<ul>  
<li>Remove Inbound rules (1000: default-allow-rdp) by clicking three dots</li>
<li>Add an inbound rule</li>
<li>Destination port ranges: * (wildcard for anything)</li>
<li>Protocol: Any</li>
<li>Action: Allow</li>
<li>Priority: 100 (low)</li>
<li>Name: Anything (ALLOW_ALL_INBOUND)</li>
<li>Select Review + create</li>
</ul>

![network_sec_grp](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/f4de8fdc-17d3-4ee9-ae68-7a3db56f6b13)
<br>

<i>Normally you would never allow all inbound traffic but for the purpose of this honeypot we want to make our machine easily discoverable for bad actors </i>
<br>

# Create Linux VM
We will create the Linux VM the same way as the Windows VM except for a few minor changes

<ul>
  <li>Make sure you choose the same resource group as the Windows VM</li>
  <li>Set to the same region as the Windows VM</li>
<li> Image: Ubuntu Server 20.04</li>
</ul>

![Linux VM](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/c1a683cf-23d2-4b14-9360-60b18e39dea6)

