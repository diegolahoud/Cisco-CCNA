# Lab 02: VLANs and Inter-VLAN Routing

## Overview
This lab demonstrates two fundamental methods for implementing Inter-VLAN routing across enterprise networks:
- **Campus A:** **Router-on-a-Stick (ROAS)** using a Layer 2 switch connected to a router via an 802.1Q trunk link.
- **Campus B:** **Multilayer Switch (Layer 3) Routing** utilizing Switch Virtual Interfaces (SVIs) and hardware IP routing (`ip routing`) across Cisco Catalyst 3560 switches.

---

## Topology & Addressing Table

### Campus A: Router-on-a-Stick (Layer 2)

| Device | Interface / Subinterface | VLAN | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Router0** | Gi0/0.10 | 10 (HR) | 192.168.0.1 | 255.255.255.0 | N/A |
| **Router0** | Gi0/0.20 | 20 (Finance) | 192.168.1.1 | 255.255.255.0 | N/A |
| **Router0** | Gi0/0.99 | 99 (IT) | 192.168.2.1 | 255.255.255.0 | N/A |
| **PC0** | Fa0 | 10 (HR) | 192.168.0.3 | 255.255.255.0 | 192.168.0.1 |
| **PC1** | Fa0 | 10 (HR) | 192.168.0.4 | 255.255.255.0 | 192.168.0.1 |
| **PC2** | Fa0 | 20 (Finance) | 192.168.1.3 | 255.255.255.0 | 192.168.1.1 |
| **Server0** | Fa0 | 99 (IT) | 192.168.2.3 | 255.255.255.0 | 192.168.2.1 |

### Campus B: Multilayer Switching (Layer 3)

| Device | Interface / SVI | VLAN | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **MLS-CAMPUSB-1** | SVI Vlan 30 | 30 (Managers) | 10.0.0.1 | 255.0.0.0 | N/A |
| **MLS-CAMPUSB-1** | SVI Vlan 40 | 40 (Executives)| 10.1.0.1 | 255.0.0.0 | N/A |
| **MLS-CAMPUSB-1** | SVI Vlan 55 | 55 (Admins) | 10.2.0.1 | 255.0.0.0 | N/A |
| **PC3** | Fa0 | 30 (Managers) | 10.0.0.10 | 255.0.0.0 | 10.0.0.1 |
| **PC4** | Fa0 | 30 (Managers) | 10.0.0.11 | 255.0.0.0 | 10.0.0.1 |
| **PC5** | Fa0 | 40 (Executives)| 10.1.0.10 | 255.0.0.0 | 10.1.0.1 |
| **PC6** | Fa0 | 55 (Admins) | 10.2.0.10 | 255.0.0.0 | 10.2.0.1 |

---

## Configuration Steps

### Campus A Configuration (Router-on-a-Stick)

#### Step 1: Configure Layer 2 Switch (`SWI-CAMPUSA-1`)
Create VLANs, assign access ports, and configure the trunk port to Router0:

```cisco
Add commands
```

#### Step 2: Configure Router Subinterfaces (`Router0`)
Configure 802.1Q subinterfaces on the router physical interface to perform Inter-VLAN routing:

```cisco
Add commands
```

---

### Campus B Configuration (Layer 3 Switch SVIs)

#### Step 3: Configure Primary Multilayer Switch (`MLS-CAMPUSB-1`)
Enable IP routing, create VLANs, configure SVIs for gateway routing, and setup the inter-switch trunk link:

```cisco
Add commands
```

#### Step 4: Configure Secondary Multilayer Switch (`MLS-CAMPUSB-2`)
Configure matching VLANs, assign access ports, and enable trunking back to `MLS-CAMPUSB-1`:

```cisco
Add commands
```

---

## Verification & Testing

1. **Verify VLAN Assignment:**
   Run `show vlan brief` on switches to verify that ports are correctly mapped to their respective VLANs.
2. **Verify Trunking Status:**
   Run `show interfaces trunk` on `SWI-CAMPUSA-1`, `MLS-CAMPUSB-1`, and `MLS-CAMPUSB-2` to confirm 802.1Q trunking is operational.
3. **Verify Routing Tables:**
   Run `show ip route` on `Router0` and `MLS-CAMPUSB-1` to confirm connected subnets are present.
4. **Test End-to-End Inter-VLAN Reachability:**
   - **Campus A:** From PC0 (`192.168.0.3`), ping PC2 (`192.168.1.3`) and Server0 (`192.168.2.3`).
   - **Campus B:** From PC3 (`10.0.0.10`), ping PC5 (`10.1.0.10`) and PC6 (`10.2.0.10`).