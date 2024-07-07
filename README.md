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

## Create Linux VM
We will create the Linux VM the same way as the Windows VM except for a few minor changes

<ul>
  <li>Make sure you choose the same resource group as the Windows VM</li>
  <li>Set to the same region as the Windows VM</li>
<li> Image: Ubuntu Server 20.04</li>
</ul>

![Linux VM](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/c1a683cf-23d2-4b14-9360-60b18e39dea6)

<br>

<ul><li>Set Admin account to same username and password as the Windows VM so its easy to remember</li>
<li>Confirm Licensing</li>
<li>Select Next : Disks ></li>
<li>Leave all Disks to Defaults</li>
<li>Select Next : Networking ></li>
<li>Copy same steps as you did for the Windows VM and allow all inbound traffic</li>
</ul>

# Step 2: Create Log Analytics Workspace

<br>

<i>Log Analytics Workspace is a repository for storing logs in JSON format. It's a versatile tool that helps  manage and analyze large volumes of data</i>

<br>

<ul><li>Search for "Log analytics workspaces"</li>
<li>Select Create Log Analytics workspace</li>
<li>Put it in the same resource group as VM</li>
<li>Give it a desired name</li>
<li>Add to same Region as VM</li>
<li>Select Review + create</li>
</ul>

![log_an_wrk](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/287d2ef3-e75c-4641-bdac-1a150520d346)

# Step 3: Configure Microsoft Sentinel

<br>

<i>Microsoft Sentinel is a cloud security information and event management service (SIEM). It is designed to provide security analytics and threat intelligence. We will forward the logs from Log Analytics Workspace to Sentinel</i>

<br>

<ul><li>Search for "Microsoft Sentinel"</li>
<li>Click Create Microsoft Sentinel</li>
<li>Select Log Analytics workspace name (honeypot-log)</li>
<li>Click Add</li>
</ul>

![sentinel_log](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/f59c6b85-e21e-4806-bff0-e2f56a947d77)

<br>

### Creating a Watchlist in Sentinel
<i>In this instance the Watchlist is network blocks with corresponding latitude and longitude points , we are going to use this to derive geolocation from attackers so we can plot them on a map to see where the attacks are coming from</i>

<br>

<ul><li>First download Geo Data file<a href="https://github.com/joshmadakor1/Cyber-Course-v2/blob/main/Sentinel-Maps(JSON)/geoip-summarized.csv"> Here</a></li>
  
<li>Then in Azure search for "Microsoft Sentinel"</li>
<li>Select the instance you've created</li>
<li>Configurations > Watchlist </li>
<li>Create New</li>
<li>Name/Alias : geoip</li>
<li>Source type: local file</li>
<li>File Type : CSV</li>
<li>Number of lines before row: 0</li>
<li>Upload the Geo data file you've downloaded</li>
<li>Search Key: Network</li>
<li>Review and Create > Create </li></ul>
<br>
<i>This could take awhile there are around 27000 rows </i>
<br>

# Step 3: Configure Microsoft Defender for Cloud



<ul><li>Search for "Microsoft Defender for Cloud"</li>
<li>Scroll down to "Environment settings" > subscription name > </li>
<li>Where you see your Log Analytics Workspace name , click the 3 dots to the right to edit settings</li>
<br>
  
![mcrsft_dfndr](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/1d89f1e7-1239-4c22-bf78-88c92b7e3141)
  
<li>Servers > Turn on > Save </li>
<li>Settings > Data collection</li>
<li>Select "All Events"</li>
<li>Save</li>
</ul>
<br>

<i>This will allow us to collect all security events from the logs</i>

<br>

![defender_plans](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/65c42701-132f-488e-ad74-821ffd72257c)

### Enable Microsoft Defender for Cloud for Subscription
