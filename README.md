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

### Adding Active Directory

- Go to Server Manager and add Active Directory Domain Services
- Promote server to a domain controller
  - Click on the flag icon next to 'Manage'
  - Add a new forest with a root domain name '{rootDomain}.local'
  - Create a password
  - Install configuration and restart the server
    - Keep clicking next
  - Go to Server Manager and add new organizational units 'IT' and 'HR' within the domain
    - Tools > Active Directory Users and Computers
    - Right click the domain '{rootDomain}.local', and add new Organizational Units
  - Add a user into each OU
    - Right click the OU you want to add the user to 

### Installing Splunk Universal Forwarder and Sysmon on Windows 10 Server

- Install Splunk Universal Forwarder on the Splunk website and set the receiving index to the Splunk Server (192.168.10.10) on port 9997
    - When you start the launcher, it will ask for the 'Receiving indexer'; that's where you'll type in the IP address and port number
- Install sysmon with the config file from 'olaf' on Github by downloading both files and running this in PowerShell:
```bash
cd C:<symon install path>
.\Symon64.exe -i ..\sysmonconfig.xml
```
- Create a file named 'inputs.conf' in the /program files/SplunkUniversalForwarder/etc/system/local directory and input the following information to ensure data is being sent to the correct places:
    - This tells the Splunk Forwarder to forward those events to the Splunk server  
    - You'll need admin privileges, so open Notepad as administrator and paste it in, then save it to that directory (Change 'save as type' to 'all files')
    - In the Services app on the Windows machine, double click the SplunkForwarder service, and go to the 'Log On' tab
        - Change it to log on as 'Local System account', this is because it may not be able to collect logs due to permissions if logged in as the other account
    - Restart the SplunkForwarder service
    
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
- Go to the splunk:8000 web ui and create an index named 'endpoint' per the inputs.conf file
    - Settings, then 'Indexes', click 'New Index' and add 'endpoint'
- Add a new receiving port of 9997 on the splunk web ui to receive data
    - Settings, 'Forwarding and receiving', 'Configure receiving', click 'New Receiving Port' and add port 9997
- Go to 'Search and Reporting' under Apps, and check to see if events are being forwarded by typing 'index=endpoint'

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
- Change PC name to 'target-PC' in settings
- Restart PC
- Change IPv4 address to '192.168.10.100', Subnet mask to '255.255.255.0', default gateway to '192.168.10.1', and DNS server to '8.8.8.8'
- Once Windows server Active Directory is configured, change the DNS server to '192.168.10.7', the Windows server address

### Adding PC to Active Directory Domain

- Type in 'This PC' in the Windows search bar and click 'properties'
  - Scroll down to 'Advanced System Settings'
  - Go to 'Computer Name', change at the bottom, and Add the domain under 'Member of'
  - Type in the administrator credentials (in a real-world environment, a new group would be created)
  - Restart PC
- Log in as the HR user to ensure everything is working properly
  - On the initial login screen, click Other User and type in the HR user credentials

### Installing Splunk Universal Forwarder and Sysmon on Windows 10 Target Machine

- Install Splunk Universal Forwarder on the Splunk website and set the receiving index to the Splunk Server (192.168.10.10) on port 9997
    - When you start the launcher, it will ask for the 'Receiving indexer'; that's where you'll type in the IP address and port number
- Install sysmon with the config file from 'olaf' on Github by downloading both files and running this in PowerShell:
```bash
cd C:<symon install path>
.\Symon64.exe -i ..\sysmonconfig.xml
```
- Create a file named 'inputs.conf' in the /program files/SplunkUniversalForwarder/etc/system/local directory and input the following information to ensure data is being sent to the correct places:
    - This tells the Splunk Forwarder to forward those events to the Splunk server  
    - You'll need admin privileges, so open Notepad as administrator and paste it in, then save it to that directory (Change 'save as type' to 'all files')
    - In the Services app on the Windows machine, double click the SplunkForwarder service, and go to the 'Log On' tab
        - Change it to log on as 'Local System account', this is because it may not be able to collect logs due to permissions if logged in as the other account
    - Restart the SplunkForwarder service
    
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
- Go to the splunk:8000 web ui and create an index named 'endpoint' per the inputs.conf file
    - Settings, then 'Indexes', click 'New Index' and add 'endpoint'
- Add a new receiving port of 9997 on the splunk web ui to receive data
    - Settings, 'Forwarding and receiving', 'Configure receiving', click 'New Receiving Port' and add port 9997
- Go to 'Search and Reporting' under Apps, and check to see if events are being forwarded by typing 'index=endpoint'

### Troubleshooting Splunk Forwarder

- Check to see if any data was being received on the 'endpoint' indexer
  - Verify all configuration files are correct
  - Create an outbound rule on the firewall to allow connections through port 9997
  - Splunk Web should now be receiving data on the 'endpoint' indexer

### Crowbar and Atomic Red Team

- Enable RDP and add both of the users that were created in Active Directory
  - This is for a later step using 'crowbar' in Kali Linux
  - Type 'This PC' in the Windows search bar, click properties, scroll down and click 'Advanced System Settings' and login as the Admin account
  - Click 'Remote' tab, and allow remote connections, then click 'Select Users'
    - Add the two users created earlier, click 'Check names' to autofill once the username is typed out
- In order for Atomic Red Team to work, add an exclusion on Microsoft Defender Antivirus for the C:\ drive
- To install Atomic Red Team, input the following in Powershell as Administrator:
```bash
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics
```
- Then, create a local account using Atomic Red Team associated with the MITRE ATT&CK id 'T1136.001' in PowerShell
  - This creates a user called 'NewLocalUser'
  ```bash
  Invoke-AtomicTest T1136.001
  ```
- Go to the Splunk Web UI to see if an event showed up for 'NewLocalUser', and after waiting a few minutes a few events should show up
- Run another technique using Atomic Red Team with the id 'T1059.001' to execute a PowerShell command
  ```bash
  Invoke-AtomicTest T1059.001
  ``` 
- This should also show up in the Splunk Web UI
 
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
  - 'index=endpoint tsmith'
  - A number of events should show up, including 20 for event code 4625, which indicates the account failed to log on 20 times
  - All events with event code 4625 happened at the exact same time
  - Should also see the source network address as the kali linux machine that initiated the attack
