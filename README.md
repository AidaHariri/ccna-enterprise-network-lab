# Enterprise Multi-Site Network Simulation — CCNA Lab

A Cisco Packet Tracer project simulating a multi-site enterprise network (Head Office, four branch offices, and a server room), built to demonstrate hands-on CCNA-level networking skills — from Layer 2 switching fundamentals up through dynamic routing, WAN connectivity, and network security.

## 📁 Topology Overview

| Site | Devices | Description |
|---|---|---|
| **Head Office** | 3 switches (triangle topology), Core Block router | Two access switches (4 PCs each) uplink to a distribution switch, which connects to the Core Block router. The Core Block router also links to the Server Room. |
| **Server Room** | Connected via Core Block router | Hosts the DHCP server, reached across a different broadcast domain from the clients (hence DHCP relay). |
| **Branch 1** | 1 Multilayer Switch (MLS) + 2 access switches (triangle topology) | MLS sits at the top of the triangle; two access switches with their own PCs connect to it. Handles inter-VLAN routing at Layer 3 (SVI). |
| **Branch 2** | 2 switches (dual parallel links) + 2 routers | Redundant switch-to-switch links; each switch connects to its own router. Uses HSRP for gateway redundancy. |
| **Branch 3** | 1 switch + 2 PCs + 1 router | Standalone branch. |
| **Branch 4** | 1 switch + 2 PCs + 1 router | Standalone branch. |

## 🛠️ Technologies & Protocols Implemented

The network was built up in layers, starting from Layer 2 physical redundancy and working up to full Layer 3 routing and WAN connectivity:

### Layer 2 — Switching Foundation
- **EtherChannel (PAgP)** — bundled physical links between switches, negotiated as `desirable`/`desirable` on both ends for load balancing and redundancy.
- **Port Security** — configured on access ports to detect/block unauthorized devices based on MAC address violations.
- **Trunking (802.1Q)** — enabled trunk links to carry traffic for multiple VLANs between switches.
- **VTP (VLAN Trunking Protocol)** — configured VTP server mode with a defined VTP domain (moving off the default null domain) in the Head Office and Branch 1 sites, so VLANs created on the server are automatically advertised to client switches.
- **VLANs** — two VLANs defined per branch, each named to reflect its client group.
- **IP Addressing** — assigned IP ranges to clients so that devices in the same VLAN (across one or two switches) can successfully ping each other.

### Spanning Tree & Redundancy
- **STP (PVST+ / Rapid-PVST+)** — deployed per branch depending on design needs.
- **PortFast** — manually enabled on PVST+ access ports for immediate forwarding on link-up (Rapid-PVST+ enables this automatically on access ports).
- **BPDU Guard** — enabled on access ports so that receiving an unexpected BPDU immediately shuts the port down.
- **Root Bridge Tuning** — assigned root primary and root secondary roles per VLAN on the same switch, so each interface takes on a different STP role depending on VLAN.

### Layer 3 — Routing
- **Inter-VLAN Routing** — implemented two ways:
  - **Router-on-a-Stick**: sub-interfaces on the router, one per VLAN.
  - **Switch Virtual Interfaces (SVI)**: `interface vlan X` on the Layer 3 (Multilayer) switch.
- **HSRP (Hot Standby Router Protocol)** — configured in Branch 2: a virtual IP shared by both routers per VLAN, used as the client gateway. Verified failover by shutting down the active gateway — connectivity is preserved as if only one router had failed.
- **OSPF** — deployed for internal dynamic routing.
- **EIGRP** — deployed alongside OSPF in a different part of the network.
- **Route Redistribution** — mutual redistribution between OSPF and EIGRP so routes learned by one protocol are visible to the other.
- **Default Routing** — a default route from each branch pointed toward the Core Block router (whose relevant sub-interfaces sit behind Frame Relay and are NAT'd).

### DHCP
- **DHCP Server** — configured to lease addresses to clients across the network.
- **DHCP Relay (`ip helper-address`)** — since the DHCP server and its clients sit in different broadcast domains, the Core Block router was configured as a relay agent, converting broadcast DHCP requests to unicast and forwarding them to the server.

### WAN & Internet Edge
- **BGP** — configured manually with defined neighbor relationships and AS numbers; used the `network` command to advertise the desired networks toward the internet edge.
- **NAT (PAT)** — translated internal client source IPs to the router's `outside` interface global address, with inside/outside roles explicitly assigned per interface.
- **GRE Tunnels** — established between routers that each have one leg facing the internet, enabling routing over the WAN as if directly connected.

### Security
- **VTY Security** — configured username/password authentication on VTY lines to secure remote (Telnet/SSH) access to network devices.

## 📝 Notes
- Before removing/adding a serial module on a router, always run `copy running-config startup-config` first — otherwise the configuration is lost on power-down.

## 📂 File
`CCNA_lab final.pkt` — open with [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (recommended version 8.x or later).

## 🎯 Skills Demonstrated
Layer 2/3 switching, VLAN design, STP tuning, EtherChannel, port security, inter-VLAN routing, First Hop Redundancy Protocols (HSRP), dynamic routing (OSPF, EIGRP, BGP), route redistribution, DHCP relay, NAT/PAT, GRE tunneling, and device hardening — core competencies aligned with the CCNA certification.
