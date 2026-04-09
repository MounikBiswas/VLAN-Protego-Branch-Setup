# 🌐 Enterprise Network Configuration: VLANs, ROAS, DHCP & Security ACLs

> A comprehensive Cisco Packet Tracer laboratory demonstrating inter-VLAN routing, dynamic IP addressing, and network traffic security using Extended Access Control Lists (ACLs).

## ✨ Project Overview

This project simulates a corporate network environment with three distinct departments: **ADMIN**, **SALES**, and **IT**. It showcases core networking principles, including segmenting broadcast domains, enabling communication via Router-on-a-Stick (ROAS), automating IP assignments, and securing departmental traffic.

**Key Implementations:**
* **VLAN Segmentation:** Layer 2 logical separation for ADMIN (10), SALES (20), and IT (30).
* **Trunking & Access Ports:** 802.1Q encapsulation for inter-switch and router communication.
* **Inter-VLAN Routing (ROAS):** Sub-interface configuration on the core router to allow routing between isolated VLANs.
* **DHCP Services:** Automated IP address allocation with excluded static ranges for network infrastructure.
* **Network Security (Extended ACL):** Traffic filtering to explicitly deny the SALES department from accessing the IT department while permitting all other necessary traffic.

---

## 🛠️ Network Topology & Addressing

| VLAN ID | Department Name | Network Subnet | Default Gateway |
| :--- | :--- | :--- | :--- |
| **10** | ADMIN | `192.168.10.0 /24` | `192.168.10.1` |
| **20** | SALES | `192.168.20.0 /24` | `192.168.20.1` |
| **30** | IT | `192.168.30.0 /24` | `192.168.30.1` |

*(Note: IP addresses `.1` through `.10` are excluded from the DHCP pool for static infrastructure assignments).*

---

## 💻 Device Configurations

The following configurations were applied to the network devices. 

### 1. Core Switch (Sw0) Configuration
*Configures local VLANs, assigns access ports, and establishes trunk links.*

```text
enable
configure terminal

! Create and Name VLANs
vlan 10
 name ADMIN
vlan 20
 name SALES
vlan 30
 name IT
exit

! Assign Access Ports to VLANs
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/3
 switchport mode access
 switchport access vlan 30
exit

! Configure Trunk Ports
interface g0/1
 switchport mode trunk
interface g0/2
 switchport mode trunk
 no shutdown
end
write memory

```
**2. Distribution Switch (Sw1) Configuration
Mirrors the core switch VLAN database and assigns respective local access ports.**

```
enable
configure terminal

! Create and Name VLANs
vlan 10
 name ADMIN
vlan 20
 name SALES
vlan 30
 name IT
exit

! Assign Access Ports to VLANs
interface fa0/1
 switchport mode access
 switchport access vlan 10
interface fa0/2
 switchport mode access
 switchport access vlan 20
interface fa0/3
 switchport mode access
 switchport access vlan 30
exit

! Configure Trunk Port to Core
interface g0/1
 switchport mode trunk
end
write memory
```
**3. Router 0 Configuration (ROAS, DHCP, and ACL)
Handles inter-VLAN routing, acts as the DHCP server, and enforces security policies.**
```
enable
configure terminal

! Activate the Physical Interface
interface g0/0
 no shutdown
exit

! Configure Sub-interfaces for Inter-VLAN Routing (ROAS)
interface g0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
interface g0/0.30
 encapsulation dot1q 30
 ip address 192.168.30.1 255.255.255.0
exit

! Configure DHCP Pools and Exclusions
ip dhcp pool ADMIN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
ip dhcp pool SALES
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
ip dhcp pool IT
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
exit

ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10

! Configure and Apply Extended ACL 100
! Rule: Deny SALES (VLAN 20) from reaching IT (VLAN 30), permit everything else.
access-list 100 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 100 permit ip any any

interface g0/0.20
 ip access-group 100 in
end
write memory

```
**✅ Verification & Testing
To ensure the network is functioning and the security policies are actively filtering traffic, the following ICMP (Ping) tests were executed from client machines.

Test 1: Permitted Traffic (ADMIN to IT)
Testing connectivity from the ADMIN network to an IT client (192.168.30.11). Result: SUCCESS.**
```
C:\>ping 192.168.30.11

Pinging 192.168.30.11 with 32 bytes of data:

Request timed out.
Reply from 192.168.30.11: bytes=32 time=1ms TTL=127
Reply from 192.168.30.11: bytes=32 time<1ms TTL=127
Reply from 192.168.30.11: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.11:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)

(Note: The initial packet loss is expected behavior due to the ARP broadcast process).
```
**Test 2: Blocked Traffic (SALES to IT)
Testing connectivity from the SALES network to an IT client (192.168.30.11) to verify the Access Control List. Result: BLOCKED.**
```
C:\>ping 192.168.30.11

Pinging 192.168.30.11 with 32 bytes of data:

Reply from 192.168.20.1: Destination host unreachable.
Reply from 192.168.20.1: Destination host unreachable.
Reply from 192.168.20.1: Destination host unreachable.
Reply from 192.168.20.1: Destination host unreachable.

Ping statistics for 192.168.30.11:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```
**The router's gateway interface (192.168.20.1) successfully intercepted and dropped the traffic as dictated by ACL 100.**
