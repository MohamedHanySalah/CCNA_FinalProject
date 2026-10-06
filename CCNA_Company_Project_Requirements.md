Requirements and Design Specification
1. Introduction
1.1 Purpose
This document defines the requirements, architecture, addressing plan and acceptance
criteria for the CCNA Company Project: a three-floor corporate campus network serving
six departments, with redundant switching, routing and internet connectivity. It is written
so that the complete network can be built from scratch and verified without any other
reference.
1.2 Scope
In scope: VLANs, trunking and STP; inter-VLAN routing; first-hop redundancy (HSRP);
dynamic routing (OSPF); internet edge (dual core routers, dual ISPs, NAT, failover); DHCP,
DNS, web, email and FTP services; wireless access; printers; device hardening and port
security; testing and documentation.
Out of scope: real ISP/BGP peering, IPv6, VoIP, firewall appliances, ACL-based filtering
between departments and backup-path NAT (listed as enhancements in section 16).
1.3 Project summary

| Item | Value |
|---|---|
| Company domain | project.local |
| Floors / departments | 3 floors / 6 departments |
| Access switches | 6 (Cisco 2960-24TT) |
| Multilayer switches | 2 (Cisco 3650-24PS) |
| Core routers | 2 (Cisco 2911) |
| ISP routers | 2 (Cisco 2811) |
| Servers | 2 |
| Access points | 6 (one per department) |

| Item | Value |
|---|---|
| End devices | 19 PCs, 11 laptops, 6 smartphones, 4 printers (40 clients) |
| Address space | 172.16.1.0/24 to 172.16.3.0/24 (LANs and internal links), 195.136.17.0/28 (ISP links) |
| Routing | OSPF process 1, area 0, plus floating static default routes |
| Simulator | Cisco Packet Tracer 9.0.1 |

2. Requirements
2.1 Functional requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-01 | The campus has three floors; each of the six departments gets a dedicated VLAN and IP subnet. | Must |
| FR-02 | Every access switch connects to both multilayer switches (Fa0/1 to multi-SW1, Fa0/2 to multi-SW2) over 802.1Q trunks. | Must |
| FR-03 | Inter-VLAN routing is performed on the multilayer switches using SVIs, so every department can reach every other department. | Must |
| FR-04 | First-hop redundancy: one HSRP group per VLAN, group number equal to the VLAN ID. multi-SW1 is Active (priority 110, preempt); multi-SW2 is Standby (priority 100). | Must |
| FR-05 | Spanning tree: multi-SW1 is root primary (priority 24576) and multi-SW2 root secondary (28672) for VLANs 10 to 60. | Must |
| FR-06 | Each multilayer switch has a routed (Layer 3) link to each core router: four independent /30 links. | Must |

| ID | Requirement | Priority |
|---|---|---|
| FR-07 | Two core routers; each has a serial link to each ISP (four WAN links). | Must |
| FR-08 | OSPF process 1, area 0 runs between both multilayer switches and both core routers; the core routers originate the default route. | Must |
| FR-09 | Internet access through NAT overload on the core routers for the inside network 172.16.0.0/16. | Must |
| FR-10 | Each core router has a static default route via ISP-1 (primary) and a floating static route (administrative distance 10) via ISP-2 (backup). | Must |
| FR-11 | DHCP serves all PCs, laptops and smartphones from multi-SW1, one pool per VLAN. | Must |
| FR-12 | Servers and printers use static addresses that lie outside the DHCP lease ranges. | Must |
| FR-13 | Server0 hosts the company website (HTTP/HTTPS) and DNS. Server1 hosts email (SMTP/POP3) and the cloud file service (FTP). | Must |
| FR-14 | DNS resolves www.project.local, mail.project.local and cloud.project.local. | Must |
| FR-15 | Mailboxes exist for Sales, HR, FIN, ICT and Admin; every PC, laptop and smartphone of those departments has its mail client configured. | Must |
| FR-16 | Each department has one access point with its own SSID and channel; every wireless client is associated with the access point of its own department. | Must |
| FR-17 | Wireless networks use WPA2-PSK (AES) with a department passphrase. | Should |
| FR-18 | Printers sit in the VLAN of their department and use static IP addresses. | Must |

| ID | Requirement | Priority |
|---|---|---|
| FR-19 | Every switch and router has hostname, banner, console password, enable secret, SSH version 2 with a local user, and vty lines restricted to SSH. | Must |
| FR-20 | Used access ports have sticky port security (maximum 1 MAC address; 5 on access-point ports). Unused ports are administratively shut down. | Must |
| FR-21 | Each switch has a management SVI in its own VLAN and a default gateway equal to the HSRP virtual IP. | Must |
| FR-22 | All device configurations are saved (startup-config equals running-config). | Must |
| FR-23 | The project is delivered with this specification, the IP plan, configuration listings and test evidence. | Must |

2.2 Non-functional requirements
Availability: the loss of any single multilayer switch, core router, ISP or uplink must not
interrupt inter-department traffic or internet reachability (see section 16 for the NAT
limitation on the backup path). DHCP and the two servers are single instances; this is
an accepted risk.
Security: no clear-text secrets in configurations, no Telnet, no unused open ports, a
warning banner on every device.
Scalability: office subnets are /25 (126 hosts) with ample growth room; link and server
subnets are sized exactly (/30 and /28).
Maintainability: consistent naming, HSRP group number equals VLAN ID, one
password pattern per device.
Compatibility: the project file opens in Cisco Packet Tracer 9.0.1.

3. Network Architecture
3.1 Hierarchy

| Layer | Devices | Function |
|---|---|---|
| Access | Sales-SW, HR-SW, FIN-SW, ICT-SW, Admin-SW, Server-SW (2960-24TT) | Endpoint attachment, VLAN membership, port security |
| Distribution and core (collapsed) | multi-SW1, multi-SW2 (3650- 24PS) | Inter-VLAN routing, HSRP, STP root, DHCP, OSPF |
| Edge | CORE-1, CORE-2 (2911) | OSPF, NAT, default routes to the internet |
| Provider | ISP-1, ISP-2 (2811) | Simulated internet (loopbacks 8.8.8.8 and 9.9.9.9) |

3.2 Floors and departments

| Floor | Department | VLAN | Access switch | SSID |
|---|---|---|---|---|
| First | Sales and Marketing | 10 | Sales-SW | Sales-WiFi |
| First | HR and Logistics | 20 | HR-SW | HR-WiFi |
| Second | Finance and Accounts | 30 | FIN-SW | FIN-WiFi |
| Second | ICT | 40 | ICT-SW | ICT-WiFi |
| Third | Admin and Public Relations | 50 | Admin-SW | Admin-WiFi |
| Third | Server Room | 60 | Server-SW | Server-WiFi |

Note: the original addressing sheet lists Admin on the second floor and ICT on the third,
while the Packet Tracer workspace places ICT on the second floor and Admin on the third.
VLAN and IP assignments are identical in both; only the floor label differs.
3.3 Logical topology
Every access switch: Fa0/1 to multi-SW1 and Fa0/2 to multi-SW2.

multi-SW1 and multi-SW2 are interconnected (multi-SW1 Gi1/0/10 to multi-SW2 Gi1/0/8).
multi-SW1 Gi1/0/1 to CORE-1 Gi0/0; multi-SW1 Gi1/0/2 to CORE-2 Gi0/0; multi-SW2
Gi1/0/1 to CORE-1 Gi0/1; multi-SW2 Gi1/0/2 to CORE-2 Gi0/1.
CORE-1 Se0/0/0 to ISP-1 Se0/2/0; CORE-1 Se0/0/1 to ISP-2 Se0/2/0; CORE-2 Se0/0/0 to
ISP-2 Se0/2/1; CORE-2 Se0/0/1 to ISP-1 Se0/2/1.
4. Device and Endpoint Inventory
4.1 Infrastructure

| Hostname | Packet Tracer label | Model | Notes |
|---|---|---|---|
| CORE-1, CORE-2 | CORE-1, CORE-2 | Cisco 2911 | HWIC-2T serial card (Se0/0/0, Se0/0/1) |
| ISP-1, ISP-2 | IPS-1, IPS-2 | Cisco 2811 | 2-port serial card (Se0/2/0, Se0/2/1). Rename the labels to ISP-1 and ISP-2. |
| Multi-SW1, Multi-SW2 | multi-SW1, multi-SW2 | Cisco 3650- 24PS | Layer 3 switching, Gi1/0/1 to Gi1/0/24 |
| Sales-SW, HR-SW, FIN- SW, ICT-SW, Admin- SW, Server-SW | same (Server-SW label: Home-Server- SW) | Cisco 2960- 24TT | Fa0/1-24 |
| Access points | Access Point-Sales, - HR, -FIN, -ICT, - Admin, -Home-Server | AccessPoint- PT | One per department |
| Server0, Server1 | Server0, Server1 | Server-PT | Web/DNS and Mail/FTP |

4.2 Endpoints by department

| Department | PCs | Laptops (WPC300N) | Smartphones | Printers | Servers |
|---|---|---|---|---|---|
| Sales | PC0, PC1, PC2, PC3 (4) | Laptop0, 1, 2 (3) | Smartphone0 (1) | none | none |

| Department | PCs | Laptops (WPC300N) | Smartphones | Printers | Servers |
|---|---|---|---|---|---|
| HR | PC5, PC6 (2) | Laptop3, 4, 5 (3) | none | Printer1, Printer2 (2) | none |
| FIN | PC7, PC8, PC9, PC10, PC11 (5) | Laptop6, 7 (2) | Smartphone1, 2 (2) | Printer0 (1) | none |
| ICT | PC12, PC13, PC14, PC15 (4) | none | Smartphone3, 4 (2) | none | none |
| Admin | PC16, PC17 (2) | Laptop8 (1) | Smartphone5 (1) | Printer3 (1) | none |
| Server | PC18, PC19 (2) | Laptop9, 10 (2) | none | none | Server0, Server1 (2) |
| Total | 19 | 11 | 6 | 4 | 2 |

5. Naming Conventions

| Object | Convention | Examples |
|---|---|---|
| Access switch hostname | Department-SW | Sales-SW, HR-SW, FIN-SW, ICT-SW, Admin-SW, Server-SW |
| Multilayer switch hostname | Multi-SWn | Multi-SW1, Multi-SW2 |
| Core router hostname | CORE-n | CORE-1, CORE-2 |
| ISP router hostname | ISP-n | ISP-1, ISP-2 |
| SSH username | hostname in lower case | sales-sw, hr-sw, multi-sw1, core-1, isp-1 |
| Passwords | name of the department or device group followed by 123.0 | sales123.0, multi123.0, core123.0, isp123.0 |
| VLAN names | Sales, HR, Finance, ICT, Admin, Server | VLAN 30 is named Finance |
| HSRP group | equals the VLAN ID | VLAN 20 uses group 20 |
| Domain | project.local | mail.project.local |

| Object | Convention | Examples |
|---|---|---|
| Banner | only authorized access!! | all devices |

6. VLAN Plan

| VLAN ID | Name | Department | Subnet | HSRP virtual IP (default gateway) |
|---|---|---|---|---|
| 10 | Sales | Sales and Marketing | 172.16.1.0/25 | 172.16.1.1 |
| 20 | HR | HR and Logistics | 172.16.1.128/25 | 172.16.1.129 |
| 30 | Finance | Finance and Accounts | 172.16.2.0/25 | 172.16.2.1 |
| 40 | ICT | ICT | 172.16.3.0/25 | 172.16.3.1 |
| 50 | Admin | Admin and Public Relations | 172.16.2.128/25 | 172.16.2.129 |
| 60 | Server | Server Room | 172.16.3.128/28 | 172.16.3.129 |

7. IP Addressing and Subnetting
7.1 Method
The base is the private block 172.16.0.0/16. The third octet identifies the floor group:
172.16.1.x for Sales and HR, 172.16.2.x for Finance and Admin, 172.16.3.x for ICT, the Server
Room and the internal links. Each /24 is split into two /25 halves (.0 and .128). The Server
Room needs only 14 hosts, so its half is reduced to a /28 (172.16.3.128/28). That frees
172.16.3.144 to 172.16.3.159 (172.16.3.144/28) for four /30 point-to-point links. The four ISP
links use 195.136.17.0/28, also split into four /30 subnets.

| Subnet | Devices today (clients, AP, infrastructure) | Prefix chosen | Usable hosts |
|---|---|---|---|
| Sales | 8 clients + 1 AP + 4 infrastructure = 13 | /25 | 126 |
| HR | 5 clients + 2 printers + 1 AP + 4 infrastructure = 12 | /25 | 126 |

| Subnet | Devices today (clients, AP, infrastructure) | Prefix chosen | Usable hosts |
|---|---|---|---|
| Finance | 9 clients + 1 printer + 1 AP + 4 = 15 | /25 | 126 |
| ICT | 6 clients + 1 AP + 4 = 11 | /25 | 126 |
| Admin | 4 clients + 1 printer + 1 AP + 4 = 10 | /25 | 126 |
| Server Room | 4 clients + 2 servers + 1 AP + 4 = 11 | /28 | 14 |
| Each point-to-point link | 2 | /30 | 2 |

7.2 Subnet table

| Subnet | VLAN | Network/prefix | Mask | Wildcard | First usable | Last usable | Broadcast | Usable |
|---|---|---|---|---|---|---|---|---|
| Sales | 10 | 172.16.1.0/25 | 255.255.255.128 | 0.0.0.127 | 172.16.1.1 | 172.16.1.126 | 172.16.1.127 | 126 |
| HR | 20 | 172.16.1.128/25 | 255.255.255.128 | 0.0.0.127 | 172.16.1.129 | 172.16.1.254 | 172.16.1.255 | 126 |
| Finance | 30 | 172.16.2.0/25 | 255.255.255.128 | 0.0.0.127 | 172.16.2.1 | 172.16.2.126 | 172.16.2.127 | 126 |
| Admin | 50 | 172.16.2.128/25 | 255.255.255.128 | 0.0.0.127 | 172.16.2.129 | 172.16.2.254 | 172.16.2.255 | 126 |
| ICT | 40 | 172.16.3.0/25 | 255.255.255.128 | 0.0.0.127 | 172.16.3.1 | 172.16.3.126 | 172.16.3.127 | 126 |
| Server Room | 60 | 172.16.3.128/28 | 255.255.255.240 | 0.0.0.15 | 172.16.3.129 | 172.16.3.142 | 172.16.3.143 | 14 |
| Internal links (block) | n/a | 172.16.3.144/28 | 255.255.255.240 | 0.0.0.15 | n/a | n/a | 172.16.3.159 | four /30 |
| ISP links (block) | n/a | 195.136.17.0/28 | 255.255.255.240 | 0.0.0.15 | n/a | n/a | 195.136.17.15 | four /30 |

7.3 Allocation inside each LAN subnet

| Subnet | HSRP VIP | Access switch SVI | multi-SW1 SVI | multi-SW2 SVI | Static hosts | Excluded from DHCP | DHCP lease range |
|---|---|---|---|---|---|---|---|
| Sales | 172.16.1.1 | 172.16.1.2 | 172.16.1.3 | 172.16.1.4 | 172.16.1.5-9 reserved | 172.16.1.1- 172.16.1.9 | 172.16.1.10- 172.16.1.126 |
| HR | 172.16.1.129 | 172.16.1.130 | 172.16.1.131 | 172.16.1.132 | Printer1 172.16.1.136, Printer2 172.16.1.137 | 172.16.1.129- 172.16.1.138 | 172.16.1.139- 172.16.1.254 |

| Subnet | HSRP VIP | Access switch SVI | multi-SW1 SVI | multi-SW2 SVI | Static hosts | Excluded from DHCP | DHCP lease range |
|---|---|---|---|---|---|---|---|
| Finance | 172.16.2.1 | 172.16.2.2 | 172.16.2.3 | 172.16.2.4 | Printer0 172.16.2.5 | 172.16.2.1- 172.16.2.9 | 172.16.2.10- 172.16.2.126 |
| ICT | 172.16.3.1 | 172.16.3.2 | 172.16.3.3 | 172.16.3.4 | 172.16.3.5-9 reserved | 172.16.3.1- 172.16.3.9 | 172.16.3.10- 172.16.3.126 |
| Admin | 172.16.2.129 | 172.16.2.130 | 172.16.2.131 | 172.16.2.132 | Printer3 172.16.2.135 | 172.16.2.129- 172.16.2.138 | 172.16.2.139- 172.16.2.254 |
| Server Room | 172.16.3.129 | 172.16.3.130 | 172.16.3.131 | 172.16.3.132 | Server0 172.16.3.133, Server1 172.16.3.134 | 172.16.3.129- 172.16.3.134 | 172.16.3.135- 172.16.3.142 |

7.4 Point-to-point links

| Link | Subnet | End A | End B |
|---|---|---|---|
| multi-SW1 to CORE-1 | 172.16.3.144/30 | multi-SW1 Gi1/0/1: 172.16.3.145 | CORE-1 Gi0/0: 172.16.3.146 |
| multi-SW2 to CORE-1 | 172.16.3.148/30 | multi-SW2 Gi1/0/1: 172.16.3.149 | CORE-1 Gi0/1: 172.16.3.150 |
| multi-SW1 to CORE-2 | 172.16.3.152/30 | multi-SW1 Gi1/0/2: 172.16.3.153 | CORE-2 Gi0/0: 172.16.3.154 |
| multi-SW2 to CORE-2 | 172.16.3.156/30 | multi-SW2 Gi1/0/2: 172.16.3.157 | CORE-2 Gi0/1: 172.16.3.158 |
| CORE-1 to ISP-1 | 195.136.17.0/30 | CORE-1 Se0/0/0: 195.136.17.1 (DCE, clock rate 2000000) | ISP-1 Se0/2/0: 195.136.17.2 |
| CORE-1 to ISP-2 | 195.136.17.4/30 | CORE-1 Se0/0/1: 195.136.17.5 | ISP-2 Se0/2/0: 195.136.17.6 (DCE) |
| CORE-2 to ISP-1 | 195.136.17.8/30 | CORE-2 Se0/0/1: 195.136.17.9 | ISP-1 Se0/2/1: 195.136.17.10 (DCE) |
| CORE-2 to ISP-2 | 195.136.17.12/30 | CORE-2 Se0/0/0: 195.136.17.13 | ISP-2 Se0/2/1: 195.136.17.14 (DCE) |

Every /30 has network address ending .0, .4, .8 or .12 (plus the base), two usable hosts and
a broadcast address ending .3, .7, .11 or .15.
7.5 Router IDs and loopbacks

| Device | OSPF router ID | Loopback |
|---|---|---|
| CORE-1 | 1.1.1.1 | none |
| CORE-2 | 2.2.2.2 | none |
| Multi-SW1 | 3.3.3.3 | none |
| Multi-SW2 | 4.4.4.4 | none |
| ISP-1 | n/a | Loopback0 8.8.8.8/32 (simulated internet host) |
| ISP-2 | n/a | Loopback0 9.9.9.9/32 (simulated internet host) |

8. Physical Connectivity
8.1 Uplinks of the access switches

| Access switch | Fa0/1 connects to | Fa0/2 connects to |
|---|---|---|
| Sales-SW | multi-SW1 Gi1/0/3 | multi-SW2 Gi1/0/9 |
| HR-SW | multi-SW1 Gi1/0/4 | multi-SW2 Gi1/0/7 |
| FIN-SW | multi-SW1 Gi1/0/5 | multi-SW2 Gi1/0/6 |
| ICT-SW | multi-SW1 Gi1/0/6 | multi-SW2 Gi1/0/5 |
| Admin-SW | multi-SW1 Gi1/0/7 | multi-SW2 Gi1/0/4 |
| Server-SW | multi-SW1 Gi1/0/8 | multi-SW2 Gi1/0/3 |

Inter-switch link: multi-SW1 Gi1/0/10 to multi-SW2 Gi1/0/8. Gi1/0/3 to Gi1/0/10 are trunk
ports on both multilayer switches.

8.2 Endpoint ports on the access switches

| Switch | Port | Device | Port security |
|---|---|---|---|
| Sales-SW | Fa0/3, Fa0/4, Fa0/5, Fa0/6 | PC0, PC2, PC1, PC3 | sticky, max 1 |
| Sales-SW | Fa0/7 | Access Point-Sales | sticky, max 5 |
| HR-SW | Fa0/3, Fa0/4 | PC5, PC6 | sticky, max 1 |
| HR-SW | Fa0/5, Fa0/6 | Printer1, Printer2 | sticky, max 1 |
| HR-SW | Fa0/7 | Access Point-HR | sticky, max 5 |
| FIN-SW | Fa0/3, Fa0/4, Fa0/6, Fa0/7, Fa0/8 | PC7, PC8, PC9, PC11, PC10 | sticky, max 1 |
| FIN-SW | Fa0/5 | Printer0 | sticky, max 1 |
| FIN-SW | Fa0/9 | Access Point-FIN | sticky, max 5 |
| ICT-SW | Fa0/3, Fa0/4, Fa0/5, Fa0/6 | PC12, PC13, PC15, PC14 | sticky, max 1 |
| ICT-SW | Fa0/7 | Access Point-ICT | sticky, max 5 |
| Admin-SW | Fa0/3, Fa0/4 | PC16, PC17 | sticky, max 1 |
| Admin-SW | Fa0/5 | Printer3 | sticky, max 1 |
| Admin-SW | Fa0/6 | Access Point-Admin | sticky, max 5 |
| Server-SW | Fa0/3, Fa0/4 | Server0, Server1 | sticky, max 1 |
| Server-SW | Fa0/5, Fa0/6 | PC18, PC19 | sticky, max 1 |
| Server-SW | Fa0/7 | Access Point-Home-Server | sticky, max 5 |

All remaining ports from Fa0/3 to Fa0/24 on every access switch are shut down.
9. Layer 2 Design
9.1 VLAN database

| Device | VLANs defined (plus default VLAN 1) |
|---|---|
| Sales-SW | 10 Sales |

| Device | VLANs defined (plus default VLAN 1) |
|---|---|
| HR-SW | 20 HR |
| FIN-SW | 30 Finance |
| ICT-SW | 40 ICT |
| Admin-SW | 50 Admin |
| Server-SW | 60 Server |
| Multi-SW1, Multi-SW2 | 10 Sales, 20 HR, 30 Finance, 40 ICT, 50 Admin, 60 Server |

9.2 Trunks and access ports
Access switches, Fa0/1 and Fa0/2: switchport mode trunk (802.1Q, native VLAN 1, all
VLANs allowed).
Multilayer switches, Gi1/0/3 to Gi1/0/10: switchport mode trunk (802.1Q).
Endpoint ports: switchport mode access and switchport access vlan <id> of the
department.
9.3 Spanning tree
PVST (default mode). Multi-SW1: spanning-tree vlan 10,20,30,40,50,60 priority
24576 (root primary). Multi-SW2: priority 28672 (root secondary). Each access switch has
one uplink to each multilayer switch, so STP blocks one uplink per VLAN and there are no
loops. If multi-SW1 fails, the Fa0/2 uplink starts forwarding. Because multi-SW1 is also the
HSRP Active router, the Layer 2 path and the Layer 3 path coincide.
9.4 Port security (endpoint ports in use)
switchport port-security , switchport port-security maximum 5 (access-point
ports only; default 1 elsewhere), switchport port-security mac-address sticky . The
violation mode is the default (shutdown). The sticky MAC addresses are learned on the
first traffic, so every device must ping once after the build. To recover an err-disabled port
use shutdown then no shutdown .

9.5 Unused ports
Every port from Fa0/3 to Fa0/24 that has no device (section 8.2) is administratively shut
down.
9.6 Management SVIs of the access switches

| Switch | SVI | IP address / mask | Default gateway |
|---|---|---|---|
| Sales-SW | Vlan10 | 172.16.1.2 / 255.255.255.128 | 172.16.1.1 |
| HR-SW | Vlan20 | 172.16.1.130 / 255.255.255.128 | 172.16.1.129 |
| FIN-SW | Vlan30 | 172.16.2.2 / 255.255.255.128 | 172.16.2.1 |
| ICT-SW | Vlan40 | 172.16.3.2 / 255.255.255.128 | 172.16.3.1 |
| Admin-SW | Vlan50 | 172.16.2.130 / 255.255.255.128 | 172.16.2.129 |
| Server-SW | Vlan60 | 172.16.3.130 / 255.255.255.240 | 172.16.3.129 |

10. Layer 3 Design
10.1 SVIs and HSRP

| VLAN | HSRP group | Virtual IP | multi-SW1 SVI (priority 110, preempt) | multi-SW2 SVI (priority 100) |
|---|---|---|---|---|
| 10 | 10 | 172.16.1.1 | 172.16.1.3 /25 | 172.16.1.4 /25 |
| 20 | 20 | 172.16.1.129 | 172.16.1.131 /25 | 172.16.1.132 /25 |
| 30 | 30 | 172.16.2.1 | 172.16.2.3 /25 | 172.16.2.4 /25 |
| 40 | 40 | 172.16.3.1 | 172.16.3.3 /25 | 172.16.3.4 /25 |
| 50 | 50 | 172.16.2.129 | 172.16.2.131 /25 | 172.16.2.132 /25 |
| 60 | 60 | 172.16.3.129 | 172.16.3.131 /28 | 172.16.3.132 /28 |

ip routing is enabled on both multilayer switches. Gi1/0/1 and Gi1/0/2 are routed ports
( no switchport ) with the /30 addresses of section 7.4.

10.2 OSPF
Process 1, area 0, statement network 172.16.0.0 0.0.255.255 area 0 on Multi-SW1
(router ID 3.3.3.3), Multi-SW2 (4.4.4.4), CORE-1 (1.1.1.1) and CORE-2 (2.2.2.2). On the
multilayer switches passive-interface default is set and only Gi1/0/1 and Gi1/0/2 are
made active, so OSPF adjacencies form towards the core routers and never towards
users. CORE-1 and CORE-2 run default-information originate . Expected result: each
multilayer switch has two FULL neighbours (CORE-1 and CORE-2) and an OSPF default
route.
10.3 Default routes and WAN failover

| Router | Primary default route | Backup (floating, distance 10) |
|---|---|---|
| CORE-1 | 0.0.0.0/0 via 195.136.17.2 (ISP-1) | 0.0.0.0/0 via 195.136.17.6 (ISP-2) |
| CORE-2 | 0.0.0.0/0 via 195.136.17.10 (ISP-1) | 0.0.0.0/0 via 195.136.17.14 (ISP-2) |

10.4 NAT
Standard ACL: access-list 1 permit 172.16.0.0 0.0.255.255 . On both core routers
Gi0/0 and Gi0/1 are ip nat inside and Se0/0/0 and Se0/0/1 are ip nat outside . CORE-
1: ip nat inside source list 1 interface Serial0/0/0 overload . CORE-2: ip nat
inside source list 1 interface Serial0/0/1 overload . Both statements use the link
to ISP-1 (the primary path).
10.5 ISP simulation
ISP-1 and ISP-2 have only connected networks and a loopback (8.8.8.8 and 9.9.9.9). They
need no route to the inside network because NAT translates inside addresses to the serial
addresses of the core routers. The DCE end of each serial link carries clock rate
2000000 .

11. Services
11.1 DHCP (configured on Multi-SW1)

| Pool | Network | Mask | Default router | DNS server | Excluded addresses |
|---|---|---|---|---|---|
| Sales | 172.16.1.0 | 255.255.255.128 | 172.16.1.1 | 172.16.3.133 | 172.16.1.1 - 172.16.1.9 |
| HR | 172.16.1.128 | 255.255.255.128 | 172.16.1.129 | 172.16.3.133 | 172.16.1.129 - 172.16.1.138 |
| FIN | 172.16.2.0 | 255.255.255.128 | 172.16.2.1 | 172.16.3.133 | 172.16.2.1 - 172.16.2.9 |
| ICT | 172.16.3.0 | 255.255.255.128 | 172.16.3.1 | 172.16.3.133 | 172.16.3.1 - 172.16.3.9 |
| Admin | 172.16.2.128 | 255.255.255.128 | 172.16.2.129 | 172.16.3.133 | 172.16.2.129 - 172.16.2.138 |
| Server | 172.16.3.128 | 255.255.255.240 | 172.16.3.129 | 172.16.3.133 | 172.16.3.129 - 172.16.3.134 |

All PCs, laptops and smartphones are set to DHCP (Desktop, IP Configuration). The
default router handed to clients is the HSRP virtual IP.
11.2 Server0: website and DNS (172.16.3.133/28)
HTTP and HTTPS: On. The index page carries the company name.
DNS: On, with these A records:

| Name | Address |
|---|---|
| www.project.local | 172.16.3.133 |
| mail.project.local | 172.16.3.134 |
| cloud.project.local | 172.16.3.134 |

11.3 Server1: email and cloud storage (172.16.3.134/28)
Email: SMTP and POP3 On, domain project.local, five mailboxes:

| User | Address | Password | Department |
|---|---|---|---|
| sales | sales@project.local | sales123.0 | Sales |
| hr | hr@project.local | hr123.0 | HR |
| fin | fin@project.local | fin123.0 | Finance |

| User | Address | Password | Department |
|---|---|---|---|
| ict | ict@project.local | ict123.0 | ICT |
| admin | admin@project.local | admin123.0 | Admin |

FTP (cloud storage): On, user cloud, password cloud123.0, permissions RWDNL. Packet
Tracer also ships the default account cisco/cisco, which should be removed in a real
deployment.
11.4 Mail client settings
Every PC, laptop and smartphone of Sales, HR, FIN, ICT and Admin uses: Your Name =
department name; Email Address = user@project.local; Incoming and Outgoing Mail
Server = 172.16.3.134; User Name = user; Password = user123.0. The Server Room devices
(PC18, PC19, Laptop9, Laptop10) have no mailbox.
11.5 Static addressing of servers and printers

| Device | Connected to | IP address | Mask | Gateway | DNS |
|---|---|---|---|---|---|
| Server0 | Server-SW Fa0/3 | 172.16.3.133 | 255.255.255.240 | 172.16.3.129 | 172.16.3.133 |
| Server1 | Server-SW Fa0/4 | 172.16.3.134 | 255.255.255.240 | 172.16.3.129 | 172.16.3.133 |
| Printer0 | FIN-SW Fa0/5 | 172.16.2.5 | 255.255.255.128 | 172.16.2.1 | 172.16.3.133 |
| Printer1 | HR-SW Fa0/5 | 172.16.1.136 | 255.255.255.128 | 172.16.1.129 | 172.16.3.133 |
| Printer2 | HR-SW Fa0/6 | 172.16.1.137 | 255.255.255.128 | 172.16.1.129 | 172.16.3.133 |
| Printer3 | Admin-SW Fa0/5 | 172.16.2.135 | 255.255.255.128 | 172.16.2.129 | 172.16.3.133 |

12. Wireless Design

| Department | Access point | Wired to | SSID | Channel | Security (WPA2-PSK, AES) | Wireless clients |
|---|---|---|---|---|---|---|
| Sales | Access Point- Sales | Sales-SW Fa0/7 | Sales- WiFi | 1 | sales123.0 | Laptop0, Laptop1, Laptop2, Smartphone0 |
| HR | Access Point- HR | HR-SW Fa0/7 | HR-WiFi | 6 | hr123.0 | Laptop3, Laptop4, Laptop5 |

| Department | Access point | Wired to | SSID | Channel | Security (WPA2-PSK, AES) | Wireless clients |
|---|---|---|---|---|---|---|
| FIN | Access Point- FIN | FIN-SW Fa0/9 | FIN-WiFi | 11 | fin123.0 | Laptop6, Laptop7, Smartphone1, Smartphone2 |
| ICT | Access Point- ICT | ICT-SW Fa0/7 | ICT-WiFi | 1 | ict123.0 | Smartphone3, Smartphone4 |
| Admin | Access Point- Admin | Admin- SW Fa0/6 | Admin- WiFi | 6 | admin123.0 | Laptop8, Smartphone5 |
| Server | Access Point- Home-Server | Server- SW Fa0/7 | Server- WiFi | 11 | server123.0 | Laptop9, Laptop10 |

Laptops need the WPC300N module: power off, remove the copper card, insert
WPC300N, power on. Smartphones use their built-in wireless adapter.
Channels 1, 6 and 11 do not overlap, and a channel is reused only on a different floor
(Sales and ICT on channel 1, HR and Admin on 6, FIN and Server on 11).
Each client is configured with the SSID and passphrase of its own department only,
and uses DHCP.

13. Security Hardening
13.1 Baseline configuration for every switch and router
hostname <name>
no ip domain-lookup
service password-encryption
banner motd #only authorized access!!#
enable secret <password>
line console 0
password <password>
login
ip domain-name project.local
username <ssh-user> secret <password>
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
line vty 0 4
login local
transport input ssh
end
write memory
On the multilayer switches the vty range is line vty 0 15 . Passwords and users are
listed in Appendix A.
13.2 Security controls summary

| Control | Where |
|---|---|
| Encrypted privileged password (enable secret) | all 12 network devices |
| Password encryption for line passwords | all 12 network devices |
| SSH version 2 only, Telnet disabled | all 12 network devices |
| Warning banner | all 12 network devices |
| Sticky port security | endpoint ports of the six access switches |
| Unused ports shut down | six access switches |
| DNS lookup disabled | all 12 network devices |

14. Acceptance Test Plan

| ID | Test | Method | Expected result |
|---|---|---|---|
| TC- 01 | Link status | Wait about 1 minute after power-on | All links green |
| TC- 02 | VLANs | show vlan brief on every switch | VLANs of section 9.1 exist; endpoint ports sit in the department VLAN |
| TC- 03 | Trunks | show interfaces trunk | Fa0/1-2 on access switches and Gi1/0/3-10 on multilayer switches trunking |
| TC- 04 | STP root | show spanning-tree vlan 10 on Multi-SW1 | This bridge is the root |
| TC- 05 | HSRP | show standby brief | Six groups; Multi-SW1 Active, Multi-SW2 Standby |
| TC- 06 | DHCP | ipconfig on every client | Address in the lease range of its VLAN, gateway = virtual IP, DNS 172.16.3.133 |
| TC- 07 | Same department | Ping between two devices of one department | Success |
| TC- 08 | Between departments | Ping from a Sales PC to a PC in HR, FIN, ICT and Admin | Success (the first packet may time out because of ARP) |
| TC- 09 | Gateways | Ping each virtual IP from a client | Success |
| TC- 10 | OSPF | show ip ospf neighbor on Multi-SW1, Multi-SW2, CORE-1, CORE-2 | Each multilayer switch has two FULL neighbours |
| TC- 11 | Routing table | show ip route on a multilayer switch | All VLAN subnets plus a default route |
| TC- 12 | Internet | Ping 8.8.8.8 from a PC in every department | Success |

| ID | Test | Method | Expected result |
|---|---|---|---|
| TC- 13 | NAT | show ip nat translations on CORE-1 | ICMP entries after TC-12 |
| TC- 14 | Core router failure | Power off CORE-1, ping 8.8.8.8 | Traffic continues through CORE-2 |
| TC- 15 | Multilayer switch failure | Power off Multi-SW1, ping a virtual IP | Connectivity returns through Multi-SW2 within seconds |
| TC- 16 | DNS and website | Browse to www.project.local and to http://172.16.3.133 | Company page opens |
| TC- 17 | Email | Send from a Sales device to hr@project.local, press Receive on an HR device | Message delivered |
| TC- 18 | Cloud FTP | ftp 172.16.3.134 with user cloud | Login succeeds |
| TC- 19 | Wireless | Check each laptop and smartphone | Associated with its own SSID, DHCP address from its own VLAN |
| TC- 20 | Printers | Ping each printer from a PC in another department | Success |
| TC- 21 | Port security | show port-security ; connect an unknown device to a used port | Port becomes err-disabled |
| TC- 22 | SSH | From a PC in the same VLAN: ssh -l sales-sw 172.16.1.2 | Login prompt; Telnet is refused |
| TC- 23 | Secrets | show running-config | enable secret shown as type 5; no clear-text secrets |
| TC- 24 | Persistence | show startup-config | Identical to running-config on every device |

15. Implementation Sequence
1. Place the devices, connect the cables of section 8, power everything on.

2. Rename labels and hostnames (section 5).
3. Apply the baseline of section 13.1 to all 12 network devices.
4. ISPs: serial addresses, clock rates, loopbacks.
5. Core routers: interfaces, OSPF, default routes, NAT.
6. Multilayer switches: VLANs, ip routing , routed ports, trunks, SVIs, HSRP, STP, OSPF,
DHCP pools.
7. Access switches: VLAN, trunks, access ports, management SVI, port security, unused
ports shut down.
8. Servers: static address and services (section 11).
9. Clients: DHCP, wireless module, SSID and WPA2, mail client.
10. Printers: static addresses.
11. Run the test plan of section 14 and correct any failure.
12. Save every device ( write memory ), save the .pkt file and export the documentation.
Deliverables: this specification, the addressing tables (sections 6 to 8), configuration
listings of all 12 network devices, the Packet Tracer file, and test evidence for TC-01 to TC-
24.
16. Assumptions, Limitations and Enhancements
Assumptions: Cisco Packet Tracer 9.0.1; the device models of section 4; 195.136.17.0/28 is
used as lab address space for the ISP links; DHCP is centralised on Multi-SW1; all
passwords follow the lab pattern and are for the lab only.

| Limitation | Enhancement |
|---|---|
| DHCP runs only on Multi-SW1 | Add DHCP on Multi-SW2 with split ranges, or use ip helper-address towards a redundant server |
| NAT is bound to the link towards ISP-1; if that link fails, the backup path forwards without NAT and the ISP drops the replies | Use route-maps with one NAT statement per ISP, or IP SLA tracking |
| No ACLs between departments | Allow departments to reach only the servers (HTTP, HTTPS, DNS, SMTP, POP3, FTP) and block Sales to Admin |

| Limitation | Enhancement |
|---|---|
| One access point and one shared WPA2 passphrase per department | Add WPA2-Enterprise with RADIUS and more access points |
| Server Room staff have no mailbox | Add a server mailbox on Server1 |
| Default FTP account cisco/cisco on the servers | Remove it |
| RSA keys are generated on each device and are not part of the config text | Generate the keys on all devices before testing SSH |

Appendix A. Credentials (lab use only)

| Device | Password (console, enable secret, SSH) | SSH username |
|---|---|---|
| Sales-SW | sales123.0 | sales-sw |
| HR-SW | hr123.0 | hr-sw |
| FIN-SW | fin123.0 | fin-sw |
| ICT-SW | ict123.0 | ict-sw |
| Admin-SW | admin123.0 | admin-sw |
| Server-SW | server123.0 | server-sw |
| Multi-SW1 | multi123.0 | multi-sw1 |
| Multi-SW2 | multi123.0 | multi-sw2 |
| CORE-1 | core123.0 | core-1 |
| CORE-2 | core123.0 | core-2 |
| ISP-1 | isp123.0 | isp-1 |
| ISP-2 | isp123.0 | isp-2 |

Wi-Fi passphrases are in section 12, mailboxes in section 11.3, and the FTP account is cloud
/ cloud123.0.

Appendix B. Verification Commands

| Device type |  | Commands |  |
|---|---|---|---|
| Access switch |  | show vlan brief , show interfaces trunk , show |  |
|  |  | interfaces status , show port-security , show spanning-tree |  |
| Multilayer switch |  | show ip route , show ip ospf neighbor , show standby brief , show ip dhcp binding , show ip interface brief |  |
| Core router |  | show ip route , show ip ospf neighbor , show |  |
|  |  | ip nat translations , show ip interface brief |  |
| ISP router |  | show ip interface brief , ping |  |
| Client |  | ipconfig /all , ping , nslookup , Web Browser, Email |  |

Core router show ip route , show ip ospf neighbor , show
ip nat translations , show ip interface brief
ISP router show ip interface brief , ping
Client ipconfig /all , ping , nslookup , Web Browser,
Email

| Appendix C. Glossary Term Meaning SVI Switch Virtual Interface: the Layer 3 interface of a VLAN on a multilayer switch HSRP Hot Standby Router Protocol: gives clients one virtual gateway that survives a device failure OSPF Open Shortest Path First: link-state routing protocol NAT overload Many inside addresses share one outside address through port translation STP / PVST Spanning Tree Protocol / per-VLAN spanning tree, prevents Layer 2 loops Sticky MAC Port-security MAC address learned dynamically and stored in the configuration DCE Serial interface that provides the clock signal |
|---|

| Term | Meaning |
|---|---|
| SVI | Switch Virtual Interface: the Layer 3 interface of a VLAN on a multilayer switch |
| HSRP | Hot Standby Router Protocol: gives clients one virtual gateway that survives a device failure |
| OSPF | Open Shortest Path First: link-state routing protocol |
| NAT overload | Many inside addresses share one outside address through port translation |
| STP / PVST | Spanning Tree Protocol / per-VLAN spanning tree, prevents Layer 2 loops |
| Sticky MAC | Port-security MAC address learned dynamically and stored in the configuration |
| DCE | Serial interface that provides the clock signal |
