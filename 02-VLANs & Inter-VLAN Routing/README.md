# Lab 02: VLANs and Inter-VLAN Routing

## Overview
This lab demonstrates two fundamental methods for implementing Inter-VLAN routing across enterprise networks:
- **Campus A:** **Router-on-a-Stick (ROAS)** using a Layer 2 switch connected to a router via an 802.1Q trunk link.
- **Campus B:** **Multilayer Switch (Layer 3) Routing** utilizing Switch Virtual Interfaces (SVIs), routed switchports (`no switchport`), and default static routing across Cisco Catalyst 3560 switches.

---

## Topology & Addressing Table

### Campus A: Router-on-a-Stick (Layer 2)

| Device | Interface / Subinterface | VLAN | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **RS-CAMPUSA-1** | Gi0/0.10 | 10 (HR) | 192.168.0.1 | 255.255.255.0 | N/A |
| **RS-CAMPUSA-1** | Gi0/0.20 | 20 (Finance) | 192.168.1.1 | 255.255.255.0 | N/A |
| **RS-CAMPUSA-1** | Gi0/0.99 | 99 (IT) | 192.168.2.1 | 255.255.255.0 | N/A |
| **PC0** | Fa0 | 10 (HR) | 192.168.0.10 | 255.255.255.0 | 192.168.0.1 |
| **PC1** | Fa0 | 10 (HR) | 192.168.0.11 | 255.255.255.0 | 192.168.0.1 |
| **PC2** | Fa0 | 20 (Finance) | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| **Server0** | Fa0 | 99 (IT) | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |

### Campus B: Multilayer Switching (Layer 3)

| Device | Interface / SVI | VLAN | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **MLS-CAMPUSB-1** | SVI Vlan 30 | 30 (Managers) | 10.0.0.1 | 255.0.0.0 | N/A |
| **MLS-CAMPUSB-1** | Gig0/1 (Routed Link) | N/A | 13.0.0.1 | 255.0.0.0 | N/A |
| **MLS-CAMPUSB-2** | SVI Vlan 40 | 40 (Executives)| 11.0.0.1 | 255.0.0.0 | N/A |
| **MLS-CAMPUSB-2** | SVI Vlan 55 | 55 (Admins) | 12.0.0.1 | 255.0.0.0 | N/A |
| **MLS-CAMPUSB-2** | Gig0/1 (Routed Link) | N/A | 13.0.0.2 | 255.0.0.0 | N/A |
| **PC3** | Fa0 | 30 (Managers) | 10.0.0.10 | 255.0.0.0 | 10.0.0.1 |
| **PC4** | Fa0 | 30 (Managers) | 10.0.0.11 | 255.0.0.0 | 10.0.0.1 |
| **PC5** | Fa0 | 40 (Executives)| 11.0.0.10 | 255.0.0.0 | 11.0.0.1 |
| **PC6** | Fa0 | 55 (Admins) | 12.0.0.10 | 255.0.0.0 | 12.0.0.1 |

---

## Configuration Steps

### Campus A Configuration (Router-on-a-Stick)

#### Step 1: Configure Layer 2 Switch (`SWI-CAMPUSA-1`)
Create VLANs, assign access ports, and configure the trunk port to `RS-CAMPUSA-1`:

```cisco
! Create VLAN 10 & Assign Access Ports
SWI-CAMPUSA-1> enable
SWI-CAMPUSA-1# configure terminal
SWI-CAMPUSA-1(config)# interface range FastEthernet0/1 - 2
SWI-CAMPUSA-1(config-if-range)# switchport mode access
SWI-CAMPUSA-1(config-if-range)# switchport access vlan 10
SWI-CAMPUSA-1(config-if-range)# exit
SWI-CAMPUSA-1(config)# vlan 10
SWI-CAMPUSA-1(config-vlan)# name HR
SWI-CAMPUSA-1(config-vlan)# exit
```
> **Note:** The `interface range` command modifies multiple switch ports simultaneously (in this case, `Fa0/1` and `Fa0/2`).

```cisco
! Create VLAN 20 & Assign Access Port
SWI-CAMPUSA-1(config)# interface FastEthernet0/3
SWI-CAMPUSA-1(config-if)# switchport mode access
SWI-CAMPUSA-1(config-if)# switchport access vlan 20
SWI-CAMPUSA-1(config-if)# exit
SWI-CAMPUSA-1(config)# vlan 20
SWI-CAMPUSA-1(config-vlan)# name Finance
SWI-CAMPUSA-1(config-vlan)# exit

! Create VLAN 99 & Assign Access Port
SWI-CAMPUSA-1(config)# interface FastEthernet0/4
SWI-CAMPUSA-1(config-if)# switchport mode access
SWI-CAMPUSA-1(config-if)# switchport access vlan 99
SWI-CAMPUSA-1(config-if)# exit
SWI-CAMPUSA-1(config)# vlan 99
SWI-CAMPUSA-1(config-vlan)# name IT
SWI-CAMPUSA-1(config-vlan)# exit

! Configure Trunk Port to RS-CAMPUSA-1
SWI-CAMPUSA-1(config)# interface FastEthernet0/5
SWI-CAMPUSA-1(config-if)# switchport mode trunk
SWI-CAMPUSA-1(config-if)# end
SWI-CAMPUSA-1# write memory
```

#### Step 2: Configure Router Subinterfaces (`RS-CAMPUSA-1`)
Configure 802.1Q subinterfaces on the router physical interface to perform Inter-VLAN routing:

```cisco
! Activate physical interface connected to SWI-CAMPUSA-1
RS-CAMPUSA-1> enable
RS-CAMPUSA-1# configure terminal
RS-CAMPUSA-1(config)# interface GigabitEthernet0/0
RS-CAMPUSA-1(config-if)# no shutdown

! Configure 802.1Q encapsulation on subinterfaces
RS-CAMPUSA-1(config)# interface GigabitEthernet0/0.10
RS-CAMPUSA-1(config-subif)# encapsulation dot1q 10
RS-CAMPUSA-1(config-subif)# ip address 192.168.0.1 255.255.255.0

RS-CAMPUSA-1(config)# interface GigabitEthernet0/0.20
RS-CAMPUSA-1(config-subif)# encapsulation dot1q 20
RS-CAMPUSA-1(config-subif)# ip address 192.168.1.1 255.255.255.0

RS-CAMPUSA-1(config)# interface GigabitEthernet0/0.99
RS-CAMPUSA-1(config-subif)# encapsulation dot1q 99
RS-CAMPUSA-1(config-subif)# ip address 192.168.2.1 255.255.255.0
RS-CAMPUSA-1(config-subif)# end
RS-CAMPUSA-1# write memory
```

> **Note:** Ensure each PC/Server has its Default Gateway pointed to its respective subinterface IP (e.g., `192.168.0.1` for PC0) so end devices can route traffic across VLANs.

---

### Campus B Configuration (Layer 3 Switch SVIs)

#### Step 3: Configure Primary Multilayer Switch (`MLS-CAMPUSB-1`)
Enable IP routing, create VLAN 30, configure access ports, set up SVI 30 as the gateway, and create the Layer 3 routed link to `MLS-CAMPUSB-2`:

```cisco
! Configure VLAN 30 Access Ports & Gateway SVI
MLS-CAMPUSB-1> enable
MLS-CAMPUSB-1# configure terminal
MLS-CAMPUSB-1(config)# interface FastEthernet0/1
MLS-CAMPUSB-1(config-if)# switchport mode access
MLS-CAMPUSB-1(config-if)# switchport access vlan 30
MLS-CAMPUSB-1(config-if)# exit

MLS-CAMPUSB-1(config)# interface FastEthernet0/2
MLS-CAMPUSB-1(config-if)# switchport mode access
MLS-CAMPUSB-1(config-if)# switchport access vlan 30
MLS-CAMPUSB-1(config-if)# exit

MLS-CAMPUSB-1(config)# vlan 30
MLS-CAMPUSB-1(config-vlan)# name managers
MLS-CAMPUSB-1(config-vlan)# exit

MLS-CAMPUSB-1(config)# interface vlan 30
MLS-CAMPUSB-1(config-if)# ip address 10.0.0.1 255.0.0.0
MLS-CAMPUSB-1(config-if)# no shutdown
MLS-CAMPUSB-1(config-if)# exit

! Enable Hardware IP Routing
MLS-CAMPUSB-1(config)# ip routing

! Configure Layer 3 Routed Link to MLS-CAMPUSB-2
MLS-CAMPUSB-1(config)# interface GigabitEthernet0/1
MLS-CAMPUSB-1(config-if)# no switchport
MLS-CAMPUSB-1(config-if)# ip address 13.0.0.1 255.0.0.0
MLS-CAMPUSB-1(config-if)# no shutdown
MLS-CAMPUSB-1(config-if)# exit

! Configure Static Route to Remote Subnets on MLS-CAMPUSB-2
MLS-CAMPUSB-1(config)# ip route 0.0.0.0 0.0.0.0 13.0.0.2
MLS-CAMPUSB-1(config)# end
MLS-CAMPUSB-1# write memory
```

> **Note:** The `interface vlan` command creates the Switched Virtual Interface (SVI), which serves as the Default Gateway IP for the subnet.

#### Step 4: Configure Secondary Multilayer Switch (`MLS-CAMPUSB-2`)
Configure access ports for VLANs 40 and 55, create SVIs 40 and 55 as gateways, enable IP routing, and configure the Layer 3 routed link back to `MLS-CAMPUSB-1`:

```cisco
! Configure VLAN 40 Access Port & Gateway SVI
MLS-CAMPUSB-2> enable
MLS-CAMPUSB-2# configure terminal
MLS-CAMPUSB-2(config)# interface FastEthernet0/1
MLS-CAMPUSB-2(config-if)# switchport mode access
MLS-CAMPUSB-2(config-if)# switchport access vlan 40
MLS-CAMPUSB-2(config-if)# exit

MLS-CAMPUSB-2(config)# vlan 40
MLS-CAMPUSB-2(config-vlan)# name executives
MLS-CAMPUSB-2(config-vlan)# exit

MLS-CAMPUSB-2(config)# interface vlan 40
MLS-CAMPUSB-2(config-if)# ip address 11.0.0.1 255.0.0.0
MLS-CAMPUSB-2(config-if)# no shutdown
MLS-CAMPUSB-2(config-if)# exit

! Configure VLAN 55 Access Port & Gateway SVI
MLS-CAMPUSB-2(config)# interface FastEthernet0/2
MLS-CAMPUSB-2(config-if)# switchport mode access
MLS-CAMPUSB-2(config-if)# switchport access vlan 55
MLS-CAMPUSB-2(config-if)# exit

MLS-CAMPUSB-2(config)# vlan 55
MLS-CAMPUSB-2(config-vlan)# name Admins
MLS-CAMPUSB-2(config-vlan)# exit

MLS-CAMPUSB-2(config)# interface vlan 55
MLS-CAMPUSB-2(config-if)# ip address 12.0.0.1 255.0.0.0
MLS-CAMPUSB-2(config-if)# no shutdown
MLS-CAMPUSB-2(config-if)# exit

! Enable Hardware IP Routing
MLS-CAMPUSB-2(config)# ip routing

! Configure Layer 3 Routed Link to MLS-CAMPUSB-1
MLS-CAMPUSB-2(config)# interface GigabitEthernet0/1
MLS-CAMPUSB-2(config-if)# no switchport
MLS-CAMPUSB-2(config-if)# ip address 13.0.0.2 255.0.0.0
MLS-CAMPUSB-2(config-if)# no shutdown
MLS-CAMPUSB-2(config-if)# exit

! Configure Static Route to Remote Subnet on MLS-CAMPUSB-1
MLS-CAMPUSB-2(config)# ip route 0.0.0.0 0.0.0.0 13.0.0.1
MLS-CAMPUSB-2(config)# end
MLS-CAMPUSB-2# write memory
```

> **Note:** The VLANs on `MLS-CAMPUSB-2` communicate across the Layer 3 point-to-point link using default static routes pointing to the next-hop IP of `MLS-CAMPUSB-1` (`13.0.0.1`) and vice versa.

---

## Verification & Testing

1. **Verify VLAN Assignment:**
   Run `show vlan brief` on all switches to verify that ports are correctly mapped to their local VLANs.
2. **Verify Layer 3 & Trunk Interfaces:**
   - On **Campus A (`SWI-CAMPUSA-1`)**: Run `show interfaces trunk` to verify the 802.1Q trunk on `Fa0/5`.
   - On **Campus B (`MLS-CAMPUSB-1` & `MLS-CAMPUSB-2`)**: Run `show ip interface brief` to verify SVIs and `Gig0/1` routed ports are `up/up`.
3. **Verify Routing Tables:**
   Run `show ip route` on `RS-CAMPUSA-1`, `MLS-CAMPUSB-1`, and `MLS-CAMPUSB-2` to confirm connected subnets and static routes (`S*`) are present.
4. **Test End-to-End Inter-VLAN Reachability:**
   - **Campus A:** From PC0 (`192.168.0.10`), ping PC2 (`192.168.1.10`) and Server0 (`192.168.2.10`).
   - **Campus B:** From PC3 (`10.0.0.10`), ping PC5 (`11.0.0.10`) and PC6 (`12.0.0.10`).