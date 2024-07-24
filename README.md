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

# Step 4: Configure Microsoft Defender for Cloud



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
<li>Scroll down to "Environment settings" > Azure Subscription</li>
<li>click the 3 dots to the right to edit settings</li>
<li>Defender Plans</li>
<li>Server > ON , Storage > ON , Key Vault > ON </li>
<li>On the Server tab , under monitoring coverage , select settings</li>

<br>

![Screenshot (58)](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/9ea2d192-e2b3-488a-992b-c9c425775421)

<br>

<li>Log Analytic Agent > Edit Configuration</li>
<li>Custom Workspace > Select your Log Analytic Workspace instance</li>
<li>Select Apply > Then Continue > Save</li>
<li>Select Continous Export > Log Analytics Workspace</li>
<li>Check boxes for : Security Recommendations , Secure Score , Security Alerts , Regulatory Compliance , Security attack paths</li>
<li>Scroll down to Export Configuration > Choose your resource group </li>
<li>Export target > Target Workspace > Select your Log Analytic Workspace instance</li>
<li>Save</li>

<br>

# Step 5: Enable Log Collection for VMs and Network Security Groups
<br>

## Create Azure Storage Account

<i>You can think of a Storage account as place to store files , it stores the logs from the Network Security Groups momentarily before it passes on to Log Analytics Workspace</i>

<ul>
<li>Search "Storage Account" </li>
<li>Create New</li>
<li>Select your resource</li>
<li>Give storage Account unique name</li>
<li>Choose same region as your VM's</li>
<li>Review and Create</li></ul>

<br>

![Storage](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/3958359d-fb4a-4d49-935d-807cce118081)

<br>

## Enable NSG flow logs for Windows and Linux VM's

<ul><li>Search "Network Security Group"</li>
<li>Select your Windows VM</li> 
<li>Monitoring > NSG flow logs</li>
<li>Create Flow logs</li>
<li>Select resource and select both Windows and Linux VM then confirm selection</li>
<li>Now choose the storage account that you've created</li>
<li>Retention Days: 0</li>
<li>Go to Analytics</li>
<li>Make sure you are on Version 2</li>
<li>Select Enable traffic analytics</li> 
 <br>
  
<i>This is where Defender will determine if traffic is malicious or beniegn . We use this later to map the attacks</i>

<br>

<li>Traffic Analytics Processing Interval : 10mins</li>
<li>Log Analytic Workspace : Choose your instance</li>
<li>Review and Create</li>
</ul>

![Screenshot (51)](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/9fe1651b-e40a-4ba8-a3ed-c0427036e850)

<br>

## Configure Data Collection Rules

<i>The data collection rule works in conjuction with Microsoft Defender to choose which logs are forwarded to Log Analytic Workspace , you dont want to forward all logs because that can become expensive</i>

<ul>
<li>Search "Log Analytics Workspace"</li>
<li>Settings > Agents</li>
<li>Select Data Collection Rules</li>
<li>Create</li>
<li>Choose rule name</li>
<li>Choose same region as your Virtual MAchines</li>
<li>Platform type : All</li>
<li>Next > Resources</li>
<li>Add Resources</li>
<li>Choose the Resource group you've created , click the arrow to expand and select both your VM's</li>
<li>Next > Collect and Deliver</li>
<li>Add Data Source</li>
<li>Select Linux Syslog</li>
<li>The only log we want is the LOG_AUTH , which is the Authentication logs
Leave it set to LOG_Debug , and turn the rest to none</li>

<br>

![Screenshot (55)](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/36033ca6-c408-4c42-a434-0e5699639200)

<li>Next > Destination</li>
<li>Add Data Source</li>
<li>Windows Event Log</li>
<li>For Application select information</li>
<li>For Security select Audit Success and Failure</li>
<liNext > Destination></li>
<li>Create the Data collection rule</li>
</ul>

## Configure Special Windows Event Data Collection rule
 <i>We are creating a Data source that will log anytime our firewall is modified or if the system detects malware on the machine</i>

 <ul>
<li>Search "Log Analytic Workspace</li>
<li>Settings > Agents</li>
<li>Select Data Collection Rule</li>
<li>Choose the collection rule that you have created</li>

![Screenshot (56)](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/caa6e9d0-0902-4f5e-9a85-840bdb216613)

<br>

<li>Data Sources > Windows Event Log</li>
<li>Change from Basic to Custom</li>       
<li>Download Xpath queries<a href="https://github.com/joshmadakor1/Cyber-Course-v2/blob/main/Special-Windows-Event-Data-Collection-Rules/Rules.txt"</a> Here</li>

<br>

 <i>We need to put a Xpath query to filter event logs. These queries are to trigger alerts when the firewall has been modified or when malware is detected on the system</i>

 <br>
 
 ![Screenshot (57)](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/e126b419-7b64-42ad-aed8-f3fe15e96091)

 <br>

 <li>Copy both queries and add them to the filter list</li>
<li>Save</li>

 
 </ul>

<br>

## Installing Windows Defender Agent on virtual Machine

<ul>
<li>Search Log Analytics Workspace</li>
<li>Select your instance</li>
<li>Agents > </li>
<li>Expand Log Analytics Agent Instruction</li>
<li>Copy the workspace ID and Primary Key into a notepad , you will need it later</li>
<li>For Download Windows Agent (64) > right click > Copy link</li>

<br>

![Screenshot (59)](https://github.com/user-attachments/assets/42ed0782-642b-4707-8f5d-3ef45e4e1718)

<br>
  
<li>Open your windows virtual machine</li>
<li>Paste the link from Download Windows Agent to in Microsoft edge</li>
<li>Download the file</li>
<li>Select connect agent to log analytics</li>
<li>Paste the Workspace ID and Primary Key</li>
<li>Azure cloud : Azure commercial</li>
<li>Next > Finsih</li> 
</ul>



## Configuring Azure Activity Logs
<i>The Activity logs are the Subsciription level logging system that logs any changes made to subscriptions such as modifying any of the Azure Resources you've created. We are going to forward all these logs to the Log Analytic Workspace</i>
<ul>
<li>Search Monitor > Activity log</li>
<li>Export Activity logs > Add diagnostic Settings</li>
<li>Diagnostic setting name : choose unique name</li>
<li>Select all categories</li>
<li>Destination details select : Send to log analytics Workspace</li>
<li>Save</li>
</ul>

![Screenshot (61)](https://github.com/user-attachments/assets/f375ef0b-8109-4198-9185-19c83e5e93fa)

<br>

## Configure Blob Storage and Key Vault to forward logs to Log Analytic Workspace
<i>Blob storage allows you to store unstructured data in the cloud as objects, or "blobs". These blobs can be any type of text or binary data, such as documents, media files, backups, logs, and more.</i>
<br>

<ul>
<li>Search Storage</li>
<li>Select your instance</li>
<li>Monitoring > Diagnostic Settings</li>
<li>Select Blob</li>
<li>Add Diagnostic Settings</li>
<li>Select Audit</li>
<li>Destination Details : Send to log analytics workspace</li>
<li>Save</li>

<br>

<i>Azure Key Vault is like a super secure digital safe where you can store and manage sensitive information like passwords, encryption keys, and certificates. It's designed to keep this information safe from unauthorized access and to help you control who can use it.</i>

<br>

<li>Search Key vault</li>
<li>Select Create Key vault</li>
<li>Select your resource group</li>
<li>Key vault name : Create unique name</li> 
<li>Select same region as your resources</li>
<li>Next > </li>
<li>Permission model > Vault Access Policy</li>
<li>Review and create</li>
<li>Search Key Vault again</li>
<li>Select your Key vault instance you just created</li>
<li>Monitoring > Diagnostic Settings</li>
<li>Add diagnostic Settings</li>
<li>Select Audit</li>
<li>Destination Details : Send to Log analytics Workspace</li>

</ul>





# Step 6. Building Attack Maps

<i>We are building a Attack map for the Windows VM - RDP Authentication failures , 
Linux - SSH Authentication Failures &
Network Sucurity Groups - Malicous inbound network flow
</i>
<br>
<ul>
  
<li>Go to JSON files<a href="https://github.com/joshmadakor1/Cyber-Course-v2/tree/main/Sentinel-Maps(JSON)"> Here </a> - We are only going to be dealing with , 
nsg-malicious-allowed-in.json , 
linux-ssh-auth-fail.json , 
windows-rdp-auth-fail.json</li> 


<li>Go back to Azure portal , Search Sentinel</li>
<li>Workbooks > Add Workbooks > Edit</li>
<li>Remove the existing workbook</li>
<li>Add > Add Query > Advanced Editor</li>
<li>Copy the linux-ssh-auth-fail.json files from the link and paste them into the editor</li> 

<br>

![Screenshot (62)](https://github.com/user-attachments/assets/3a1de231-8104-4d35-8f7f-4f757d58dd04)

<br>

<li>Done editing > Save > Name your map " linux-ssh-auth-fail.json"</li>
<li>Repeat steps to create maps for the windows-rdp-auth-fail.json and nsg-malicious-allowed-in.json</li>
<li>You Map shoud look like this : </li>
 
 <br>

![Screenshot (63)](https://github.com/user-attachments/assets/a3b99810-58ea-41e3-8a67-37f4962e2a59)


</ul>

## Create Alerts for Microsoft Sentinel
<ul>
<li>Go to Microsoft Sentinel > Analytics > Active Rules </li>
<li>Create Schedule Query Rule</li>
<li>NAME : TEST: Brute Force ATTEMPT - Windows</li>
<li>Description : When the same person fails to log in to the same VM more then 10 times</li>
<li>Next > Rule Logic</li>
<li>Paste in the KQL Query - 

SecurityEvent 

<br>

| where EventID == 4625 

<br>

| where TimeGenerated > ago(60m)

<br>

| summarize FailureCount = count() by AttackerIP = IpAddress, EventID, Activity, DestinationHostName = Computer

<br>

| where FailureCount >= 10</li>

<br>

![Screenshot (64)](https://github.com/user-attachments/assets/b5104fa2-17e9-4b16-9847-b97b1eb3c0b4)





<li>Add new Entity</li>
<li>IP > Address > Attacker IP</li>
<li>Host > Hostname > DestinationHostname</li>
<li>Run Query: every 5 minutes</li>
<li>Look up Data from the past 5 hours</li>
<li>Next > Incident Settings > Next > Save</li>  
</ul>




