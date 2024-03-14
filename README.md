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

## Splunk

- Installed Ubuntu Server on a VM using VirtualBox
  - Configured with 4gb of memory, 1vcpu, and 100 gb of storage
- Configured username and password, and ran:

```bash
sudo apt-get update && sudo apt-get upgrade
```

## Kali Linux

- Installed Kali Linux on a VM using VirtualBox from the Kali Linux official website

## Work In Progress

- This project is still a work in progress! I will be documenting it as I go
