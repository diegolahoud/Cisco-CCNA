# Lab 14-01: Test knowdledge between lab01 and lab 05

## Overview
This first mega lab will have the instructions in the .PDF file. The first part, it will be a Spanning Tree Protocol calculation. The answers will be on a different .PDF named "STP-Answers". The second part of the lab will contain a topology named "Lab14-01-Original.pkt" made by the professor to test our knowledge. He provided the topology, and with the instructiosn on the .PDF file we need to compelted. The completed part will be on the "Lab14.01-Answer.pkt"

==========

# Lab 14-01: Comprehensive Assessment (Labs 01–05)

## Overview
This comprehensive assessment integrates and evaluates core networking concepts covered across **Lab 01 through Lab 05**, including VLAN segmentation, Inter-VLAN routing, Spanning Tree Protocol (STP), EtherChannel link aggregation, and centralized DHCPv4 with DHCP Relay.

The assessment consists of two core components: a theoretical STP topology calculation and a hands-on multi-device implementation within Cisco Packet Tracer.

---

## Repository Artifacts & Lab Files

| File / Artifact | Description |
| :--- | :--- |
| **`Lab14-01-Instructions.pdf`** | Primary assessment instructions, requirements, and topology specifications. |
| **`STP-Answers.pdf`** | Answer key and detailed path calculations for the theoretical STP exercise. |
| **`Lab14-01-Original.pkt`** | Baseline, unconfigured Packet Tracer topology provided for the assessment. |
| **`Lab14-01-Answer.pkt`** | Fully configured and verified solution topology in Packet Tracer. |

---

## Assessment Structure & Scope

### Part 1: Theoretical STP Calculation
- **Focus:** Spanning Tree Protocol logic, path cost evaluations, Root Bridge selection, and port role determination (Root Ports, Designated Ports, and Blocked/Alternate Ports).
- **Deliverable:** Theoretical analysis verified against `STP-Answers.pdf`.

### Part 2: Practical Network Implementation
- **Focus:** Full end-to-end configuration of the `Lab14-01-Original.pkt` topology according to the requirements in `Lab14-01-Instructions.pdf`.
- **Core Topics Evaluated:**
  - **VLAN & Trunking:** VLAN creation, access interface assignments, and 802.1Q trunk links.
  - **Inter-VLAN Routing:** Router-on-a-Stick (ROAS) or Multilayer Switch SVIs.
  - **EtherChannel Aggregation:** Layer 2 / Layer 3 PortChannel bundling using PAgP/LACP.
  - **DHCPv4 Services:** Centralized DHCP pool configuration and `ip helper-address` relay setup.
- **Deliverable:** Complete operational network saved in `Lab14-01-Answer.pkt`.

---

## Verification & Success Criteria

1. **STP Logic:** Validate manual calculations against `STP-Answers.pdf`.
2. **Dynamic Addressing:** Confirm end hosts successfully acquire dynamic IP addresses from the centralized DHCP server across routed VLAN boundaries.
3. **Link Aggregation:** Verify PortChannel status on switches using `show etherchannel summary`.
4. **End-to-End Connectivity:** Execute ICMP ping tests across all subnets to confirm complete network reachability.



-------

## Lab14-01-Answer.pkt step-by-step configuration

1. VLANs creation, and port assigments on the Access Layer Switches: 

! Some VLANs are missing, and interfaces are not configured.

! AS1

```cisco
AS1(config)#int range fa0/1 - 6
AS1(config-if-range)#switchport mode access
AS1(config-if-range)#switchport access vlan 10
AS1(config-if-range)#int range fa0/7 - 12
AS1(config-if-range)#switchport mode access 
AS1(config-if-range)#switchport access vlan 13
```

! AS2
```cisco
AS2(config)#int range fa0/1 - 6
AS2(config-if-range)#switchport mode access 
AS2(config-if-range)#switchport access vlan 10
AS2(config-if-range)#vlan 10
AS2(config-vlan)#name ENG
AS2(config-vlan)#int range fa0/7 - 12
AS2(config-if-range)#switchport mode access 
AS2(config-if-range)#switchport access vlan 13
% Access VLAN does not exist. Creating vlan 13
AS2(config-if-range)#vlan 13
AS2(config-vlan)#name HR
```

! AS3
```cisco
AS3(config)#int range fa0/1 - 6
AS3(config-if-range)#switchport mode access 
AS3(config-if-range)#switchport access vlan 10
AS3(config-if-range)#vlan 10
AS3(config-vlan)#name ENG
AS3(config-vlan)#int range fa0/7 - 12
AS3(config-if-range)#switchport mode access 
AS3(config-if-range)#switchport access vlan 13
AS3(config-if-range)#vlan 13
AS3(config-vlan)#name HR
```

2) Trunk ports creation from Access Layer Switches to Distribution Layer Switches

! AS1
```cisco
AS1(config)#int range fa0/19 - 20
AS1(config-if-range)#switchport mode trunk 
```

! AS2
```cisco
AS2(config-vlan)#int range fa0/21 -22
AS2(config-if-range)#switchport mode trunk 
```

! AS3
```cisco
AS3(config)#int range fa0/23 - 24
AS3(config-if-range)#switchport mode trunk 
```

3) Add VLANs and SVI on the Distrubution Layer switches. 

! DS1
```cisco
DS1(config)#int vlan 10
DS1(config-if)#ip add 10.0.0.1 255.0.0.0
DS1(config-if)#vlan 10
DS1(config-vlan)#name ENG
```

! DS2
```cisco
DS2(config)#int vlan 13
DS2(config-if)#ip add 13.0.0.2 255.0.0.0
DS2(config-if)#vlan 13
DS2(config-vlan)#name HR
```

! DS3

```cisco
DS3(config)#int range fa0/13 - 15
DS3(config-if-range)#switchport mode access 
DS3(config-if-range)#switchport access vlan 14
DS3(config-if-range)#int vlan 14
DS3(config-if)#ip add 14.0.0.1 255.0.0.0
DS3(config-if)#vlan 14
DS3(config-vlan)#name 14
DS3(config-vlan)#
```

4) Configure the routing capabilities on the Layer 3 switches. They will be DS1, DS2, Core1, and DS3. Run the same command to each of them in global configuration mode

```cisco
hostname(config)#ip routing
```

5) Add L3 Fast Etherchannel between Core1 and distributions swtiches (DS1, DS2, and DS3) with PAgP negotiation protocol. 

! Link Between DS1 & Core1. This is Po1 in the 11.0.0.0/8 network

```cisco
DS1(config-vlan)#int range fa0/1 - 2
DS1(config-if-range)#shutdown 
DS1(config-if-range)#no switchport 
DS1(config-if-range)#channel-group 1 mode desirable 
DS1(config-if-range)#int po1
DS1(config-if)#ip add 11.0.0.2 255.0.0.0
DS1(config-if)#no shutdown
```
&
```cisco
Core1(config)#int range fa0/1 - 2
Core1(config-if-range)#no shutdown 
Core1(config-if-range)#no switchport 
Core1(config-if-range)#channel-group 1 mode desirable 
Core1(config-if-range)#int po1
Core1(config-if)#ip add 11.0.0.1 255.0.0.0
Core1(config-if)#no shutdown 
```

! link Between DS2 & Core1. This is Po2 in the 12.0.0.0/8 network.

```cisco
DS2(config)#int range fa0/3 - 4
DS2(config-if-range)#shutdown 
DS2(config-if-range)#no switchport 
DS2(config-if-range)#channel-group 2 mode desirable 
DS2(config-if-range)#int po2
DS2(config-if)#ip add 12.0.0.2 255.0.0.0
DS2(config-if)#no shutdown
```
&&

```cisco
Core1(config-if)#int range fa0/3 - 4
Core1(config-if-range)#shutdown 
Core1(config-if-range)#no switchport 
Core1(config-if-range)#channel-group 2 mode desirable 
Core1(config-if-range)#int po2
Core1(config-if)#ip add 12.0.0.1 255.0.0.0
Core1(config-if)#no sh
Core1(config-if)#no shutdown 
```

! Link between Core1 & DS3. This is Po3 in the 15.0.0.0/8 network. 

```cisco
Core1(config-if)#int range gig0/1 - 2
Core1(config-if-range)#shutdown 
Core1(config-if-range)#no switchport 
Core1(config-if-range)#channel-group 3 mode desirable 
Core1(config-if-range)#int po3
Core1(config-if)#ip add 15.0.0.1 255.0.0.0
Core1(config-if)# no shutdown
```

```cisco
DS3(config)#int range gig0/1 - 2
DS3(config-if-range)#shutdown 
DS3(config-if-range)#no switchport 
DS3(config-if-range)#channel-group 3 mode desirable 
DS3(config-if-range)#int po3
DS3(config-if)#ip add 15.0.0.2 255.0.0.0
DS3(config-if)#no shutdown 
```

6) Configure the STP root brige for vlan 10 and vlan 13

! DS1

```cisco
DS1(config)#spanning-tree vlan 10 root primary
DS1(config)#spanning-tree vlan 13 root secondary 
```

! DS2

```cisco
DS2(config)#spanning-tree vlan 13 root primary 
DS2(config)#spanning-tree vlan 10 root secondary 
```

7) Configure DS1, DS2, and DS3 so that the DHCP server can deliver the correct IP addresses to all PCs

! DS1

```cisco
DS1(config)#int vlan 10
DS1(config-if)#ip helper-address 14.0.0.5
```

! DS2

```cisco
DS2(config)#int vlan 13
DS2(config-if)#ip helper-address 14.0.0.5
```

! DS3

```cisco
DS3(config)#int vlan 14
DS3(config-if)#ip helper-address 14.0.0.5
```

8) Follow the instructions in the .PDF instructions to verify the lab is completed, and succesfull. 

! Verify EtherChannel status

```cisco
Core1# show etherchannel summary
DS1# show etherchannel summary
```

! Verify Spanning Tree Root Bridge 

```cisco
DS1# show spanning-tree vlan 10
DS2# show spanning-tree vlan 13
```