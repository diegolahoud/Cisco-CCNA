# Lab 04: PortChannel (EtherChannel)

## Overview
The purpose of this lab is to demonstrate how multiple physical switch interfaces are bundled together into a single logical link called a **PortChannel** (EtherChannel). PortChannels provide link redundancy, increased bandwidth, and load balancing across active links while preventing Spanning Tree Protocol (STP) blocking.

This lab compares two distinct EtherChannel deployment models across two campuses:
- **Campus A (Layer 3 PortChannel):** Routed EtherChannel utilizing `no switchport` across `PortChannel 4` with IP routing between subnets (`192.168.0.0/24`, `192.168.1.0/24`, `192.168.2.0/24`).
- **Campus B (Layer 2 PortChannel):** Switching EtherChannel utilizing 802.1Q trunking across `PortChannel 7` for VLAN traffic (`10.1.0.0/24`).

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

| Device | Interface | Remote Device | Remote Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PCB** | Gig0 | MLSB-1 | Gig1/0/24 | 10.1.0.10 | 255.255.255.0 | 10.1.0.1 |
| **MLSB-1** | Gig1/0/5 - 8 | MLSB-2 | Gig1/0/5 - 8 | PortChannel 7 | N/A | N/A |
| **MLSB-2** | Gig1/0/24 | ServerB | Gig0 | N/A (Access) | N/A | N/A |
| **ServerB** | Gig0 | MLSB-2 | Gig1/0/24 | 10.1.0.20 | 255.255.255.0 | 10.1.0.1 |

---

## Configuration Steps

### Campus A Configuration (Layer 3 PortChannel)

#### Step 1: Configure Layer 3 EtherChannel on `MLSA-1` & `MLSA-2`
Convert interface range `Gig1/0/1 - 4` to Layer 3 (`no switchport`), assign to `channel-group 4 mode active` (LACP), configure the routed IP on `interface Port-channel 4`, and enable `ip routing`.

```cisco
! TODO: Insert MLSA-1 Layer 3 PortChannel commands here