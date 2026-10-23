# Lab 05: Dynamic Host Configuration Protocol (DHCPv4)

## Overview
The primary objective of this lab is to configure a central Cisco router (`R1`) as a **DHCPv4 Server** to automatically allocate IP addresses, subnet masks, default gateways, and DNS information to end devices across multiple VLANs. Utilizing central DHCP service reduces administrative overhead, minimizes manual addressing errors, and simplifies network scaling.

This topology demonstrates a centralized DHCP architecture where an intermediate router (`R2`) uses **DHCP Relay** (`ip helper-address`) to forward client broadcast requests across a routed transit link to the centralized DHCP server (`R1`).

---

## Equipment
- **1 x Router:** `R1` (Cisco 2901) – Central DHCP Server
- **1 x Router:** `R2` (Cisco 2901) – Inter-VLAN Router & DHCP Relay Agent
- **1 x Switch:** `SW1` (Cisco Catalyst 2960) – Access Layer Switch
- **1 x PC:** `PC1` (VLAN 10 - HR) – DHCP Client
- **1 x PC:** `PC0` (VLAN 20 - Marketing) – DHCP Client

---

## Topology & Addressing Table

| Device | Interface / Subinterface | Remote Device | Remote Interface | IP Address / Allocation | Subnet Mask | Default Gateway | VLAN |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **PC1** | Fa0 | SW1 | Fa0/1 | DHCP (10.x.x.x) | 255.0.0.0 | 10.0.0.1 | VLAN 10 (HR) |
| **PC0** | Fa0 | SW1 | Fa0/2 | DHCP (20.x.x.x) | 255.0.0.0 | 20.0.0.1 | VLAN 20 (MARKETING) |
| **SW1** | Fa0/1 | PC1 | Fa0 | N/A (Access) | N/A | N/A | VLAN 10 (HR) |
| **SW1** | Fa0/2 | PC0 | Fa0 | N/A (Access) | N/A | N/A | VLAN 20 (MARKETING) |
| **SW1** | Gig0/1 | R2 | Gig0/1 | N/A (802.1Q Trunk) | N/A | N/A | Trunk |
| **R2** | Gig0/1.10 | SW1 | Gig0/1 | 10.0.0.1 | 255.0.0.0 | N/A | VLAN 10 (HR) |
| **R2** | Gig0/1.20 | SW1 | Gig0/1 | 20.0.0.1 | 255.0.0.0 | N/A | VLAN 20 (MARKETING) |
| **R2** | Gig0/0 | R1 | Gig0/0 | 30.0.0.1 | 255.255.255.252 | N/A | Routed Link |
| **R1** | Gig0/0 | R2 | Gig0/0 | 30.0.0.2 | 255.255.255.252 | N/A | Central DHCP Server |

---

## Key Technical Concepts

1. **Router-on-a-Stick (ROAS):** `R2` uses 802.1Q encapsulation on subinterfaces `Gig0/1.10` and `Gig0/1.20` to perform inter-VLAN routing for SW1.
2. **Centralized DHCP Server (`R1`):** `R1` defines excluded address ranges and configures pools (`NETWORK10` and `NETWORK20`) for both VLAN subnets.
3. **Subnet Specifications:** 
   - **NETWORK10:** `10.0.0.0/8` – Default Gateway: `10.0.0.1` – Excluded IP Range: `10.0.0.1` to `10.0.0.100`
   - **NETWORK20:** `20.0.0.0/8` – Default Gateway: `20.0.0.1` – Excluded IP Range: `20.0.0.1` to `20.0.0.100`
4. **DHCP Relay Agent (`ip helper-address` on `R2`):** Because client DHCP DISCOVER broadcasts are dropped by default at Layer 3 boundaries, `R2` requires `ip helper-address 30.0.0.2` configured under each subinterface to relay broadcast traffic as unicast to `R1`.

---

## Configuration Steps

> **Notes:** The underlying topology (VLAN 10, VLAN 20, 802.1Q trunking on `SW1`, and Inter-VLAN routing subinterfaces on `R2`) is pre-configured. Hosts initially used static IP addressing to verify baseline Inter-VLAN connectivity. The objective of this lab is to configure the central DHCP server on `R1`, enable DHCP Relay on `R2`, and transition hosts from static to dynamic addressing.

### Step 1: Configure Excluded Addresses & DHCP Pools on `R1`

> **Notes:** You need to exlude the addresses in the router configuration mode. If you do it inside the dhcp-configuration mode it will not work so make sure you are in the configure mode.

```cisco
R1(config)# ip dhcp excluded-address 10.0.0.1 10.0.0.100
R1(config)# ip dhcp excluded-address 20.0.0.1 20.0.0.100
R1(config)# ip dhcp pool NETWORK10
R1(dhcp-config)#network 10.0.0.0 255.0.0.0
R1(dhcp-config)# default-router 10.0.0.1
R1(dhcp-config)#dns-server 8.8.8.8
R1(dhcp-config)# ip dhcp pool NETWORK20
R1(dhcp-config)#network 20.0.0.0 255.0.0.0
R1(dhcp-config)# default-router 20.0.0.1
R1(dhcp-config)#dns-server 8.8.8.8
R1(dhcp-config)# exit
R1(config)# do write memory
```

### Step 2: Configure the link between R1 and R2 on the 30.0.0.0/8 subnet

! This is in R1

```cisco
R1(config)#int gig0/0
R1(config-if)#ip add 30.0.0.2 255.0.0.0
R1(config-if)#no shutdown
```

! This is in R2
```cisco
R2(config)#int gig0/0
R2(config-if)#ip add 30.0.0.1 255.0.0.0
R2(config-if)#no shutdown
```

### Step 3: Configure the IP helper on the R2 subinterfaces so they can relay to the DHCP server

**Notes:** Remember the default interface on each subinterface is already set

```cisco
R2(config)#int gig0/1.10
R2(config-subif)#ip helper-address 30.0.0.2
R2(config)#int gig0/1.20
R2(config-subif)#ip helpper-address 30.0.0.2
R2(config-subif)# exit
R2(config)# do write memory
```

### Step 4: Configure static routes between R1 and R2

! This is for R2
```cisco
R2(config)# ip route 10.0.0.0 255.0.0.0 30.0.0.2
R2(config)# ip route 20.0.0.0 255.0.0.0 30.0.0.2
```

! This is for RR
```cisco
R1(config)# ip route 10.0.0.0 255.0.0.0 30.0.0.1
R1(config)# ip route 20.0.0.0 255.0.0.0 30.0.0.1
```

## Step 5: Test the DHCP server is handling the IP to the devices

You need to switch the PC from static IP to DHCP, and click on the forward buttom to accelerate the process. If succesful, the desktop should have a received an IP above the 100.
