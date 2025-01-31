# Building a SOC/Honeynet in Azure (Live Traffic)

## Introduction 

This project’s aim was to build a mini Honeynet in Azure and ingest the logs from various sources into a Log Analytics Workspace, which is then used by Microsoft Sentinel to trigger alerts, create incidents and build Attack Maps. In this project, using KQL queries we will measure some security metrics for a 24-hour period in the insecure environment, we will implement security controls to harden the environment, measure the metrics for another 24-hour and show the results. The metrics we will show are: 

- Syslog (Linux Event Logs)  
- SecurityEvent (Windows Event Logs)   
- SecurityAlert (Log Analytics Alert Triggered)   
- SecurityIncident (Incidents Created by Sentinel)   
- AzureNetworkAnalytics_CL (Malicious flows allowed into our honeynet)   


## Infrastructure and log aggregation 

![Screenshot 2024-11-15 212138](https://github.com/user-attachments/assets/a3035102-49ea-44b3-8ce4-dfd3ea530d09)

The architecture of the Honeynet consists of the following components:

- Virtual Network (VNet)  
- Virtual Machines (1x Linux, 2x Windows)   
- SQL Database (Installed on a Windows VM)   
- Network Security Groups (NSG’s)   
- Azure Key Vault (containing stored secrets)   
- Azure Blob Storage Account (containing geoip data)   
- Log Analytics Workspace   
- Microsoft Sentinel   


After deploying the resources mentioned above, data collection rules were configured to send the collected logs to a central repository (Log Analytics Workspace), and several rules were established to trigger incidents in Microsoft Sentinel. A virtual machine (VM) was utilized to simulate an attacker, testing the environment. This approach verified that logs were correctly aggregated into the Log Analytics Workspace and ensured that the analytic rules in Microsoft Sentinel triggered incidents as expected. 


## Measuring the metrics in the INSECURE environment 


![Screenshot 2024-11-15 212205](https://github.com/user-attachments/assets/94a68dfd-0687-46a2-a655-886f5f638e6b)

 
All resources were initially deployed with public exposure to the internet. This setup was intentionally built to attract malicious actors, generate logs and observe their tactics. The Virtual Machines had both their Network Security Groups (NSGs) and built-in firewalls wide open to the internet, allowing unrestricted access from any source. All other resources, such as storage accounts and databases, were deployed with public endpoints visible to the internet. 

During the 24-hour window our Attack VM was used to generate brute-force attacks on an Azure Account logging in with incorrect credentials multiple times and eventually logging in with the correct ones, then the same account was used to access Azure Key Vault secrets. An EICAR test file was also deployed on the Windows-VM to simulate a Malware Alert. Please note that except for those incidents all other incidents were created by actors trying to brute-force into our resources. 

The following table shows the metrics we measured in our INSECURE environment for 24 hours:
Start Time 11/11/2024 10:34:12 
Stop Time 11/12/2024 10:34:12 

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 26191 
| Syslog                   | 21607 
| SecurityAlert            | 10 
| SecurityIncident         | 663 
| AzureNetworkAnalytics_CL | 5186 


## ATTACK MAPS BEFORE HARDENING 

![(before)nsg-mallicious-allowed-in](https://github.com/user-attachments/assets/302bb4e9-cd1d-4349-ab2d-5d5d83095273)

![(before)linux-ssh-auth-fail](https://github.com/user-attachments/assets/9a37089d-3992-43de-92e4-4e702720fa96)

![(before)windows-rpd-auth-fail](https://github.com/user-attachments/assets/47203900-907e-4627-b032-970edc3dfd12)

![(before)mssql-auth-fail](https://github.com/user-attachments/assets/d8daae66-9880-4da1-a843-3c20872d7ce8)

## Incident handling 

 

The incident handling guidance was done in accordance with NIST SP 800-61 Revision 2 and the security controls were implemented using Microsoft Defender for Cloud in accordance with NIST SP 800-53 Revision 5.   

 

During the log investigation: 


- It was concluded that there were no actual breaches into our VM’s and SQL Database but a large number of brute-force attempts from multiple sources due to insecure NSG’s.    
- An EICAR file was deployed on the windows machine to test trigger an antivirus response.                  
- An Azure account was compromised by a brute-force attack and a stored secret was viewed in Key Vault.   

 

 

The hardening measures and security controls implemented include: 


- Network Security Groups (NSG’s): All inbound and outbound traffic was blocked with the exception of my admin workstation Public IP Address ensuring that only trusted traffic is allowed to access the virtual machines.   
- Firewalls: The built-in firewalls were configured to protect resources and restrict access from unauthorized connections minimizing the attack surface.   
- Private Endpoints: The Public Endpoints were replaced with Private Endpoints ensuring that access is limited and the sensitive resources such as databases are protected.   
- Azure Password reset: The affected Azure Account user had their password reset.   
- MFA: Multi factor authentication was implemented for all Azure accounts to protect against brute force attempts.   
- Regenerated the compromised Key Vault Secrets   

 

## Measuring the metrics in the SECURE environment 


![Screenshot 2024-11-15 212220](https://github.com/user-attachments/assets/16a0c4d2-5215-45a9-9632-49212a007d09)


The following table shows the metrics we measured in our environment for another 24 hours, but after we have applied security controls:
Start Time 11/14/2024, 7:03:02 PM 
Stop Time	11/15/2024, 7:03:02 PM 


| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 6281
| Syslog                   | 826
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0


## ATTACK MAPS AFTER HARDENING 


The Map queries have not returned any results and therefore no maps have been generated. 


## Conclusion 


This project highlights the importance of building a secure environment to mitigate cybersecurity threats. The initial insecure configuration allowed for significant malicious activity to take place. After implementing security controls such as Private Endpoints, secure Network Security Groups (NSG’s), built-in firewalls, Multi-Factor Authentication (MFA), and password resets, the malicious activity was greatly reduced. The metrics after hardening showed a complete reduction in malicious flows, Sentinel incidents, and security alerts, demonstrating the effectiveness of the implemented measures however it’s worth mentioning that in an environment with more resources and active users a larger number of security alerts and events would likely have been generated. 
