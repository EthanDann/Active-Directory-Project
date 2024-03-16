# Active Directory Project

The goal of this project is to gain hands-on experience with a SIEM in an Active Directory environment, generating telemetry related to attacks actually seen in the real world so that I can better detect them in the future.

## Components Used

- **Active Directory**: Microsoft's Active Directory is utilized as the directory service and identity management solution.
- **Splunk**: Splunk is used as the SIEM (Security Information and Event Management) and XDR (Extended Detection and Response) platform.

## Project Logical Diagram

![Project Logical Diagram](https://github.com/EthanDann/Active-Directory-Project/blob/main/Active_Directory_Project.drawio.png?raw=true)

## Active Directory

- Installed Windows Server 2022 on a VM using VirtualBox
  - Configured with 4gb of memory, 1 vcpu, and 50 gb of storage
- Configured to use NAT network on VirtualBox 

## Splunk Server

- Installed Ubuntu Server on a VM using VirtualBox
  - Configured with 4gb of memory, 1vcpu, and 100 gb of storage
- Configured username and password, and ran:
```bash
sudo apt-get update && sudo apt-get upgrade
```

- Configured to use NAT network on VirtualBox
- Configured to use a static IP address of '192.168.10.10/24', the DNS server as Google's (8.8.8.8), and set it to route to the gateway (192.168.10.1)
- Signed up for a Splunk account and downloaded the splunk debian file
- Installed virtualbox guest additions by running:
```bash
sudo apt-get install virtualbox-guest-additions-iso
```
- Installed virtualbox guest utils by running:
```bash
sudo apt-get install virtualbox-guest-utils
```
- Selected a shared folder where the Spunk download was placed
- Rebooted machine to finalize installations
- Added user to group 'vboxsf' by running:
```bash
sudo adduser <user> vboxsf
```
- Created a directory named 'share'
- Mounted shared folder to directory 'share' by running:
```bash
sudo mount -t vboxsf -o uid=1000,gid=1000 <shared folder name> share/
```
- Installed Splunk by changing to the 'share' directory and running:
```bash
sudo dpkg -i <splunk install file name>
```
- Switched to user 'splunk' and ran the splunk installer
- Once installed, enabled splunk to start on boot and switch to user 'splunk' by running:
```bash
sudo ./splunk enable boot-start -user splunk
```

## Windows 10 Target Machine

- Changed pc name to 'target-PC' in settings
- Changed IP address to '192.168.10.100', default gateway to '192.168.10.1', and DNS server to '8.8.8.8'
- Installed Splunk Universal Forwarder and set the receiving index to the Splunk Server (192.168.10.10) on port 9997
- Installed sysmon with the config file from 'olaf' by downloading both files and running this on PowerShell:
```bash
cd C:<symon install path>
.\Symon64.exe -i ..\sysmonconfig.xml
```
- Created a file named 'inputs.conf' and inputted the following information to ensure data was being sent to the correct places:
```bash
[WinEventLog://Application]

index = endpoint

disabled = false

[WinEventLog://Security]

index = endpoint

disabled = false

[WinEventLog://System]

index = endpoint

disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]

index = endpoint

disabled = false

renderXml = true

source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
``` 
- Changed the SplunkForwarder service to login as the system account in the Services application and restarted the service
- Went to the splunk:8000 web ui and created an index named 'endpoint' per the inputs.conf file
- Added a new receiving port of 9997 on the splunk web ui to receive data
### Troubleshooting
- Checked to see if any data was being received on the 'endpoint' indexer
  - Verified all configuration files were correct
  - Created an outbound rule on the firewall to allow connections through port 9997
  - Splunk Web was now receiving data on the 'endpoint' indexer    

## Kali Linux

- Installed Kali Linux on a VM using VirtualBox from the Kali Linux official website

## Work In Progress

- This project is still a work in progress! I will be documenting it as I go
