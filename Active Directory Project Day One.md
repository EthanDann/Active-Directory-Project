# ﻿Day One

- Create Logical Diagram:
  * Windows 10 Client (Target Machine)
      * Utilizes sysmon and atomic red team 
      * Forwards data to splunk server
   * Kali Linux (Attacker)
      * Attacks target machine to generate telemetry
   * Splunk Server
      * Receives data from target machine
   * Active Directory Server:
      * Sends data to Splunk
      * Utilized as the directory service and identity management solution
   *  Network Information
      * Domain: DannClan
      * Network: 192.168.10.0/24
      * Splunk Server: 192.168.10.10
      * Active Directory Server: 192.168.10.7
      * Attacker: 192.168.10.250
   * Router
      * Ensures connectivity amongst all devices within the network and provides access to the internet
      * Forwards network packets between devices within the local network and beyond
   * Internet
      * Enables connectivity to resources and services beyond the local network
### Reflection  
- Creating the logical diagram helped me gain insight into the workflow of an Active Directory environment that utilizes a SIEM such as Splunk.

## Next Steps

Tomorrow I will be installing all virtual machines for the project, including Windows 10, Kali Linux, Ubuntu Server, and Windows Server 2022
