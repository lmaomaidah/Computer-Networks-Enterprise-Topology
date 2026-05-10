# Enterprise Multi-Protocol Network Topology
### Cisco Packet Tracer · FAST-NUCES · Computer Networks · Spring 2026

> A fully operational multi-zone enterprise network implementing OSPF, EIGRP, and RIP with inter-protocol redistribution, centralised DHCP, VLSM subnetting, NAT, ACLs, and email services — configured entirely in Cisco Packet Tracer.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Topology Architecture](#topology-architecture)
- [Routing Zones](#routing-zones)
- [Network Addressing (VLSM)](#network-addressing-vlsm)
- [Key Configurations](#key-configurations)
  - [Route Redistribution](#route-redistribution)
  - [Centralised DHCP](#centralised-dhcp)
  - [NAT on Router20](#nat-on-router20)
  - [Access Control Lists](#access-control-lists)
  - [Email Services](#email-services)
- [Device Inventory](#device-inventory)
- [How to Open](#how-to-open)
- [Verification Commands](#verification-commands)
- [Assignment Context](#assignment-context)

---

## Project Overview

This project simulates a multi-site enterprise network spread across four routing domains interconnected through **route redistribution**. Each domain runs a distinct dynamic routing protocol, and all end-user hosts receive addresses dynamically from a **single centralised DHCP server** located in the RIP zone, served across routing boundaries using `ip helper-address`.

The topology demonstrates:
- **Three routing protocols** (OSPF, EIGRP, RIP) coexisting in a single network
- **VLSM** applied to 11 end-user networks plus all router-to-router links (/30)
- **Redistribution** at four border routers bridging the protocol domains
- **NAT** translating a private IP to a public address at the network edge
- **Extended ACLs** restricting specific hosts from reaching the Web Server
- **Mail server** with per-host email accounts across OSPF Area 1

---

## Topology Architecture

    +-------------------------+     +--------------------------------------+     +-------------------------+
    |       OSPF Area 1       |     |             EIGRP AS 5               |     |       OSPF Area 2       |
    |                         |     |                                      |     |                         |
    |  Net A  55.0.0.0/15     |     |  Net D  55.18.128.0/18 (smallest)   |     |  Net G  55.18.0.0/17    |
    |  Net B  55.16.0.0/15    +-----+  Net E  55.12.0.0/15                +-----+  Net H  55.10.0.0/15    |
    |  Net C  55.8.0.0/15     |     |  Net F  55.4.0.0/15                 |     |  Net I  55.2.0.0/15     |
    |  [Mail Server]          |     |                                      |     |  [Web Server]           |
    +-------------------------+     +-----------------+--------------------+     +-------------------------+
       Redistribution @ Router5                       |
                                                      | Redistribution @ Router13
                                                      |
                                          +-----------+----------+
                                          |         RIP          |
                                          |                      |
                                          |  Net J  55.14.0.0/15 |
                                          |  Net K  55.6.0.0/15  |
                                          |  [DHCP Server] <--------- serves ALL zones via ip helper-address
                                          |  [NAT @ Router20]    |
                                          +----------------------+

---

## Routing Zones

### OSPF Area 1 — Top Left

| Router | Role |
|--------|------|
| Router0 | Internal OSPF Area 1 |
| Router1 | Internal + connects to EIGRP boundary |
| Router3 | Internal OSPF Area 1 |
| Router4 | Internal OSPF Area 1 |
| Router5 | **Border router — OSPF Area 1 to EIGRP redistribution** |

Networks: A, B, C — contains the **Mail Server** (Network B)

---

### EIGRP Autonomous System 5 — Centre

| Router | Role |
|--------|------|
| Router6 | Internal EIGRP |
| Router8 | Internal EIGRP |
| Router9 | Internal EIGRP |
| Router10 | Internal EIGRP |
| Router11 | Internal EIGRP |
| Router12 | **Border router — EIGRP to OSPF Area 2** |
| Router14 | **Border router — EIGRP to OSPF Area 2** |

Networks: D, E, F — central backbone connecting all three other zones

---

### OSPF Area 2 — Top Right

| Router | Role |
|--------|------|
| Router_15 | Internal OSPF Area 2 |
| Router_16 | Internal OSPF Area 2 |
| Router14 | Border (shared with EIGRP zone) |

Networks: G, H, I — contains the **Web Server** (Network H)

---

### RIP — Bottom

| Router | Role |
|--------|------|
| Router13 | **Border router — RIP to EIGRP redistribution** |
| Router15 | Internal RIP |
| Router16 | Internal RIP |
| Router17 | Internal RIP |
| Router18 | Internal RIP |
| Router20 | **NAT router** (Network J) |

Networks: J, K — contains **DHCPServer2** (Network K) serving the entire topology

---

## Network Addressing (VLSM)

All subnets carved from the assigned IP space using Variable Length Subnet Masking.

| Network | Zone | Subnet | Mask | Gateway | DHCP Start | Max Hosts | Devices |
|---------|------|--------|------|---------|------------|-----------|---------|
| A | OSPF 1 | 55.0.0.0 | /15 | 55.0.0.1 | 55.0.0.2 | 131,070 | Switch1, Laptop2, Laptop3 |
| B | OSPF 1 | 55.16.0.0 | /15 | 55.16.0.1 | 55.16.0.2 | 131,070 | Switch2, MailServer, PC0, PC1 |
| C | OSPF 1 | 55.8.0.0 | /15 | 55.8.0.1 | 55.8.0.2 | 131,070 | Switch0, Laptop0, Laptop1 |
| D | EIGRP | 55.18.128.0 | /18 | 55.18.128.1 | 55.18.128.2 | 16,382 | Switch3, PC2, Laptop4 |
| E | EIGRP | 55.12.0.0 | /15 | 55.12.0.1 | 55.12.0.2 | 131,070 | Switch9, PC5, Laptop6 |
| F | EIGRP | 55.4.0.0 | /15 | 55.4.0.1 | 55.4.0.2 | 131,070 | Switch4, PC3, Laptop5 |
| G | OSPF 2 | 55.18.0.0 | /17 | 55.18.0.1 | 55.18.0.2 | 32,766 | Switch6, PC6, PC7 |
| H | OSPF 2 | 55.10.0.0 | /15 | 55.10.0.1 | static | 131,070 | Switch8, Web Server |
| I | OSPF 2 | 55.2.0.0 | /15 | 55.2.0.1 | 55.2.0.2 | 131,070 | Switch7, PC8, PC9 |
| J | RIP | 55.14.0.0 | /15 | 55.14.0.1 | 55.14.0.2 | 131,070 | Switch9, PC10, PC11 |
| K | RIP | 55.6.0.0 | /15 | 55.6.0.1 | 55.6.0.2 | 131,070 | Switch10, PC12, PC13, DHCPServer2 |

> Router-to-router links all use /30 subnets (4 addresses: network, router A, router B, broadcast).

---

## Key Configurations

### Route Redistribution

**OSPF Area 1 to EIGRP — Router5**

    router eigrp 5
     redistribute ospf 1 metric 10000 100 255 1 1500

    router ospf 1
     redistribute eigrp 5 subnets metric-type 2

**EIGRP to OSPF Area 2 — Router12 and Router14**

    router eigrp 5
     redistribute ospf 1 metric 10000 100 255 1 1500

    router ospf 1
     redistribute eigrp 5 subnets metric-type 2

**EIGRP to RIP — Router13**

    router eigrp 5
     redistribute rip metric 10000 100 255 1 1500

    router rip
     version 2
     redistribute eigrp 5 metric 5

---

### Centralised DHCP

DHCPServer2 in Network K holds address pools for every end-user network. All routers use `ip helper-address` to forward DHCP broadcasts across routing boundaries.

**Pool example — Network A**

    ip dhcp excluded-address 55.0.0.1
    ip dhcp pool NetworkA
     network 55.0.0.0 255.254.0.0
     default-router 55.0.0.1

**Relay on the router interface facing each LAN**

    interface [LAN interface]
     ip helper-address [DHCPServer2 IP]

> The Web Server (Network H) is statically addressed and excluded from DHCP.

---

### NAT on Router20

Static NAT translates the assigned private IP to a public address for Network J.

    ip nat inside source static [private IP] [public IP]

    interface [LAN interface toward Network J]
     ip nat inside

    interface [WAN interface]
     ip nat outside

Verify with: `show ip nat translations`

---

### Access Control Lists

Two rules block access to the Web Server (Network H — 55.10.0.0/15).

**Rule 1 — one specific PC in Network A is denied**

    access-list 101 deny ip host [PC-A IP] 55.10.0.0 0.1.255.255
    access-list 101 permit ip any any

**Rule 2 — all hosts in Network D are denied**

    access-list 102 deny ip 55.18.128.0 0.0.63.255 55.10.0.0 0.1.255.255
    access-list 102 permit ip any any

Both ACLs applied inbound on the router interfaces facing their respective networks.

Verify with: `show access-lists` — test by pinging the Web Server from a blocked vs allowed host.

---

### Email Services

Mail Server in Network B runs SMTP (outbound) and POP3 (inbound). All hosts in OSPF Area 1 (Networks A, B, C) have accounts.

**Server — Services > Email**
- Set domain name
- Create a user account for each host (e.g. laptop2@domain, pc0@domain)

**Each PC — Desktop > Email**
- Incoming mail server: Mail Server IP, POP3
- Outgoing mail server: Mail Server IP, SMTP

Test by sending an email from PC0 to PC1 and verifying receipt.

---

## Device Inventory

| Type | Count | Models |
|------|-------|--------|
| Routers | 19 | Cisco 2811 — Router0 through Router18, Router20 |
| Switches | 11 | Cisco 2960-24TT — Switch0 through Switch10 |
| PCs | 14 | PC0 through PC13 |
| Laptops | 7 | Laptop0 through Laptop6 |
| Servers | 3 | MailServer, Web Server, DHCPServer2 |

---

## How to Open

1. Install Cisco Packet Tracer (version 8.x or later recommended)
2. Clone or download this repository
3. Open `project.pkt` in Packet Tracer
4. Wait for all devices to finish booting (link lights turn green or amber)
5. Run the verification commands below to confirm everything is working

---

## Verification Commands

    show ip route                   ! Check for O, D, R route entries
    show ip ospf neighbor           ! OSPF adjacencies — state should be FULL
    show ip eigrp neighbors         ! EIGRP adjacencies
    show ip rip database            ! RIP learned routes
    show ip dhcp binding            ! Active leases from the centralised server
    show ip nat translations        ! NAT table — run on Router20
    show access-lists               ! ACL hit counters
    ping [destination IP]           ! End-to-end connectivity
    traceroute [destination IP]     ! Path across redistribution boundaries

**Expected route codes by zone**

| Zone | Codes you should see |
|------|----------------------|
| OSPF Area 1 routers | O (local OSPF), D EX (from EIGRP) |
| EIGRP routers | D (EIGRP), O E2 (from OSPF), R (from RIP) |
| OSPF Area 2 routers | O (local OSPF), D EX (from EIGRP) |
| RIP routers | R (RIP), D EX (from EIGRP) |

---

## Assignment Context

**Course:** Computer Networks — FAST-NUCES Islamabad, Spring 2026
**Student:** Maidah Binte Junaid (i242116)

Each student was assigned a unique starting IP and unique host-per-subnet counts. This topology uses my assigned address space (55.x.x.x). Subnet sizes, DHCP pools, and the NAT private IP will differ from other students' submissions.

The `.pkt` file is fully self-contained — all router configs, server settings, and end-device configurations are saved inside it. No additional files are needed.
