# Setting Up Active Directory on Home Lab (Virtualbox)

The following will be a basic tutorial on setting up a home lab using Virtualbox on a WindowsOS

![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/home_lab_vm_outline.png) <br />

## Overview
In this project, you’ll learn the basics of setting up a virtual home lab to gain practical experience. Note the following changes to the diagram above: Windows Server 2022 and Oracle VirtualBox instead of Windows Server 2019 and VMWare Network respectively. This project is for educational purposes; some tools may require a license for commercial use. 


## General Concepts
* Setting up Virtual Machines (VMs)
* Virtual Network Interface Cards
* Active Directory Domain Services (AD DS)
* Configuring IP addresses and NAT Networks.
* Troubleshooting network connectivity (ping, DNS settings).
* PowerShell scripting


## Tools Used
* Oracle VirtualBox: Virtual machine, type 2 hypervisor
* Windows Server 2022 ISO: Operating system (OS) with network specializations
* Windows 10 ISO: OS for host
* PowerShell: Command line


## Part 1: Virtualization
### 1.1 Install Oracle VirtualBox
Visit [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads). Download and install both the platform package and extension pack. Note: the extension pack is free to use for personal and educational users, not commercial use.
### 1.2 Install and Initialize Windows Server
Visit [Windows Server 2022](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022) and download the ISO file. In VirtualBox Manager, click “New” to create a VM, name it (Office_DC in this case, which will serve as the domain controller), select version: “Other Windows (64-bit), and finish. Navigate to the newly created VM's settings to choose 2 CPUs, 4096GB base memory (RAM) or 2048GB based on your hardware, 20GB disk space, 96MB video memory, enable network adaptor 2 as an internal network (2 NICs as shown in the diagram above), shared clipboard as well as drag’n’drop to bidirectional, and finish. 
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/VB_manager.png) <br />
Power on the VM, attach the Windows Server ISO file, and custom install for the standard evaluation desktop experience. Complete the sign-in* instructions. The host key will likely be the right CTRL. You can also navigate to the sign-in page using Keyboard under Input at the top left of the VM. After signing in, go to your files > This PC > Open guest additions > Run the amd64 version > Restart the VM <br /> <br />
*Keep track of all your sign-in information somewhere secure such as a password manager. 
### 1.3 Install Windows 10
Visit [Windows 10](https://www.microsoft.com/en-us/software-download/windows10). Download the Windows 10 installation media, run it, create an ISO file, and save it in a directory you can remember for later use.


## Part 2: Create a domain controller (DC)
### 2.1 Set up and rename networks
Navigate to Settings > Network & Internet > Ethernet > Change Adaptor Options. Here, rename the two NICs. The internal private network will be the “unidentified network.” For the internal private network, named Cobel in this case, the IP address has to be set manually. For the internet network, named Mrs. Selvig in this case, gets auto IP addressing from our router network. Many of the names used for the configurations will be in reference to a great TV series called Severance (no, I am not a sponsor for this series). Feel free to name them whatever you like.
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/NICs.png) <br />
### 2.2 Setup IP Addressing for NICs
Right-click on the private network > Properties > Double-click Internet Protocol Version 4 (TCP/IPv4) > Select Use the following IP address > Type `172.16.0.1` for the IP address and `255.255.255.0` for the subnet mask. You don’t need a default gateway in this case since the domain controller (Severance) will itself serve as the default. There are a lot of nuances to IP addressing. For now, simply use the ones mentioned here. For the DNS server, type `127.0.0.1` for the preferred DNS server > Ok
### 2.3 Rename the PC 
To help identify the PC as the domain controller, rename the PC. In this case, “Severance.” Navigate to Settings > System > About > Rename this PC > Restart the VM



## Part 3: Create a domain / Active Directory Domain Services(AD DS)
### 3.1 AD DS
Open Server Manager if it is not already open > Add roles and features > Next until Server Roles, check “Active Directory Domain Services” and add features > Next until Results, install, and finish > Restart the server manager after the installation is complete. 
### 3.2 Create a domain
At this stage, you’ve installed the software for Active Directory Domain Services (AD DS) but you haven’t created a domain yet. Click on the yellow flag on the top right and select “Promote this server to a domain controller.”
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/server_to_domain.png) <br />
Click “Add a new forest” > “Root domain name:” You can name this anything you want followed by “.com” and in this case, it’s “myseverance.com” > Next > create a password > Next until you reach Results, install, and restart the virtual machine if it doesn’t do so automatically.
### 3.3 Create an admin account
Navigate to start > Windows Administrative Tools > Active Directory Users and Computers. Now, create an organizational unit (OU) to put the admin account in. Right-click on the domain name (myserverance.com), new, click organizational unit, name it (“S_ADMINS” in this case). Now, right-click on S_ADMINS, new, click user, and put any name in (“Dan Erickson” in this case), set a user logon name (a-derickson this case, as in admin Dan Erickson), and click next. Set a password (uncheck “User must change the password on next logon”). Usually, this isn’t done but for testing purposes, and since you have control of this account, you can uncheck it while also checking “password never expires.” 
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/domain_admin_account.png) <br />
You still have to give this account admin permissions.  Right-click on the newly created user account (Dan Erickson) and click properties > Member Of, add, type “Domain Admins” under “Enter the object names to select (examples),” click “Check Names,” apply, finish, and restart VM


## Part 4: RAS / NAT
Configure remote access service (RAS) and network address translation (NAT) so the clients on the private network can reach the internet through the domain controller.<br /> <br />
Login as “other user” with the newly created admin account.
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/domain_admin_account_login.png) <br />
On the Server Manager, navigate to “Add roles and features” > click next until “Server Roles” and check the box that says “remote access” > Next until “Role Services” and check the boxes that say “Routing” and “DirectAccess and VPS (RAS)” and add features > Next until you reach Results, install, and finish <br /> <br />
On the Server Manager, navigate to tools, and select routing and remote access
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/routing_and_remote_access.png) <br />
Now, right-click on the domain controller, and click “Configure and Enable Routing and Remote Access,” and click next > click “Network Address Translation (NAT),” select the router network (Mrs_Selvig), next, and finish
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/RAS_NAT.png) <br />


## Part 5: Setup a DHCP server on the domain controller and add sample users
This is to give Windows clients an IP address automatically.
### 5.1 Create DHCP role
On the Server Manager, navigate to “Add roles and features” >  click next until “Server Roles,” check the box that says “DHCP Server,” and add features. Next until Results, install, and finish. <br /> <br />
On the Server Manager, navigate to tools, and select DHCP to set up the scope. Click the domain dropdown, right-click on “IPv4,” new scope > Next and name it `172.16.0.100-200` for this current project > Next > Set `172.16.0.100` for “Start IP address,” `172.16.0.200` for “End IP address,” and 24 for “Length” to match the subnet mask > Next until “Router (Default Gateway),” set IP address as `172.168.0.1` (internal NIC), and click Add > Next until the end and finish. <br /> <br />
On the DHCP manager, right-click on the domain name, click authorize, right-click again, and click refresh to activate the IP addresses. Now, right-click on “Server Options,” configure, set `172.16.0.1` for the IP address, add, apply, and finish. Finally, right-click on the domain (severance.myseverance.com), all tasks, and restart.
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/DHCP.png) <br />
### 5.2 Create a sample of 1000 users in Active Directory using a PowerShell script
Open Microsoft Edge in the VM, paste this link, download, and unzip the contents in the directory. Navigate to start, Windows PowerShell, right-click PowerShell ISE, more, run as administrator, and select yes. Click on the “Open Script” file icon within PowerShell, navigate to the downloaded file, select “1_CREATE_USERS,” and open. In the command line, enter “Set-ExecutionPolicy Unrestricted” to enable execution of all scripts, and select “yes to all.”  Navigate to the directory holding the script. In this case, it’s “cd C:\Users\a-derickson\Downloads\AD_PS-master”
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/script_image_1.png) <br /> <br /> 
Click the green play button at the top, run the script, and wait until all the users have been created. <br />
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/script_image_2.png) <br />


## Part 6: Create Windows 10 VM for Client1
### 1. Create a new VM for Client1
In VirtualBox Manager, click “New” to create another VM, name it (Client1) in this case), select version: “Windows 10 (64-bit), and finish. Navigate to the newly created VM settings, choose 2 CPUs, 4096GB base memory (RAM) or 2048GB based on your hardware, 20GB disk space, 96MB video memory, and change network adaptor 1 to “Internal Network.” Change shared clipboard as well as drag’n’drop to bidirectional, and finish. 
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/VB_manager_2.png) <br />
Open the VM, attach the Windows 10 ISO file from earlier, continue, and follow the instructions. Select “I don’t have a product key,” Window 10 Pro, and custom install. Follow the instructions, select “I don’t have internet,” and continue with limited setup. Enter a name (u-hriggs in this case), skip the password, uncheck all privacy settings, and accept. After signing in, go to your files > This PC > Open guest additions > Run the amd64 version > Restart the VM
### 2. Confirm DHCP for client
With both VMs running, open the command prompt on the client VM. Enter “ipconfig” and check if the default gateway is set. If it isn’t, enter “ipconfig /renew” and the default gateway should now be shown. You can use the ping function to confirm that your router is working.
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/ipconfig_ping.png) <br />
Navigate to Settings > System > About > scroll down and select “Rename this PC (advanced)”  > Change > enter a computer name (“u-hriggs” in this case), enter domain (“myseverance.com” in this case), click OK, enter your domain admin account credentials (“a-derickson” in this case) to give permission, and restart. Additionally, on this client VM, you can log in with any of the 1000 users created earlier using the password from the script.
![alt text](https://github.com/bqazi/home_lab_vb/blob/main/images/rename_with_permissions.png) <br />
You can view the IP address lease, users, and more from the domain controller VM’s Server Manager. There are many more configurations and setups, but we have now completed setting up a simple home lab running Active Directory resembling a corporate environment. 


## Licensing
Feel free to use anything for personal use, including images.
Note that the Windows and Oracle software, however, require a license for commercial use.












