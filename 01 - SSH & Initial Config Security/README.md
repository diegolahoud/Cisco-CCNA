# Lab 01: Initial Device Configuration & SSH Security

## Overview
This lab focuses on fundamental initial configurations for Cisco routers and switches. The primary objective is to harden device management, securing both physical console access and remote administration using encrypted protocols.

## Objectives
- **Device Identification:** Assign unique hostnames to clearly identify devices across the network topology.
- **Console Line Security:** Configure password protection and local user authentication on the physical console line to restrict physical access.
- **Password Encryption:** Enable global password encryption (`service password-encryption`) and secure Privileged EXEC mode using encrypted secrets (`enable secret`).
- **Secure Remote Access (SSH):** Configure domain names, RSA key pairs, and SSH on Virtual Terminal (VTY) lines to ensure remote management traffic is encrypted and Telnet is disabled.
- **Banner Configuration:** Implement a Message of the Day (MOTD) banner to display legal warnings to unauthorized users.

---

## Equipment
- **1 x PC** (PC0)
- **1 x Cisco Switch** (Catalyst 2960)

---

## Configuration Steps

### Step 1: Physical Connections & Terminal Setup
1. Connect a **Console Cable** from the PC's `RS-232` port to the `Console` port on **SW1**.
2. Connect a **Copper Straight-Through** cable from the PC's `FastEthernet0` port to `FastEthernet0/1` on **SW1**.
3. On **PC0**, open the **Desktop** tab, launch **Terminal**, leave the default serial settings (9600 baud, 8 data bits, no parity, 1 stop bit, no flow control), and click **OK** to enter the switch CLI.

### Step 2: Configure Device Hostname
Enter Global Configuration Mode and assign a hostname to the switch:

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# do write memory
```

### Step 3: Secure Physical Console Access
Configure password protection on the physical console line to restrict local access:

```cisco
SW1(config)# line console 0
SW1(config-line)# password lab1
SW1(config-line)# login
SW1(config-line)# end
SW1# write memory
```
*Verification:* Type `exit` to log out of the switch session. Press `Enter` to reconnect and verify that the switch prompts for the console password (`lab1`).

### Step 4: Secure Privileged EXEC Mode
Configure an encrypted secret to protect access to Privileged EXEC mode (`#` prompt):

```cisco
SW1(config)# configure terminal
SW1(config)# enable secret lab1-secret
SW1(config)# end
SW1# write memory
```
*Verification:* From User EXEC mode (`SW1>`), type `enable`. Verify that the switch requires `lab1-secret` before granting access to Privileged EXEC mode (`SW1#`).

### Step 5: Encrypt Cleartext Passwords
By default, console passwords appear as plain text in the running configuration. Enable global password encryption to obfuscate all cleartext passwords:

```cisco
SW1(config)# configure terminal
SW1(config)# service password-encryption
SW1(config)# end
SW1# write memory
```
*Verification:* Run `show running-config` and check the `line con 0` section. Verify that the password string is now obfuscated with Type 7 encryption instead of plain text.

### Step 6: Configure SSH on the Switch
Configure SSH to enforce encrypted remote management and restrict unencrypted Telnet access across VTY lines (0–15):

```cisco
SW1# configure terminal
SW1(config)# ip domain-name cisco.com
SW1(config)# crypto key generate rsa
```
> **Note:** Enter `1024` for the modulus size when prompted by Cisco IOS.

```cisco
SW1(config)# ip ssh version 2
SW1(config)# username admin secret lab1-ssh
SW1(config)# line vty 0 15
SW1(config-line)# transport input ssh
SW1(config-line)# login local
SW1(config-line)# end
SW1# write memory
```
*Verification:* Type `show ip ssh` in privileged EXEC mode to verify SSH version 2 is active.

### Step 7: Configure Management SVI & Test SSH Connection

1. **Assign PC IP Address:** On **PC0**, go to **Desktop > IP Configuration** and assign IP `192.168.1.1` with subnet mask `255.255.255.0`.
2. **Configure VLAN & Switch Virtual Interface (SVI):** Create management VLAN 1000, assign port `FastEthernet0/1` to VLAN 1000, and assign IP `192.168.1.2` to the SVI:

```cisco
SW1# configure terminal
SW1(config)# vlan 1000
SW1(config-vlan)# name Management
SW1(config-vlan)# exit

SW1(config)# interface FastEthernet 0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 1000
SW1(config-if)# exit

SW1(config)# interface vlan 1000
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# end
SW1# write memory
```

3. **Test Remote SSH Access:** On **PC0**, open **Command Prompt** and initiate an SSH session to the switch SVI:
```bash
ssh -l admin 192.168.1.2
```
Enter password `lab1-ssh` when prompted to verify successful remote access.