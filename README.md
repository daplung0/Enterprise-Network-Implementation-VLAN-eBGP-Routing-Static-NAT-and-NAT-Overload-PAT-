# Enterprise Network Implementation: VLAN, eBGP Routing, Static NAT, and NAT Overload (PAT)

**Project Type:** Network Engineering Lab  
**Platform:** Cisco Packet Tracer  
**Technology:** VLAN, Router-on-a-Stick, eBGP, Static NAT, NAT Overload (PAT), DHCP  
**Project Status:** Completed

---

# 1. Project Overview

## 1.1 Background

In a corporate network environment, some devices on the internal network require access to the external network, while certain internal servers need to be accessible from other networks.

However, private IP addresses on the internal network cannot be used directly as globally routable addresses. Therefore, a Network Address Translation (NAT) mechanism is required to translate private IP addresses into other IP addresses used on networks outside the internal network.

In this project, a corporate network simulation was built using Cisco Packet Tracer. The network consists of three VLAN segments connected via a Router-on-a-Stick, two autonomous systems that exchange routing information using the External Border Gateway Protocol (eBGP), and implementations of Static NAT and NAT Overload (PAT).

The two main requirements simulated are:

1. Providing external network access to several internal devices via NAT Overload (PAT).
2. Enabling access to the internal web server from the external network using Static NAT.

## 1.2 Project Objectives

The objectives of this network implementation are:

- To establish internal network segmentation using VLAN 10, VLAN 20, and VLAN 30.
- To implement inter-VLAN routing using a Router-on-a-Stick.
- To configure eBGP for routing exchange between AS1 and AS2.
- To implement NAT Overload (PAT) for devices on VLAN 10 and VLAN 20.
- To implement Static NAT for the web server on VLAN 30.
- To test the connectivity of internal devices to external servers.
- To test access to the internal web server via the IP address resulting from Static NAT translation. 

## 1.3 Project Scope

The scope of implementation includes:

| Component | Implementation |
|---|---|
| Network Segmentation | VLAN 10, VLAN 20, VLAN 30 |
| Inter-VLAN Routing | Router-on-a-Stick |
| Routing Protocol | eBGP |
| NAT | Static NAT dan NAT Overload (PAT) |
| IP Addressing | IPv4 |
| DHCP | Pool for the client network as configured |
| Server | External server and internal web server |
| Testing | ICMP Ping, NAT Translation, BGP Verification, Web Browser |

---

# 2. Network Fundamentals

## 2.1 Network Address Translation (NAT)

Network Address Translation is a mechanism for translating source or destination IP addresses when packets pass through a router.

NAT is generally used to connect internal networks that use private addresses to external networks.

In this project, NAT is used for two different purposes: NAT Overload and Static NAT.

## 2.2 NAT Overload (PAT)

Port Address Translation (PAT), also known as NAT Overload, allows multiple devices to use a single translated IP address by distinguishing communication sessions based on port numbers.

In this project, devices from VLAN 10 and VLAN 20 use R3’s external interface address, `10.20.20.2`, to access servers on the external network.

Thus, multiple internal devices can use a single translated address simultaneously.

## 2.3 Static NAT

Static NAT is a mechanism for translating a single private IP address to a single, fixed IP address.

In this project, the internal web server uses the following addresses:

- Inside Local: `192.168.30.2`
- Inside Global: `10.20.20.3`

With this configuration, devices on the external network can access the internal web server via the address `10.20.20.3`.

## 2.4 External Border Gateway Protocol (eBGP)

BGP is a routing protocol used to exchange routing information between autonomous systems.

In this simulation, eBGP is used to connect two autonomous systems:

- AS1: Google
- AS2: ISP

R1 and R2 exchange routing information so that Google’s server network and the ISP’s network can reach each other.

## 2.5 Virtual Local Area Network (VLAN)

VLAN are used to logically segment networks within a single switching infrastructure.

In this project, there are three VLANs:

- VLAN 10: the first client network.
- VLAN 20: the second client network.
- VLAN 30: the internal server network.

This segmentation helps organize the network based on the needs of devices and services.

---

# 3. Network Topology and IP Addressing

## 3.1 Network Topology

The network topology consists of three routers, one switch, two servers, and several PCs.

Device configuration:

| Device | Functions |
|---|---|
| R1 - Google | Router in AS1 |
| R2 - ISP | Router in AS2 |
| R3 - Customer | Router network internal and NAT |
| Switch 2960 | Connecting Devices in a VLAN |
| Server Google | Servers on an external network |
| SVR | Web server internal |
| PC0 & PC1 | Client VLAN 10 |
| PC2 & PC3 | Client VLAN 20 |

## 3.2 IP Addressing Table

### A. External Network

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Server Google | Fa0 | 172.16.10.2 | 255.255.255.0 |
| R1 | Fa0/0 | 172.16.10.1 | 255.255.255.0 |
| R1 | Fa0/1 | 10.10.10.1 | 255.255.255.0 |
| R2 | Fa0/0 | 10.10.10.2 | 255.255.255.0 |
| R2 | Fa0/1 | 10.20.20.1 | 255.255.255.0 |
| R3 | Fa0/0 | 10.20.20.2 | 255.255.255.0 |

### B. Internal Network

| VLAN | Network Address | Gateway | Subnet Mask |
|---|---|---|---|
| VLAN 10 | 192.168.10.0/24 | 192.168.10.1 | 255.255.255.0 |
| VLAN 20 | 192.168.20.0/24 | 192.168.20.1 | 255.255.255.0 |
| VLAN 30 | 192.168.30.0/24 | 192.168.30.1 | 255.255.255.0 |

### C. Server Addressing

| Server | IP Address | Gateway |
|---|---|---|
| Server Google | 172.16.10.2 | 172.16.10.1 |
| Internal Web Server (SVR) | 192.168.30.2 | 192.168.30.1 |

### D. NAT Address Mapping

| NAT Type | Inside Local | Inside Global |
|---|---|---|
| PAT | 192.168.10.0/24 | 10.20.20.2 (shared) |
| PAT | 192.168.20.0/24 | 10.20.20.2 (shared) |
| Static NAT | 192.168.30.2 | 10.20.20.3 |

**Important:** ``10.20.20.2` is the external interface address of R3 used for PAT, while `10.20.20.3` is the dedicated translation address for the internal web server.

---

# 4. Network Implementation and Configuration

## 4.1 Switch Configuration

First, create a VLAN on the switch and connect the devices to their respective VLANs.

### VLAN Creation

```cisco
enable
configure terminal

vlan 10
 name VLAN10-CLIENT

vlan 20
 name VLAN20-CLIENT

vlan 30
 name VLAN30-SERVER

end
```

### Access Port Configuration

The ports connected to the PC and the server must be configured as access ports according to their respective VLANs.

Configuration :

```cisco
configure terminal

interface range fa0/2-3
 switchport mode access
 switchport access vlan 10

interface range fa0/4-5
 switchport mode access
 switchport access vlan 20

interface fa0/6
 switchport mode access
 switchport access vlan 30

end
```

Adjust the port numbers to match the actual connections in the topology.

### Trunk Port Configuration

Port switch yang terhubung ke R3 digunakan untuk membawa trafik dari ketiga VLAN.

```cisco
configure terminal

interface fa0/1
 switchport mode trunk

end
```

Verification:

```cisco
show vlan brief
show interfaces trunk
```

## 4.2 R1 Configuration - Google (AS1)

R1 acts as a router for the Google network and is connected to R2 via the `10.10.10.0/24` network.

### Interface Configuration

```cisco
enable
configure terminal

interface fa0/0
 ip address 172.16.10.1 255.255.255.0
 no shutdown

interface fa0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown

end
```

### eBGP Configuration

R1 menggunakan Autonomous System Number 1.

```cisco
configure terminal

router bgp 1
 neighbor 10.10.10.2 remote-as 2

 network 10.10.10.0 mask 255.255.255.0
 network 172.16.10.0 mask 255.255.255.0

end
```

This configuration establishes an eBGP relationship with R2 in AS2 and advertises the networks connected to R1.

## 4.3 R2 Configuration - ISP (AS2)

R2 functions as an ISP router that connects the AS1 network to the customer's network.

### Interface Configuration

```cisco
enable
configure terminal

interface fa0/0
 ip address 10.10.10.2 255.255.255.0
 no shutdown

interface fa0/1
 ip address 10.20.20.1 255.255.255.0
 no shutdown

end
```

### eBGP Configuration

R2 menggunakan Autonomous System Number 2.

```cisco
configure terminal

router bgp 2
 neighbor 10.10.10.1 remote-as 1

 network 10.10.10.0 mask 255.255.255.0
 network 10.20.20.0 mask 255.255.255.0

end
```

This configuration allows R2 to exchange routing information with R1 via eBGP.

## 4.4 R3 Configuration - Customer Router

R3 serves as a gateway for the entire internal network. This router also handles inter-VLAN routing, DHCP as needed by clients, and NAT translation.

### External Interface Configuration

```cisco
enable
configure terminal

interface fa0/0
 ip address 10.20.20.2 255.255.255.0
 ip nat outside
 no shutdown
```

Interface `Fa0/0` It serves as the external interface because it is connected to the ISP network.

### Router-on-a-Stick Configuration

Interface `Fa0/1` digunakan sebagai trunk menuju switch.

```cisco
interface fa0/1
 no ip address
 no shutdown
```

Subinterface VLAN 10:

```cisco
interface fa0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
```

Subinterface VLAN 20:

```cisco
interface fa0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
```

Subinterface VLAN 30:

```cisco
interface fa0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
```

With this configuration, R3 can perform inter-VLAN routing and serve as the default gateway for the entire internal network.

### Default Route Configuration

Traffic from the internal network that will be forwarded to the ISP. That means R3 needs a default route.

```cisco
ip route 0.0.0.0 0.0.0.0 10.20.20.1
```

Next-hop `10.20.20.1` is the interface address of R2 that is directly connected to R3.

## 4.5 DHCP Configuration

DHCP is used to automatically assign IP configurations to client devices.

Configuration VLAN 10 dan VLAN 20:

```cisco
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1

ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
```

For the web server on VLAN 30, the IP address is statically configured:

- IP Address: `192.168.30.2`
- Subnet Mask: `255.255.255.0`
- Default Gateway: `192.168.30.1`


## 4.6 NAT Overload (PAT) Configuration

NAT Overload is used so that devices on VLAN 10 and VLAN 20 can access the external network using a single translated IP address—namely, the address of R3’s outside interface.

### Access Control List

```cisco
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
```

The ACL specifies the internal networks that are allowed to use NAT Overload.

### NAT Overload Configuration

```cisco
ip nat inside source list 1 interface fa0/0 overload
```

This command enables PAT translation using the IP address of interface `Fa0/0`.

As a result, devices on VLAN 10 and VLAN 20 use the address `10.20.20.2` as their translated address when accessing the external network.

## 4.7 Static NAT Configuration

Static NAT is used to map internal web servers so they can be accessed from an external network.

Information address:

- Inside Local: `192.168.30.2`
- Inside Global: `10.20.20.3`

Configuration:

```cisco
ip nat inside source static 192.168.30.2 10.20.20.3
```

With this configuration, requests to the address `10.20.20.3` will be routed to the internal web server at `192.168.30.2`.

The web server must have an active HTTP service and a correct default gateway.

---

# 5. Testing and Verification

The testing phase was conducted to ensure that the network configuration and NAT translation were functioning as intended.

## 5.1 BGP Neighbor Verification

Command used:

```cisco
show ip bgp summary
```

Test objectives:

- Verify that an eBGP connection between R1 and R2 is established.
- Verify that the BGP neighbor is in the “Established” state.
- Verify that prefix information can be exchanged.
  
**Result:** The BGP connection has been successfully tested based on the tests conducted in the simulation.

## 5.2 NAT Overload Testing - VLAN 10 & VLAN 20

Testing was run from PC0 and PC2, located on VLAN 10 and VLAN 20, to the Google server with the IP address `172.16.10.2`.

Command:

```text
ping 172.16.10.2
```

## 5.3 Static NAT Testing - Internal Web Server

Static NAT testing is performed by accessing the internal web server through a Google Server device.

Address entered in the web browser:

```text
http://10.20.20.3
```

That address is the Inside Global address of the internal web server.

The actual address of the web server:

```text
192.168.30.2
```

## 5.4 NAT Translation Verification

Check the NAT translation table on R3 using the following command:

```cisco
show ip nat translations
```

To view statistic NAT:

```cisco
show ip nat statistics
```

Expected output:

- Dynamic translation for VLAN 10 and VLAN 20.
- Static translation from `192.168.30.2` to `10.20.20.3`.

These commands are recommended as additional evidence to supplement the test results documentation.

## 5.6 Testing Summary

| No. | Test Scenario | Expected Result | Status |
|---|---|---|---|
| 1 | BGP Neighbor Verification | Neighbor Established | Passed |
| 2 | Ping VLAN 10 to Google Server | Reply berhasil | Passed |
| 3 | Ping VLAN 20 to Google Server | Reply berhasil | Passed |
| 4 | Static NAT Web Access | Web server appears in the browser | Passed |
| 5 | NAT Translation Table | PAT and Static NAT translations are visible | Passed |
| 6 | DHCP Verification | Client received the correct IP configuration | Passed |

---

# 6. Troubleshooting and Diagnostic Procedures

In network implementation, several issues may arise due to configuration errors involving interfaces, VLANs, routing, or NAT.

The following is a troubleshooting procedure that can be used in this project.

## 6.1 VLAN Connectivity Issues

Possible causes:
- The switch port has not been assigned to the correct VLAN.
- The VLAN has not been created.
- The trunk configuration is incorrect.

Verification commands:

```cisco
show vlan brief
show interfaces trunk
```

## 6.2 Inter-VLAN Routing Issues

Possible causes:
- The subinterface has not been configured.
- The dot1Q encapsulation is incorrect.
- The device gateway is incorrect.

Verification commands:

```cisco
show ip interface brief
show running-config
```

## 6.3 BGP Connectivity Issues

Possible causes:
- The IP addresses between routers do not match.
- The Autonomous System Number is incorrect.
- The neighbor address is incorrect.
- The interface is not active.

Verification commands:

```cisco
show ip bgp summary
show ip route
show ip bgp
```

## 6.4 NAT Overload Issues

Possible causes:
- The ACL does not cover the internal network.
- The “inside” and “outside” interfaces are swapped.
- The default route has not been configured.
- Routing to the external network is not available.

Verification commands:

```cisco
show ip nat translations
show ip nat statistics
show access-lists
show ip route
```

## 6.5 Static NAT Web Access Issues

Possible causes:
- Incorrect Static NAT mapping.
- The server has not enabled the HTTP service.
- The server’s default gateway is incorrect.
- VLAN 30 or the R3 subinterface has not been configured.
- A routing path from the external network to the Inside Global address is not available.

Verification commands:

```cisco
show ip nat translations
show ip nat statistics
show ip route
```

The procedures above serve as a diagnostic guide. Not all of these issues have been reported to have occurred during project implementation.

---

# 7. Project Results

Based on the implementation and testing that have been conducted, this project has successfully demonstrated several network functions.

## 7.1 Network Segmentation

The internal network has been successfully divided into three segments:

- VLAN 10 for clients.
- VLAN 20 for clients.
- VLAN 30 for internal servers.

The three segments are connected via the Router-on-a-Stick configuration on R3.

## 7.2 External Routing

R1 and R2 successfully established an eBGP connection for the exchange of routing information between AS1 and AS2.

## 7.3 NAT Overload

PC0 on VLAN 10 and PC2 on VLAN 20 successfully pinged the Google server `172.16.10.2`, with 0% packet loss as shown in the test screenshot.

This indicates that internal network connectivity to the external server is successful.

## 7.4 Static NAT

The internal web server with IP address `192.168.30.2` was successfully accessed via the translated address `10.20.20.3`.

Testing was performed using a web browser on a Google server, and the web server’s page was successfully displayed.

## 7.5 Overall Result

This project successfully simulated a corporate network with VLAN segmentation, inter-AS routing, and two NAT translation methods to meet the needs for internal network access and server service access.

---

# 8. Lessons Learned

Through this project, several technical insights were gained:

1. **Network Segmentation:** VLANs help separate networks based on device functions without requiring a different physical switch for each segment.

2. **Inter-VLAN Routing:** Router-on-a-Stick enables communication between VLANs through a single physical interface that uses multiple subinterfaces.

3. **Dynamic Routing:** eBGP enables the exchange of routing information between autonomous systems.

4. **NAT Overload:** PAT allows multiple internal devices to use a single translated IP address by utilizing port numbers.

5. **Static NAT:** Static NAT can be used to provide fixed address mappings for internal servers that need to be accessed from external networks.

6. **Network Verification:** Testing using ping, BGP summaries, NAT translation tables, and a web browser helps ensure the network functions as intended.

7. **Troubleshooting:** An understanding of interfaces, routing tables, ACLs, VLANs, and NAT is essential for analyzing network issues.

---

# 9. Project Resources

Dokumentasi dan file simulasi proyek dapat disimpan dalam repository GitHub dengan struktur berikut:

```text
enterprise-network-nat/
│
├── README.md
│
├── topology/
│   └── enterprise-network-topology.png
│
├── screenshots/
│   ├── network-topology.png
│   ├── ping-vlan10.png
│   ├── ping-vlan20.png
│   ├── static-nat-web-access.png
│   ├── bgp-verification.png
│   └── nat-translations.png
│
└── simulation/
    └── enterprise-network-nat.pkt
```

---

# 10. References

- Cisco Systems. Cisco IOS IP Addressing Services Configuration Guide.
- Cisco Systems. Cisco IOS NAT Configuration Guide.
- Cisco Systems. Border Gateway Protocol (BGP) Configuration Documentation.
- Cisco Systems. VLAN and Inter-VLAN Routing Documentation.
- Agunacourse. CCNA 200-301 Learning Materials.
- Cisco Packet Tracer. Network Simulation Environment.

---

