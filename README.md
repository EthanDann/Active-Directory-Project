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

### Troubleshooting Splunk Forwarder

- Checked to see if any data was being received on the 'endpoint' indexer
  - Verified all configuration files were correct
  - Created an outbound rule on the firewall to allow connections through port 9997
  - Splunk Web was now receiving data on the 'endpoint' indexer
 
### Active Directory

- Added PC to the active directory domain '{domainName}.local', but could not communicate with the Active Directory Server
  - Changed the DNS server to point to the Active Directory Server IP address
  - PC could now be added to the domain
  - Restarted PC
- Logged in as the HR user to ensure everything was working properly
 
## Windows Server (Active Directory)

- Changed pc name to 'ADDC01' in settings
- Changed IP address to '192.168.10.7', default gateway to '192.168.10.1', and DNS server to '8.8.8.8'
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
- Verified that Splunk Web was now receiving data on the 'endpoint' indexer

### Add Active Directory

- Went to Server Manager and added Active Directory Domain Services
- Promoted server to a domain controller
  - Added a new forest with a root domain name '{rootDomain}.local
  - Created a password
  - Installed configuration and restarted the server
  - Went to Server Manager and aded new organizational units 'IT' and 'HR' within the domain
  - Added a user into each OU
## Kali Linux

- Installed Kali Linux on a VM using VirtualBox from the Kali Linux official website
- Changed IP address to 192.168.10.250 based on the network diagram
- Installed a tool called "Crowbar" by typing:
```bash
sudo apt-get install -y crowbar
```
- Created a new directory called 'ad-project'
- Unzipped the 'rockyou.txt.gz' wordlist and copied it to the 'ad-project' directory
- For purposes of the demo, copied only the first 20 lines of rockyou.txt onto a new txt file called 'passwords.txt' in the ad-project directory
```bash
head -n 20 rockyou.txt > passwords.txt
```
- Also added in the actual password of the user being targeted on the Windows 10 machine, for purposes of the demo
- Then went to the Windows 10 target machine, enabled RDP and added both of the users that were created in Active Directory
- On the kali linux terminal, started a brute force attack via 'crowbar' by typing:
```bash
crowbar -b rdp -u tsmith -C passwords.txt -s 192.168.10.100/32
```
- This specifies to use rdp as the targets service, the username being tsmith, the wordlist being passwords.txt, and the target IP address as 192.168.10.100/32 (/32 to indicate a single target IP address)
  - Since the actual password was added to the txt file, the attack was successful
- To check the event in splunk, went over to the Splunk web interface and checked for events tying to 'tsmith'
  - "index='endpoint' tsmith"
  - A number of events show up, including 20 for event code 4625, which indicates the account failed to log on 20 times
  - All events with event code 4625 happened at the exact same time
  - Can also see the source network address as the kali linux machine that initiated the attack

## Work In Progress

- This project is still a work in progress! I will be documenting it as I go
