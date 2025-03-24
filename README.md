# Azure Sentinel Honeypot for Real-Time RDP Attack Detection

# Overview
This document outlines the detailed steps to deploy a honeypot system on Azure using a Virtual Machine (VM), integrated with Azure Sentinel to detect real-time RDP (Remote Desktop Protocol) attacks. The honeypot leverages Sysmon for endpoint-level visibility, capturing detailed Windows event logs, and Log Analytics for centralizing logs. This setup helps demonstrate proactive security monitoring using Azure's cloud-native SIEM capabilities.

Objective:

1. Deploy and monitor a Windows VM acting as a honeypot.
2. Capture malicious behavior using Sysmon.
3. Detect RDP brute force attempts or unauthorized logins.
4. Forward logs to Azure Sentinel for real-time detection and alerting.

# Summary of Steps Completed
1. Created a Windows-based Virtual Machine on Azure.
2. Installed and configured Sysmon using SwiftOnSecurity's configuration file.
3. Verified Sysmon logging via Windows Event Viewer.
4. Created an Azure Log Analytics Workspace.
5. Deployed Azure Sentinel and connected it to the workspace.
6. Connected the VM to Log Analytics for centralized log forwarding.



# Step 1: Create Virtual Machine (Windows)

# Instance details
   VM Name (AzureSentinelHoneypotVM)
   Recommended region ((US) West US 2)
   Availability options: No infrastructure redundancy required
   Security type: Standard
   Image: Windows 10 Pro, version 21H2 - x62 Gen2
   VM Architecture: x64
   Size: Default is okay (Standard_D21_v2 – 1vcpus, 3.5 GiB memory)


# Login to Azure
az login

# Set variables
resourceGroup="Honeypot_RG"
location="westus2"
vmName="AzureSentinelHoneypotVM"
adminUser="mo7mzaa"
adminPassword="Bcsf12@0271122"

# Create Resource Group
az group create --name $resourceGroup --location $location

# Create VM
az vm create \
  --resource-group $resourceGroup \
  --name $vmName \
  --image Win2019Datacenter \
  --admin-username $adminUser \
  --admin-password $adminPassword \
  --size Standard_DS1_v2 \
  --output json

# Step 2: Install Sysmon on VM

1. Download Sysmon:
   https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon

   Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig.xml" -OutFile "sysmonconfig.xml"

2. Install Sysmon:
   sysmon.exe -accepteula -i sysmonconfig.xml

3. Verify:
   Open Event Viewer
   Navigate to Applications and Services Logs > Microsoft > Windows > Sysmon > Operational








