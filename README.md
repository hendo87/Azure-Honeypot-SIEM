# Azure Honeypot + SIEM
![Project 1](https://github.com/hendo87/Azure-Honeypot-SIEM/assets/95535877/2d43475d-b02f-4025-a0be-13c62ccb0f63)

# Introduction
<br>
In this project I build a honeypot and SIEM using Microsoft Azure. I start by building 2 virtual machines , 1 Windows VM with a SQL server installed and 1 Linux VM. I leave these machines open to the public to create real traffic and alerts from bad actors trying to hack into the VM's. This creates incidents that i walkthrough how to respond to using NIST 800-53 Access Control and NIST 800-61 Incident response in Micorsoft Sentinel . In this project I also configure Active Directory , Azure Activity logs , and Azure Key Vault storage to create logs that get ingested through Log Analytic Workspace. I configure Microsoft Sentinel to read the log files from Log Analytic Workspace then use KQL scripts to format the logs. 

