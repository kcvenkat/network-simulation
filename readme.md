Small Town Networking Project
=============================

This is a simulation of a small town's network designed to simulate realistic protocols, security configurations, and network setups on Cisco Packet Tracer.

## Table of Contents

-[Usage](#usage)
- [Planning](#planning)
- [Ethernet Networks](#ethernet-networks)
- [Wireless Networks](#wireless-networks)
- [VLAN Segmentation and IP Assignment](#vlan-segmentation-and-ip-assignment)
- [Routing and Subnetting](#routing-and-subnetting)
- [Server Configuration](#server-configuration)
- [Security Implementation](#security-implementation)
- [Risk Assessment](#risk-assessment)
- [Conclusion](#conclusion)

---
# Usage

This project is designed to be opened and interacted with through Cisco Packet Tracer. The included `.pkt` file contains the complete simulated town network topology, including all routers, switches, VLAN configurations, ACLs, NAT/PAT rules, DMZ architecture, DHCP pools, DNS services, and ASA firewall configurations described in this documentation.

## Requirements

- Cisco Packet Tracer
- Recommended version: Packet Tracer 8.x or newer

## Running the Simulation

1. Download the `.pkt` file from this repository.
2. Open the file using Cisco Packet Tracer.
3. Allow the topology and services to fully initialize.
4. Test connectivity between networks using:
   - `ping`
   - web browser HTTP requests
   - DNS lookups
   - inter-VLAN routing tests
5. Inspect router, switch, and ASA configurations through the CLI to review:
   - VLAN assignments
   - Router-on-a-Stick subinterfaces
   - NAT/PAT rules
   - ACL filtering
   - DHCP pools
   - DMZ segmentation
   - ASA firewall behavior

## Included Features

- VLAN segmentation
- Router-on-a-Stick inter-VLAN routing
- DHCP and DNS services
- Wired and wireless LANs
- NAT/PAT implementation
- Static NAT for public services
- Extended ACL perimeter filtering
- DMZ architecture
- Cisco ASA 5506-X firewall configuration
- Star topology backbone simulation
- NIST SP 800-30 risk assessment
- NIST SP 800-53 security control remediation
---

# Planning

The planning for this was relatively simple. To supplement my learning of internet protocols, I wanted to have a plethora of different networks that could act as a simulation of how a town could look while being able to experiment with different network setups and configurations.

The initial setup was as follows:

- A law firm with three LANs with wired ethernet configurations.
- A hospital with four LANs, three with cabled ethernet configurations and one wireless guest network.
- A school with four LANs, three with wireless configurations and one with cabled ethernet configuration.
- A tech company with a DMZ hosting a public web and DNS server, an ASA firewall, and an internal employee wireless network.
- A few wireless home networks with one LAN each.

Each router is connected to a central switch simulating the Internet backbone using a star topology. (Note: the original design used a ring topology, but this was changed to eliminate single points of failure and more accurately reflect how the real Internet routes traffic.)

---

# Routing and Subnetting

Most networks were configured using the "router-on-a-stick" method because each ISR4331 router didn't have enough ports to support multiple connections. The router would be connected to a 2960 IOS15 switch through one router port, and the switch would be connected to multiple devices to account for the lack of router ports.

## Ethernet Networks

The steps to form an Ethernet connection with static IP address assignment are as follows:

1. Connect the router to the switch through a GigabitEthernet or FastEthernet port using a straight-through copper cable.
2. Connect the switch to the device through a FastEthernet port using another straight-through copper cable.
3. Open the port connecting the router to the switch on the router's CLI and assign it an IP address:

```bash
Router> enable
Router# configure terminal
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# no shutdown
Router(config-if)# ip address 192.168.2.1 255.255.255.0
```

4. Assign the PC's interface an IP address from its configuration tab (for example: IPv4 address 192.168.2.2, subnet mask 255.255.255.0).
5. Ping the router from the PC's command prompt to verify connectivity:

```bash
ping 192.168.2.1
```

If the PC receives ping replies from the router, the Ethernet connection is successfully established and working as expected.

## Wireless Networks

Wireless networks were assigned a separate VLAN and broadcasted the network through a WAP (the AccessPointPT device on Cisco Packet Tracer).

The steps to form a wireless connection with DHCP are as follows:

1. Connect the router to the switch through a GigabitEthernet or FastEthernet port using a straight-through copper cable.
2. Connect the switch to the WAP through an Ethernet port using another straight-through copper cable.
3. Open the port connecting the router to the switch on the router's CLI and assign it an IP address:

```bash
Router> enable
Router# configure terminal
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# no shutdown
Router(config-if)# ip address 192.168.5.1 255.255.255.0
```

4. Enable DHCP on the router for the wireless network:

```bash
Router(config)# ip dhcp pool GUEST_WIFI
Router(dhcp-config)# network 192.168.5.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.5.1
Router(dhcp-config)# dns-server 79.52.164.250
```

5. Ping the router from the PC's command prompt to verify connectivity:

```bash
ping 192.168.5.1
```

If the PC receives ping replies from the router, the wireless connection is successfully established and working as expected.

## VLAN Segmentation and IP Assignment

VLANs (Virtual Local Area Networks) allow network administrators to logically separate traffic on a single physical switch. This improves security, reduces congestion, and makes network management easier. Each VLAN acts as an independent subnet, preventing broadcast traffic from crossing between networks such as administrative LANs and guest Wi-Fi.

The following steps describe how to create VLANs, assign ports, and configure router subinterfaces for inter-VLAN communication.

1. **Create VLANs on the switch**

```bash
Switch> enable
Switch# configure terminal
Switch(config)# vlan 2
Switch(config-vlan)# name HR
Switch(config)# exit
Switch(config)# vlan 1000
Switch(config-vlan)# name DEAD
```

(Note: VLAN 1000 is designated as the "dead VLAN" — all unused switch ports are assigned here to prevent unauthorized access from untagged connections.)

2. **Assign switch ports to VLANs**

```bash
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 2

Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 1000
Switch(config-if)# switchport trunk allowed vlan 2,3,4,5
```

3. **Configure router subinterfaces for inter-VLAN routing (Router-on-a-Stick)**

```bash
Router(config)# interface GigabitEthernet0/0/0.2
Router(config-subif)# encapsulation dot1Q 2
Router(config-subif)# ip address 192.168.2.1 255.255.255.0
Router(config-subif)# exit
```

The dot1Q VLAN ID must match the VLAN number on the switch exactly. The subinterface number is typically set to match the VLAN ID for clarity and convention.

4. **Test VLAN connectivity**

From a device on the 192.168.3.0/24 network, ping a device on the 192.168.2.0/24 network:

```bash
ping 192.168.2.2
```

If devices within each VLAN receive IP addresses and successful ping responses, the VLAN configuration and routing setup are functioning correctly.

---

# Routing and Subnetting

The network operates on a star topology, using a central switch to forward information between institution routers. This central switch simulates the Internet backbone — each institution connects to it with a public IP on the 79.52.164.0/24 subnet. (Note: the original design called for a ring topology with direct router-to-router connections. This was changed because a ring topology creates a single point of failure and does not accurately reflect how the real Internet routes traffic.)

## Internal Configuration

Each institution's router only needs a single default route pointing to the central switch gateway:

```bash
Router(config)# ip route 0.0.0.0 0.0.0.0 79.52.164.1
```

## Central Switch Configuration

The central switch uses static routes to reach each institution's internal subnets. For the Law Firm (WAN: 79.52.164.8) hosting networks 192.168.2-5.0/24:

```bash
Router(config)# ip route 192.168.2.0 255.255.255.0 79.52.164.8
Router(config)# ip route 192.168.3.0 255.255.255.0 79.52.164.8
Router(config)# ip route 192.168.4.0 255.255.255.0 79.52.164.8
Router(config)# ip route 192.168.5.0 255.255.255.0 79.52.164.8
```

This is repeated for every institution. (Note: after NAT/PAT was implemented, these static routes to private subnets became unnecessary for inter-institution traffic. Private IPs are now hidden behind each institution's public WAN IP, accurately simulating real Internet behavior where private addresses are not globally routable.)

---

# Server Configuration

There are two essential services required for a device to connect to a website: HTTP and DNS.

HTTP (Hypertext Transfer Protocol) handles the actual connection to the web server and loads the contents of the requested website. DNS (Domain Name System) maps human-readable domain names to their corresponding IP addresses, eliminating the need to memorize numerical addresses.

## Configure HTTP

The HTTP service is enabled on the Server-PT device in Cisco Packet Tracer. The contents of index.html were edited to produce the page you are currently reading. The server's private IP is 192.168.42.250, hosted in the Tech Company's DMZ subnet (192.168.42.0/24).

## Configure DNS

After enabling the DNS service, readme.com and www.readme.com are mapped to the server's private IP 192.168.42.250. A static NAT entry on the Tech Company router maps the public IP 79.52.164.250 to the server's private IP, making it reachable from any institution on the network.

```bash
ip nat inside source static 192.168.42.250 79.52.164.250
```

Each device's DNS server is configured to point to 79.52.164.250. For DHCP pools:

```bash
Router(dhcp-config)# dns-server 79.52.164.250
```

Once configured, every device on every network can access readme.com through the web browser. (Note: the original design placed the server directly on the edge router's network with no isolation. It was later moved into the Tech Company's DMZ subnet for proper network segmentation.)

---

# Security Implementation

Security was implemented across all five institutions following a defense-in-depth approach. Each layer builds on the previous: NAT/PAT hides the internal network, perimeter ACLs filter traffic at the boundary, and the Tech Company additionally implements a DMZ architecture and ASA firewall for its public-facing services.

## NAT / PAT

PAT (Port Address Translation) is configured on every institution's router. All internal private IP addresses are translated to the router's single public WAN IP before leaving the network. This hides the internal network topology from the backbone and prevents private IPs from being directly targeted by external actors.

```bash
! Define which subnets are eligible for translation
access-list 1 permit 192.168.2.0 0.0.0.255
access-list 1 permit 192.168.3.0 0.0.0.255
access-list 1 permit 192.168.4.0 0.0.0.255
access-list 1 permit 192.168.5.0 0.0.0.255

! Mark inside and outside interfaces
interface GigabitEthernet0/0/0
 ip nat inside
interface GigabitEthernet0/0/1
 ip nat outside

! Enable PAT using the WAN interface IP
ip nat inside source list 1 interface GigabitEthernet0/0/1 overload
```

The Tech Company additionally uses a static NAT entry to expose the web and DNS server publicly:

```bash
ip nat inside source static 192.168.42.250 79.52.164.250
```

## Perimeter ACLs

Two extended ACLs are applied to each institution's router. BLOCK_INBOUND is applied inbound on the WAN interface to deny unsolicited external traffic. ALLOW_OUTBOUND is applied inbound on the internal interface to restrict outbound protocols to only what is necessary for network operation.

```bash
! BLOCK_INBOUND -- applied inbound on WAN interface (G0/0/1)
ip access-list extended BLOCK_INBOUND
 permit icmp any any echo-reply
 permit tcp any any established
 permit udp host 79.52.164.250 eq 53 any
 deny ip any any

! ALLOW_OUTBOUND -- applied inbound on internal interface (G0/0/0)
ip access-list extended ALLOW_OUTBOUND
 permit tcp any any eq 80
 permit tcp any any eq 443
 permit tcp any any eq 53
 permit udp any any eq 53
 permit icmp any any
 deny ip any any

interface GigabitEthernet0/0/1
 ip access-group BLOCK_INBOUND in
interface GigabitEthernet0/0/0
 ip access-group ALLOW_OUTBOUND in
```

The deny ip any any at the end of each ACL enforces deny-by-default. Outbound traffic is restricted to HTTP (80), HTTPS (443), DNS (53), and ICMP. All other protocols and ports are blocked in both directions.

The Tech Company's BLOCK_INBOUND additionally permits inbound HTTP, HTTPS, and DNS traffic specifically destined for the public server IP:

```bash
ip access-list extended BLOCK_INBOUND
 permit tcp any host 79.52.164.250 eq 80
 permit tcp any host 79.52.164.250 eq 443
 permit icmp any any echo-reply
 permit udp host 79.52.164.250 eq 53 any
 deny ip any any
```

## DMZ Architecture (Tech Company)

The Tech Company implements a three-zone architecture separating the internet, the DMZ, and the internal employee network. The web and DNS server sits in the DMZ subnet (192.168.42.0/24, VLAN 3) connected to the DMZ switch. The ASA firewall separates the DMZ switch from the internal employee wireless network (192.168.40.0/24, VLAN 2).

Traffic rules enforced by router ACLs and the ASA:

- External traffic can reach the DMZ server on ports 80, 443, and 53 only.
- External traffic cannot reach the internal employee network.
- Internal employees can reach the DMZ server and the internet.
- The DMZ server cannot initiate connections to the internal employee network.

## ASA 5506-X Firewall

A Cisco ASA 5506-X firewall is placed between the DMZ switch and the employee access point. The outside interface (facing the DMZ) is set to security-level 50 and the inside interface (facing the employee network) is set to security-level 100. Higher security levels can initiate connections to lower security levels by default, but not the reverse. The ASA also handles DHCP for employee laptops and PAT for their outbound traffic.

```bash
interface GigabitEthernet1/1
 nameif outside
 security-level 50
 ip address 192.168.42.2 255.255.255.0
 no shutdown

interface GigabitEthernet1/2
 nameif inside
 security-level 100
 ip address 192.168.40.1 255.255.255.0
 no shutdown

! Default route toward the router
route outside 0.0.0.0 0.0.0.0 192.168.42.1 1

! PAT for employee outbound traffic
object network INSIDE
 subnet 192.168.40.0 255.255.255.0
 nat (inside,outside) dynamic interface

! DHCP for employee laptops
dhcpd address 192.168.40.2-192.168.40.254 inside
dhcpd dns 79.52.164.250
dhcpd option 3 ip 192.168.40.1
dhcpd enable inside
```

# Risk Assessment

A Risk Assessment Report (RAR) was conducted prior to security implementation following NIST SP 800-30 guidelines. The IS was categorized as SC = (Confidentiality, Low), (Integrity, Low), (Availability, Low). Four threat events were identified: unauthorized network reconnaissance, unauthorized access, denial of service attacks, and establishment of command and control chains. All four were rated Moderate risk or below given existing VLAN segmentation and WPA2-PSK wireless security. However, significant non-compliance was found against NIST SP 800-53 controls SC-7, SC-5, and SI-4(4) due to the absence of perimeter filtering, NAT, and traffic monitoring.

The following controls were implemented to remediate the identified findings:

**SC-7 (Boundary Protection)** — Extended ACLs deployed on the WAN interface of every institution's router. The Tech Company additionally implements a Cisco ASA 5506-X firewall at the boundary between the DMZ and the internal employee network.

**SC-7(5) (Deny by Default)** — All ACLs terminate with an explicit deny ip any any rule. No traffic is permitted unless explicitly allowed.

**SC-7(8) (Controlled Managed Interface)** — All traffic passes through a single router perimeter with ACLs applied. The Tech Company's ASA provides a second controlled boundary for internal employee traffic.

**SC-5 (Denial of Service Protection)** — ALLOW_OUTBOUND ACLs restrict outbound traffic to ports 80, 443, 53, and ICMP only, significantly reducing the attack surface. NAT/PAT hides all internal endpoints from direct targeting.

**NAT/PAT** — PAT configured on all five institution routers. Static NAT configured on the Tech Company router for the public-facing web and DNS server at 79.52.164.250.

**DMZ Architecture** — Web and DNS server isolated in a dedicated DMZ subnet. External access restricted to ports 80, 443, and 53 only. Server-initiated connections to the internal network are blocked.

**SI-4(4) (Residual Gap)** — Inbound and outbound traffic is restricted to necessary services only via ACLs. However, active traffic monitoring and detection are not fully implementable in Cisco Packet Tracer. In an enterprise environment, a stateful firewall, NGFW, or IDS/IPS would be required to fully satisfy this control.

---

# Conclusion

The purpose of this project was to design and simulate a small town's interconnected network infrastructure using Cisco Packet Tracer. What began as a basic connectivity exercise evolved into a full network security audit simulation, implementing NAT/PAT, perimeter ACLs, VLAN segmentation, DMZ architecture, and an ASA hardware firewall across five simulated institutions.

Each organization was configured with distinct Ethernet and wireless LANs segmented by VLANs. A star topology through a central backbone switch enables full inter-institution communication. Security controls assessed against the NIST SP 800-53 framework were implemented to remediate all identified risks, with the exception of SI-4(4) traffic monitoring due to Cisco Packet Tracer simulation environment limitations.

This project reinforced understanding of IP addressing, DHCP, VLAN management, inter-VLAN routing, NAT/PAT, ACL-based filtering, DMZ design, and hardware firewall configuration, as well as formal risk assessment methodology following NIST SP 800-30.
