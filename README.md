# Configuring-Cisco-ASA-Basic-Settings-and-Firewall-Using-CLI

Acting as a Security Analyst, this lab was based on a real scenario in the implementation of a Cisco ASA 5506, to create a Firewall and protect a corporation’s internal network from external intruders while allowing internal hosts to have access to the internet.

When we are implementing an ASA, three security interfaces are created: Outside, Inside and DMZ. This will allow Outside users to have limited access to the DMZ and no access to Inside resources.

Step-by-Step:

Part 1: in firts part of this lab you will configure the topology and non-ASA devices.
Part 2 — Part 4: In Parts 2 through 4 you will configure basic ASA settings and the firewall between the inside and outside networks
Part 5: In Part 5 you will configure the ASA for additional services, such as AAA, and SSH connection.
Part 6: In Part 6, you will configure a DMZ on the ASA and provide access to a server in the DMZ.

Topology of the project -> 

![image](https://github.com/user-attachments/assets/fb47f280-fc3d-48c8-aded-e1a3c1d7af2f)


IP Addressing Table:

![image](https://github.com/user-attachments/assets/eb03fd3b-65dc-4601-a2b6-a03896af5745)

Part 1: Basic Router/Switch/PC Configuration → In Part 1 of this lab, you will configure basic settings on the routers, such as interface IP addresses and static routing. (Note: Do not configure ASA settings at this time).

Step 1: Configure basic settings for routers and switches →

Configure hostnames as shown in the topology for each router.
Configure router interface IP addresses as shown in the IP Addressing Table
Step 2: Configure static routing on the routers →

Configure a static default route from R1 to R2 and from R3 to R2: R1(config)# ip route 0.0.0.0 0.0.0.0 Serial0/0/0 R3(config)# ip route 0.0.0.0 0.0.0.0 Serial0/0/1

![image](https://github.com/user-attachments/assets/6fdc6605-e25c-4827-bd89-4070a344075e)

Configure a static route from R2 to the R1 G0/0 subnet (connected to ASA interface Gi1/1) and a static route from R2 to the R3 LAN: R2(config)# ip route 209.165.200.224 255.255.255.248 Serial0/0/0 R2(config)# ip route 172.16.3.0 255.255.255.0 Serial0/0/1

![image](https://github.com/user-attachments/assets/f91158b6-4a70-4a0a-bca3-d07128ed5320)

Step 3: configure a user account, encrypted passwords, and crypto keys for SSH connection →

Configure a domain name:
R1(config)# ip domain-name ccnasecurity.com

Configure crypto keys for SSH.
R1(config)# crypto key generate rsa general-keys modulus 1024

Configure an admin01 user account using algorithm-type scrypt for encryption and a password of admin01pass:
R1(config)# username admin01 algorithm-type scrypt secret admin01pass

Configure line vty 0 4 to use the local user database for logins and restrict access to only SSH connections:
R1(config)# line vty 0 4 R1
(config-line)# login local
R1(config-line)# transport input ssh
R1(config-line)# exec-timeout 5 0
NOTE = Configure the enable password with strong encryption.

Step 4 and Step 5: Configure PC host IP settings and Verify connectivity→ Configure a static IP address, subnet mask, and default gateway for PC-A, PC-B, and PC-C as shown in the IP Addressing Table and verify the connectivity.

Step 6: Save the basic running configuration for each router and switch.


Part 2 and Part 3: Accessing the ASA Console and Using CLI Setup and Configuring Basic Settings →


In Part 2 of this lab, you will access the ASA via the console and use various show commands to determine hardware, software, and configuration settings. You will clear the current configuration and use the CLI interactive setup utility to configure basic ASA settings. Note: Do not configure ASA settings at this time.


In Part 3, you will configure basic settings by using the ASA CLI, even though some of them were already configured using the Setup mode interactive prompts in Part 2. In this part, you will start with the settings configured in Part 2 and then add to or modify them to create a complete basic configuration.

Step 1: Configure the hostname and domain name:

ASA-Init# config t ASA-Init(config)#
ASA-Init(config)# hostname CCNAS-AS
CCNAS-ASA(config)# domain-name ccnasecurity.com


Step 2: Configure the inside and outside interfaces:

Configure the Gi1/2 interface for the inside network (192.168.1.0/24) and set the security level to the highest setting of 100:

CCNAS-ASA(config)# interface gi1/2
CCNAS-ASA(config-if)# nameif inside
CCNAS-ASA(config-if)# ip address 192.168.1.1 255.255.255.0
CCNAS-ASA(config-if)# security-level 100
CCNAS-ASA(config-if)# no shutdown

Configure the G1/1 interface for the outside network (209.165.200.224/29), set the security level to the lowest setting of 0, and access the Gi1/1 interface:

CCNAS-ASA(config-if)# interface G1/1
CCNAS-ASA(config-if)# nameif outside INFO: Security level for “outside” set to 0 by default.
CCNAS-ASA(config-if)# ip address 209.165.200.226 255.255.255.248
CCNAS-ASA(config-if)# no shutdown

![image](https://github.com/user-attachments/assets/a193c21a-7f58-4c19-b070-80756422c386)

![image](https://github.com/user-attachments/assets/07ac09bc-c251-4be1-ac33-811ab289f35f)


Step 4: Test connectivity to the AS → .

Ensure that PC-B has a static IP address of 192.168.1.3, a subnet mask of 255.255.255.0, and a default gateway of 192.168.1.1 (the IP address of the Gi1/2 inside interface).
You should be able to ping from PC-B to the ASA inside interface address and ping from the ASA to PCB. If the pings fail, troubleshoot the configuration as necessary.

Part 3: Configuring Routing, Address Translation, and Inspection Policy Using the CLI →
In Part 3 of this lab, you will provide a default route for the ASA to reach external networks. You will configure address translation using network objects to enhance firewall security. You will then modify the default application inspection policy to allow specific traffic.


Step 1: Configure a static default route for the ASA:

Create a “quad zero” default route using the route command, associate it with the ASA outside interface, and point to the R1 G0/0 at IP address 209.165.200.225 as the gateway of last resort. The default administrative distance is one by default.
CCNAS-ASA(config)# route outside 0.0.0.0 0.0.0.0 209.165.200.225

![image](https://github.com/user-attachments/assets/d290ae65-e8ad-4066-9944-551731425df6)


Step 2: Configure address translation using PAT and network objects:

Create the network object INSIDE-NET and assign attributes to it using the subnet and nat commands:
CCNAS-ASA(config)# object network INSIDE-NET
CCNAS-ASA(config-network-object)# subnet 192.168.1.0 255.255.255.0 CCNAS-ASA(config-network-object)# nat (inside,outside) dynamic interface CCNAS-ASA(config-network-object)# end

![image](https://github.com/user-attachments/assets/54dd37de-f735-48bb-897b-7433da284436)

From PC-B, attempt to ping the R1 G0/0 interface at IP address 209.165.200.225. Notice the pings are not successful at this time as the default inspection policy does not allow ICMP to pass through the firewall.

![image](https://github.com/user-attachments/assets/0087d402-509d-4d23-a596-32425eb3174a)

![image](https://github.com/user-attachments/assets/69476592-645e-461a-b0e5-b7093c90e0f2)

Part 4: Configuring DHCP, AAA, and SSH →

In Part 4, you will configure ASA features, such as DHCP and enhanced login security, using AAA and SSH.


Step 1: Configure the ASA as a DHCP server:

Configure a DHCP address pool and enable it on the ASA inside interface. This is the range of addresses to be assigned to inside DHCP clients. Attempt to set the range from 192.168.1.5 through 192.168.1.100.

CCNAS-ASA(config)# dhcpd address 192.168.1.5–192.168.1.100 inside

CCNAS-ASA(config)# dhcpd dns 209.165.201.2

CCNAS-ASA(config)# dhcpd option 3 ip 192.168.1.1

CCNAS-ASA(config)# dhcpd enable inside

Verify DHCP:

CCNAS-ASA(config)# show run dhcpd
dhcpd dns 209.165.201.2
dhcpd option 3 ip 192.168.1.1
!
dhcpd address 192.168.1.5–192.168.1.100 inside
dhcpd enable inside
!

Step 2: Configure AAA to use the local database for authentication:

Define a local user named admin by entering the username command. Specify a password of admin01pass. CCNAS-ASA(config)# username admin password admin01pass

Configure AAA to use the local ASA database for SSH user authentication. CCNAS-ASA(config)# aaa authentication ssh console LOCAL


Step 3: Configure SSH remote access to the ASA:

CCNAS-ASA(config)# crypto key generate rsa modulus 1024 (INFO: The name for the keys will be: <Default-RSA-Key>) Keypair generation process begin. Please wait…

CCNA-ASA# write mem Building configuration…
Cryptochecksum: 43b3e351 6b3fd965 fc8c4869 b46424c8 4844 bytes copied in 0.280 secs [
OK]

CCNAS-ASA(config)# ssh 192.168.1.0 255.255.255.0 inside
CCNAS-ASA(config)# ssh 172.16.3.3 255.255.255.255 outside
CCNAS-ASA(config)# ssh timeout 10


Part 5: Configuring DMZ, Static NAT, and ACLs →

Step 1: Configure the DMZ interface Gi1/3 on the ASA:

Configure DMZ interface Gi1/3, which is where the public access web server will reside. Assign Gi1/3 the IP address 192.168.2.1/24, name it dmz, and assign a security level of 70.
CCNAS-ASA(config)# int gi1/3
CCNAS-ASA(config-if)# ip address 192.168.2.1 255.255.255.0
CCNAS-ASA(config-if)# nameif dmz (
INFO: Security level for “dmz” set to 0 by default.INFO: Security level for “dmz” set to 0 by default).

CCNAS-ASA(config-if)# security-level 70
CCNAS-ASA(config-if)# no shut


Step 2: Configure static NAT to the DMZ server using a network object:

Configure a network object named dmz-server and assign it the static IP address of the DMZ server (192.168.2.3). While in object definition mode, use the nat command to specify that this object is used to translate a DMZ address to an outside address using static NAT, and specify a public translated address of 209.165.200.227.

CCNAS-ASA(config)# object network dmz-server
CCNAS-ASA(config-network-object)# host 192.168.2.3
CCNAS-ASA(config-network-object)# nat (dmz,outside) static 209.165.200.227

Step 3: Configure an ACL to allow access to the DMZ server from the Internet →

Configure a named access list (OUTSIDE-DMZ) that permits any IP protocol from any external host to the internal IP address of the DMZ server. Apply the access list to the ASA outside interface in the IN direction:

CCNAS-ASA(config)# access-list OUTSIDE-DMZ permit ip any host 192.168.2.3 CCNAS-ASA(config)# access-group OUTSIDE-DMZ in interface outside

Step 4: Test access to the DMZ server.

Create a loopback 0 interface on Internet R2 representing an external host. Assign Lo0 IP address 172.30.1.1 and a mask of 255.255.255.0. Ping the DMZ server public address from R2 using the loopback interface as the source of the ping. The pings should be successful.

R2(config-if)# interface lo0
R2(config-if)# ip address 172.30.1.1 255.255.255.0
R2(config-if)# end R2# ping 209.165.200.227 source lo0

![image](https://github.com/user-attachments/assets/e4376e8b-a158-4602-bd67-ca5a24b0183c)

![image](https://github.com/user-attachments/assets/49264442-5eca-417f-8191-d16f08cb1284)

After your deployment has been successfully, try all the connections using ICMP Protcol. In case you have to do the troubleshoot you may use the documentation,also Youtube as a guide to solve the problem.
