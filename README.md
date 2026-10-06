<img width="1312" height="1199" alt="image" src="https://github.com/user-attachments/assets/c8296f8f-7438-4b3c-a6d9-bda56abf5a7b" /># CCNA_Final_Project
### Project Overview

The project focuses on the **design and implementation of a three-floor Corporate Network**, with the main objective of providing a **structured, secure, reliable, and fault-tolerant network infrastructure**.

The company is divided into **six departments**:

* **Sales**
* **HR**
* **Finance**
* **ICT**
* **Admin**
* **Server**

Each department is separated using **VLANs**, providing logical network segmentation while allowing controlled communication between departments through **Inter-VLAN Routing**.

### Network Components

The network consists of:

* **6 Access Switches** for connecting end-user devices.
* **2 Multilayer Switches** for Inter-VLAN Routing and network redundancy.
* **2 Core Routers** for connecting the corporate network to the Internet.
* **2 ISP Routers** to provide primary and backup Internet connectivity.
* **2 Servers** for hosting network services.
* **Wireless Access Points** to provide wireless connectivity.
* **PCs, Laptops, Smartphones, and Printers** as end-user devices.

### Network Architecture and Operation

Each department is assigned a dedicated **VLAN**, allowing devices to be logically separated according to their department.

The Access Switches are connected to the Multilayer Switches using **802.1Q Trunk Links**, allowing multiple VLANs to traverse the network.

To prevent Layer 2 loops and provide a stable switching topology, **STP/PVST** is implemented. One Multilayer Switch operates as the **Root Primary**, while the other operates as the **Root Secondary**.

For gateway redundancy, **HSRP** is configured between the Multilayer Switches. This ensures that if one Multilayer Switch becomes unavailable, the other can continue providing the default gateway for the connected devices.

### Routing and Internet Connectivity

**OSPF** is used as the dynamic routing protocol between the Multilayer Switches and Core Routers.

Internet connectivity is implemented using:

* **NAT/PAT** to translate private internal IP addresses for Internet access.
* **ISP1** as the primary Internet connection.
* **ISP2** as the backup connection in case of ISP1 failure.

This design provides **redundancy and improved network availability**.

### Network Servers

The project includes two servers providing essential network services.

**Server 1:**

* **DNS**
* **Web**

**Server 2:**

* **Email**
* **FTP**

The DNS service allows users to access internal resources using domain names instead of IP addresses.

### Wireless Network

Wireless Access Points are deployed across the departments to provide wireless connectivity. The wireless network uses:

* **Separate SSIDs**
* **WPA2-PSK**
* **AES Encryption**
* **DHCP** for automatic IP address assignment

### Network Security

The project also implements several network security and device-hardening mechanisms, including:

* **Enable Secret**
* **Password Encryption**
* **SSH Version 2**
* **Telnet Disabled**
* **Local User Authentication**
* **RSA Keys**
* **Security Warning Banner**
* **Port Security**
* **Sticky MAC**
* **Shutdown of Unused Ports**

### Project Objective

The overall objective is to build a complete **Enterprise Network Infrastructure** that provides:

**Network Segmentation + Routing + Redundancy + Internet Connectivity + Network Services + Wireless Connectivity + Security**

Finally, the network is validated through a series of **Acceptance Tests** to ensure that the VLANs, Trunk Links, STP, HSRP, DHCP, OSPF, NAT, Internet connectivity, server services, wireless connectivity, and security mechanisms are functioning correctly.
