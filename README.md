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
 Username: "put yours“                                                                                                                                                                           
Password: "put yours"

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



# Step 03: Create Log Analytics Workspace

  logWorkspace="SentinelWorkspace"
  resourceGroup="Honeypot_RG"
  location="westus2"

  az monitor log-analytics workspace create \
    --resource-group $resourceGroup \
    --workspace-name $logWorkspace \
    --location $location




# Step 4: Enable Microsoft Sentinel on the Workspace

  logWorkspace="SentinelWorkspace"
  resourceGroup="Honeypot_RG"

  az sentinel workspace create \
    --resource-group $resourceGroup \
    --workspace-name $logWorkspace


# Step 5: Connect the VM to Log Analytics


  # Variables (Update these values accordingly)
    resourceGroup="Honeypot_RG"
    vmName="AzureSentinelHoneypotVM"
    workspaceName="log-honeypot"
    workspaceResourceGroup="Honeypot_RG" 
    location="westus2" 

  # Get Workspace ID
    workspaceId=$(az monitor log-analytics workspace show \
    --resource-group "$workspaceResourceGroup" \
    --workspace-name "$workspaceName" \
    --query customerId -o tsv)

  # Connect VM to Log Analytics workspace
    az vm extension set \
    --resource-group "$resourceGroup" \
    --vm-name "$vmName" \
    --name "OmsAgentForLinux" \
    --publisher "Microsoft.EnterpriseCloud.Monitoring" \
    --settings "{\"workspaceId\":\"$workspaceId\"}"

    echo "VM '$vmName' has been connected to Log Analytics Workspace '$workspaceName'"



# Step 06: Detection Rules in Microsoft Sentinel
  
  1. Go to Microsoft Sentinel --> Analytics -->  Create --> Scheduled rule
  2. Or Second way: Go to Workspace Log Analytics --> Logs --> KQL Query
  2. Execute Query KQL
     SecurityEvent
      | where EventID == 4625
      | where AccountType == "User"
      | summarize FailedAttempts = count() by Account, bin(TimeGenerated, 1h)
      | where FailedAttempts >= 5


# Step 7: Disable the Firewall in Virtual Machine
  1. Go to Virtual Machine "AzureSentinelHoneypotVM"
  2. Log into the VM via Remote Desktop Protocol (RDP) with credentials
  3. Select NO for all Choose privacy settings for your device
  4. Click Start and search for "wf.msc" (Windows Defender Firewall)
  5. Click "Windows Defender Firewall Properties"
  6. Turn Firewall State OFF for Domain Profile Private Profile and Public Profile

