# Lab 04: PortChannel (EtherChannel)

## Overview
The purpose of this lab is to demonstrate how multiple physical switch interfaces are bundled together into a single logical link called a **PortChannel** (EtherChannel). PortChannels provide link redundancy, increased bandwidth, and load balancing across active links while preventing Spanning Tree Protocol (STP) blocking.

This lab compares two distinct EtherChannel deployment models across two campuses:
- **Campus A (Layer 3 PortChannel):** Routed EtherChannel utilizing `no switchport` across `PortChannel 4` with IP routing between subnets (`192.168.0.0/24`, `192.168.1.0/24`, `192.168.2.0/24`).
- **Campus B (Layer 2 PortChannel):** Switching EtherChannel utilizing 802.1Q trunking across `PortChannel 7` for VLAN traffic (`10.1.0.0/24`).Both campuses will be using pAgP as it is the Cisco propietary negotiation protocol

> **Key Rule for EtherChannel Configuration:** To form an operational EtherChannel, all bundled interfaces on both switches must match in:
> - **Speed and Duplex settings**
> - **Switchport Mode** (Access or Trunk)
> - **VLAN Membership** (if access) or **Native VLAN & Allowed VLANs** (if trunk)

---

## Equipment
- **1 x Multilayer Switch:** `MLSA-1` (Cisco Catalyst 3650)
- **1 x Multilayer Switch:** `MLSA-2` (Cisco Catalyst 3650)
- **1 x Multilayer Switch:** `MLSB-1` (Cisco Catalyst 3650)
- **1 x Multilayer Switch:** `MLSB-2` (Cisco Catalyst 3650)
- **1 x PC:** `PCA` (Equipped with `PT-HOST-NM-1CGE` module for Gigabit Ethernet)
- **1 x Server:** `ServerA` (Equipped with `PT-HOST-NM-1CGE` module for Gigabit Ethernet)
- **1 x PC:** `PCB` (Equipped with `PT-HOST-NM-1CGE` module for Gigabit Ethernet)
- **1 x Server:** `ServerB` (Equipped with `PT-HOST-NM-1CGE` module for Gigabit Ethernet)

> **Note:** On Cisco 3650 switches in Packet Tracer, remember to drag and drop a power supply module into the switch rear slot to power on the device.

---

## Topology & Addressing Table

### Campus A: Layer 3 PortChannel

| Device | Interface | Remote Device | Remote Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PCA** | Gig0 | MLSA-1 | Gig1/0/24 | 192.168.0.10 | 255.255.255.0 | 192.168.0.1 |
| **MLSA-1** | Gig1/0/1 - 4 | MLSA-2 | Gig1/0/1 - 4 | PortChannel 4 | N/A | N/A |
| **MLSA-2** | Gig1/0/24 | ServerA | Gig0 | N/A (Access) | N/A | N/A |
| **ServerA** | Gig0 | MLSA-2 | Gig1/0/24 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |

### Campus B: Layer 2 PortChannel

| Device | Interface | Remote Device | Remote Interface | IP Address | Subnet Mask | Default Gateway | VLAN
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PCB** | Gig0 | MLSB-1 | Gig1/0/24 | 10.1.0.10 | 255.0.0.0 | Not Needed as it is Layer2/on the same subnet | VLAN 4 (HR)
| **MLSB-1** | Gig1/0/5 - 8 | MLSB-2 | Gig1/0/5 - 8 | PortChannel 7 | N/A | N/A |
| **MLSB-2** | Gig1/0/24 | ServerB | Gig0 | N/A (Access) | N/A | N/A |
| **ServerB** | Gig0 | MLSB-2 | Gig1/0/24 | 10.1.0.20 | 255.0.0.0 | Not Needed as it is Layer2/on the same subnet | VLAN 4 (HR)

---

## Configuration Steps

### Campus A Configuration (Layer 3 PortChannel)

#### Step 1: Configure Layer 3 EtherChannel on `MLSA-1` & `MLSA-2`
Convert interface range `Gig1/0/1 - 4` to Layer 3 (`no switchport`), assign to `channel-group 4 mode active` (LACP), configure the routed IP on `interface Port-channel 4`, and enable `ip routing`.

```cisco
! TODO: Insert MLSA-1 Layer 3 PortChannel commands here
```

### Campus B Configuration (Layer 2 PortChannel)

**Notes:** Before starting, review that all the port settings are the same. For this lab, VLAN 4 (HR) was already created in MLSB-1 and MLSB-2. Also, verified the speed, duplex, allowed vlans, etc. 

```cisco
MSLB-1(config)#int range gig1/0/5 -8
MSLB-1(config)#shutdown 
```
**Note:** We turn off the links so we do not get an error while we are configuring the ports. Make sure you turn off the ports from gig1/0/5 to 8 in MSLB-2 as well

'''cisco
MSLB-1(config-if-range)#channel-group 7 mode desirable
MSLB-1(config-if-range)#int po7
MSLB-1(config-if)#switchport mode trunk
MSLB-1(config-if)# no shutdown
'''

**Note:** The Etherchannel is not created yet as we need to configure the same settings on the switch ports, on MLSB-2. Remember the links should be shutdown from the previous step. Moreover, any modification that you do in the interface PortChannel itself, it will modified the settings in all the physical ports members of the EtherChannel

'''cisco
MSLB-1(config) int range gig1/0/5 - 8
MSLB-1(config-if-range)#channel-group 7 mode desirable
MSLB-1(config-if-range)#int po7
MSLB-1(config-if)#switchport mode trunk
MSLB-1(config-if)# no shutdown
MSLB-1(config-if)#exit
MSLB-1(config)#do sh wri mem
MSLB-1(config)#do wri mem
MSLB-1(config)# do etherchannel sum
'''

**Note1:** You will see that the links are now all green, this is because the EtherChannel is considered as one virtual port, and STP do not need to make the blocking port in each individual physical port, they are all acting like one. You you can ping from the PCB to ServerB, and the trunking EtherChannel will send the traffic. 

**Note2:** At the verification step, you should see the number of the Etherchannel (in this case, Po7), the physical ports assigned to that EtherChannel, and that it is In Use (U) and Layer2(S)