# Active Directory Project

The goal of this project is to gain hands-on experience with a SIEM in an Active Directory environment, generating telemetry related to attacks actually seen in the real world so that I can better detect them in the future.

## Components Used

- **Active Directory**: Microsoft's Active Directory is utilized as the directory service and identity management solution.
- **Splunk**: Splunk is used as the SIEM (Security Information and Event Management) and XDR (Extended Detection and Response) platform.

## Project Logical Diagram

![Project Logical Diagram](Active%20Directory%20Project%20Logical%20Diagram.jpg)

## Windows 10 Server (Active Directory)

- Install Windows Server 2022 on a VM using VirtualBox
  - Configure with 4gb of memory, 1 vcpu, and 50 gb of storage
- Configure to use NAT network on VirtualBox
- Change PC name to 'ADDC01' in settings
- Change IP address to '192.168.10.7', default gateway to '192.168.10.1', and DNS server to '8.8.8.8'

### Installing Splunk Universal Forwarder and Sysmon on Windows 10 Server

- Install Splunk Universal Forwarder and set the receiving index to the Splunk Server (192.168.10.10) on port 9997
- Install sysmon with the config file from 'olaf' on Github by downloading both files and running this on PowerShell:
```bash
cd C:<symon install path>
.\Symon64.exe -i ..\sysmonconfig.xml
```
- Create a file named 'inputs.conf' and input the following information to ensure data is being sent to the correct places:
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
- Change the SplunkForwarder service to login as the system account in the Services application and restarted the service
- Go to the splunk:8000 web ui and create an index named 'endpoint', per the inputs.conf file
- Add a new receiving port of 9997 on the splunk web ui to receive data
- Verify that Splunk Web is now receiving data on the 'endpoint' indexer

### Adding Active Directory

- Go to Server Manager and add Active Directory Domain Services
- Promote server to a domain controller
  - Adde a new forest with a root domain name '{rootDomain}.local
  - Create a password
  - Install configuration and restarted the server
  - Go to Server Manager and add new organizational units 'IT' and 'HR' within the domain
  - Add a user into each OU 

## Splunk Server

- Install Ubuntu Server on a VM using VirtualBox
  - Configure with 4gb of memory, 1vcpu, and 100 gb of storage
- Configure username and password, and run:
```bash
sudo apt-get update && sudo apt-get upgrade
```

- Configure to use NAT network on VirtualBox
- Configure to use a static IP address of '192.168.10.10/24', the DNS server as Google's (8.8.8.8), and set it to route to the gateway (192.168.10.1)
  - Go to `/etc/netplan/00-installer-config.yaml` file and edit it with the following:
    ```bash
    dhcp4: no
    addresses: [192.168.10.10/24]
    nameservers:
        addresses: [8.8.8.8]
    routes:
        - to: default
          via: 192.168.10.1
    ```
- Ping 8.8.8.8 to test connectivity
- Sign up for a Splunk account and download the splunk debian file
- Install virtualbox guest additions by running:
```bash
sudo apt-get install virtualbox-guest-additions-iso
```
- Install virtualbox guest utils by running:
```bash
sudo apt-get install virtualbox-guest-utils
```
- Select a shared folder where the Spunk download was placed
  - Click 'devices' at the top left of the VirtualBox ui
  - Click 'shared folders', then 'shared folder settings'
  - Set the folder path to the Project folder path
  - Set it to read-only, auto mount, and make permanent
- Reboot machine to finalize installations
  ```bash
  sudo reboot
  ``` 
- Add user to group 'vboxsf' by running:
```bash
sudo adduser <user> vboxsf
```
- Create a directory named 'share'
- Mount shared folder to directory 'share' by running:
```bash
sudo mount -t vboxsf -o uid=1000,gid=1000 <shared folder name> share/
```
  - May need to log out and log back in, if having trouble
- Install Splunk by changing to the 'share' directory and running:
```bash
sudo dpkg -i <splunk install file name>
```
- Switch to user 'splunk' and run the splunk installer
  ```bash
  sudo -u splunk bash
  cd bin
  ./splunk start
  ``` 
- Once installed, switch back to other user and enable splunk to start on boot and to switch to user 'splunk' by running:
```bash
sudo ./splunk enable boot-start -user splunk
```

## Windows 10 Target Machine

- Install Windows 10 Pro on a VM using VirtualBox
- Change pc name to 'target-PC' in settings
- Change IP address to '192.168.10.100', default gateway to '192.168.10.1', and DNS server to '8.8.8.8'

### Installing Splunk Universal Forwarder and Sysmon on Windows 10 Target Machine

- Install Splunk Universal Forwarder and set the receiving index to the Splunk Server (192.168.10.10) on port 9997
- Install sysmon with the config file from 'olaf' on Github by downloading both files and running this on PowerShell:
```bash
cd C:<symon install path>
.\Symon64.exe -i ..\sysmonconfig.xml
```
- Create a file named 'inputs.conf' and inputt the following information to ensure data was being sent to the correct places:
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
- Change the SplunkForwarder service to login as the system account in the Services application and restarted the service
- Go to the splunk:8000 web ui and created an index named 'endpoint' per the inputs.conf file
- Added a new receiving port of 9997 on the splunk web ui to receive data

### Troubleshooting Splunk Forwarder

- Check to see if any data was being received on the 'endpoint' indexer
  - Verify all configuration files were correct
  - Create an outbound rule on the firewall to allow connections through port 9997
  - Splunk Web should now be receiving data on the 'endpoint' indexer

### Crowbar and Atomic Red Team

- Enable RDP and add both of the users that were created in Active Directory
  - This is for a later step using 'crowbar' in Kali Linux
- In order for Atomic Red Team to work, add an exclusion on Microsoft Defender Antivirus for the C:\ drive
- To install Atomic Red Team, input the following in Powershell as Administrator:
```bash
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics
```
- Then, create a local account using Atomic Red Team associated with the MITRE ATT&CK id 'T1136.001'
  - This create a user called 'NewLocalUser'
- Go to the Splunk Web UI to see if an event showed up for 'NewLocalUser', and after waiting a few minutes a few events should show up
- Run another technique using Atomic Red Team with the id 'T1059.001' to execute a PowerShell command
- This should also show up in the Splunk Web UI
 
### Adding PC to Active Directory Domain

- Add PC to the active directory domain '{domainName}.local', but it can't communicate with the Active Directory Server yet
  - Change the DNS server to point to the Active Directory Server IP address
  - PC can now be added to the domain
  - Restart PC
- Log in as the HR user to ensure everything is working properly
 
## Kali Linux

- Install Kali Linux on a VM using VirtualBox from the Kali Linux official website
- Change IP address to 192.168.10.250 based on the network diagram

### Installing and utilizing Crowbar

- Install a tool called "Crowbar" by typing:
```bash
sudo apt-get install -y crowbar
```
- Create a new directory called 'ad-project'
- Unzip the 'rockyou.txt.gz' wordlist and copy it to the 'ad-project' directory
- For purposes of the demo, copy only the first 20 lines of rockyou.txt into a new txt file called 'passwords.txt' in the ad-project directory
```bash
head -n 20 rockyou.txt > passwords.txt
```
- Add in the actual password of the user being targeted on the Windows 10 machine, for purposes of the demo
- On the kali linux terminal, start a brute force attack via 'crowbar' by typing:
```bash
crowbar -b rdp -u tsmith -C passwords.txt -s 192.168.10.100/32
```
- This specifies to use rdp as the targets service, the username being tsmith, the wordlist being passwords.txt, and the target IP address as 192.168.10.100/32 (/32 to indicate a single target IP address)
  - Since the actual password was added to the txt file, the attack is successful
- To check the event in splunk, go over to the Splunk web interface and check for events tying to 'tsmith'
  - `"index='endpoint' tsmith"`
  - A number of events should show up, including 20 for event code 4625, which indicates the account failed to log on 20 times
  - All events with event code 4625 happened at the exact same time
  - Should also see the source network address as the kali linux machine that initiated the attack
