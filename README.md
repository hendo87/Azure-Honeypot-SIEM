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
