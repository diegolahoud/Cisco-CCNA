# Lab 05: Dynamic Host Configuration Protocol (DHCPv4)

## Overview
The primary objective of this lab is to configure a central Cisco router (`R1`) as a **DHCPv4 Server** to automatically allocate IP addresses, subnet masks, default gateways, and DNS information to end devices across multiple VLANs. Utilizing central DHCP service reduces administrative overhead, minimizes manual addressing errors, and simplifies network scaling.

This topology demonstrates a centralized DHCP architecture where a intermediate router (`R2`) uses **DHCP Relay** (`ip helper-address`) to forward client broadcast requests across a routed transit link to the centralized DHCP server (`R1`).

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
| **R2** | Gig0/0 | R1 | Gig0/0 | Transit IP | Transit Mask | N/A | Routed Link |
| **R1** | Gig0/0 | R2 | Gig0/0 | Transit IP | Transit Mask | N/A | Central DHCP Server |

---

## Key Technical Concepts

1. **Router-on-a-Stick (ROAS):** `R2` uses 802.1Q encapsulation on subinterfaces `Gig0/1.10` and `Gig0/1.20` to perform inter-VLAN routing for SW1.
2. **Centralized DHCP Server (`R1`):** `R1` defines excluded address ranges and configures pools (`NETWORK10` and `NETWORK20`) for both VLAN subnets.
3. **DHCP Relay Agent (`ip helper-address` on `R2`):** Because client DHCP DISCOVER broadcasts are dropped by default at Layer 3 boundaries, `R2` requires `ip helper-address <R1_IP>` configured under each SVI/subinterface to relay broadcast traffic as unicast to `R1`.

---

## Configuration Steps

### Step 1: Configure Excluded Addresses & DHCP Pools on `R1`

```cisco

```
