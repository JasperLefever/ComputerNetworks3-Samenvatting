# SRWE Module 11: Switch Security Configuration

## **11.1 Implement Port Security**

### Purpose

- Port security protects against Layer 2 attacks, such as **MAC address table flooding**, unauthorized device access, and network reconnaissance.
- Prevents unauthorized MAC addresses from sending/receiving traffic through switch ports.

---

### Securing Unused Ports

- **Why?** Unused ports are open entry points for attackers.
- **How?** Use `shutdown` to administratively disable unused ports. Enables reactivation later with `no shutdown`.
- **Best Practice:** Assign unused ports to a dummy VLAN (e.g., VLAN 999).

---

### Mitigating MAC Address Table Attacks

**MAC Flooding Attack**:

- Attacker floods switch with fake MAC addresses.
- Switch MAC table overflows → acts as a hub → sends frames out all ports → attacker captures traffic (sniffing).

**Prevention**:

- **Port Security** limits the number of **valid MACs** per port.
- MACs can be:
  - **Statically configured**
  - **Dynamically learned**
  - **Dynamically learned and stored (sticky)**

---

### Enabling Port Security

**Requirements**:

- Port must be in `access` or manually configured `trunk` mode.
- By default, ports use **Dynamic Auto** mode, which does **not** support port security.

**Steps**:

1. Set port mode to access: `switchport mode access`
2. Enable port security: `switchport port-security`

---

### Configuring Port Security Options

#### Max MAC Addresses

- Default: 1
- Configurable with:  
  `switchport port-security maximum [value]`
- Switch-dependent max limit (e.g., up to 8192 on some platforms)

#### Secure MAC Address Methods:

1. **Static**

   - MACs explicitly configured
   - Stored in running config
   - Persist through reload
   - Command: `switchport port-security mac-address [MAC]`

2. **Dynamic**

   - Learned at runtime
   - Not saved to config → lost after reboot
   - Useful for temporary devices

3. **Sticky**
   - Dynamically learned + added to running-config
   - Must **save config** to persist
   - Command: `switchport port-security mac-address sticky`

---

### Port Security Violation Modes

Violations occur when:

- More MACs than allowed are detected
- Unknown MAC attempts to send traffic

| Mode       | Behavior                                                 |
| ---------- | -------------------------------------------------------- |
| `shutdown` | Default. Port goes **err-disabled**. Logs, LED off.      |
| `restrict` | Drops traffic, logs violation, increments counter.       |
| `protect`  | Drops traffic silently. No logging or counter increment. |

**Command**:  
`switchport port-security violation [shutdown | restrict | protect]`

---

### Error-Disabled Ports

- Port is administratively and operationally **down**.
- **Indicators**: LED off, `show interface` shows `err-disabled`.

**Recovery**:

1. Diagnose violation cause.
2. Clear state:
   - `shutdown`
   - `no shutdown`

---

### Port Security Aging

- Automatically removes secure MACs after a time period.
- Useful for temporary access or mobile devices.

**Types**:

- **Absolute**: MAC removed after set time.
- **Inactivity**: MAC removed after no activity for set time.

**Command**:  
`switchport port-security aging [static | time <minutes> | type {absolute | inactivity}]`

---

### Verification Commands

| Command                              | Use                                |
| ------------------------------------ | ---------------------------------- |
| `show port-security`                 | Summary of all ports with security |
| `show port-security interface [int]` | Details for a specific port        |
| `show port-security address`         | All secure MAC addresses           |
| `show running-config`                | Check if sticky MACs are saved     |
| `show interface [int]`               | Port status and errors             |

---

## **11.2 Mitigate VLAN Attacks**

### VLAN Hopping Attacks

**1. Switch Spoofing (DTP Attack)**:

- Attacker sends spoofed **Dynamic Trunking Protocol (DTP)** messages to enable trunking.
- Gains access to all VLANs.

**2. Rogue Switch**:

- Attacker connects unauthorized switch.
- Uses DTP to establish trunk → accesses all VLANs.

**3. Double Tagging**:

- Attacker sends packet with two VLAN tags.
- First tag is stripped → inner tag is forwarded to another VLAN.

---

### VLAN Hopping Prevention

| Step | Action                                                                                                        |
| ---- | ------------------------------------------------------------------------------------------------------------- |
| 1    | Set ports to **access mode**: `switchport mode access`                                                        |
| 2    | **Disable unused ports** and assign to unused VLAN                                                            |
| 3    | Manually configure trunks: `switchport mode trunk`                                                            |
| 4    | Disable DTP: `switchport nonegotiate`                                                                         |
| 5    | Change **native VLAN** (not VLAN 1): `switchport trunk native vlan [X]` and **don’t use it for data traffic** |

---

## **11.3 Mitigate DHCP Attacks**

### DHCP Starvation

- Attacker exhausts IP pool with spoofed requests.
- Prevents legitimate clients from getting IPs.

**Mitigation**:

- Use **Port Security** to limit MACs per port.

---

### DHCP Spoofing

- Rogue server assigns incorrect IP/default gateway.
- Can reroute traffic through attacker.

**Mitigation**:  
Use **DHCP Snooping**.

---

### DHCP Snooping

**What it does**:

- Filters DHCP messages on **untrusted ports**
- Allows only DHCP servers on **trusted ports**
- Builds **binding table** of MAC ↔ IP

**Binding Table**:

- Stores: VLAN, MAC, IP, lease time, interface
- Used by DAI and IP Source Guard

---

### Configuring DHCP Snooping

| Step                  | Command                             |
| --------------------- | ----------------------------------- |
| Enable globally       | `ip dhcp snooping`                  |
| Trust server ports    | `ip dhcp snooping trust`            |
| Limit rate on clients | `ip dhcp snooping limit rate [pps]` |
| Enable for VLANs      | `ip dhcp snooping vlan [vlan-id]`   |

**Verify**:

- `show ip dhcp snooping`
- `show ip dhcp snooping binding`

---

## **11.4 Mitigate ARP Attacks**

### ARP Spoofing/Poisoning

- Attacker associates own MAC with default gateway IP.
- Can intercept or redirect traffic.

---

### Dynamic ARP Inspection (DAI)

**Function**:

- Intercepts ARP packets on untrusted ports.
- Compares to DHCP binding table (must exist!).
- Drops invalid or spoofed ARP replies.

**Required**: DHCP Snooping must be enabled first.

---

### DAI Configuration Guidelines

1. Enable **DHCP Snooping**
2. Enable **DAI** on specific VLANs: `ip arp inspection vlan [vlan-id]`
3. Trust uplinks: `ip arp inspection trust`
4. Set access ports as untrusted (default)

---

### DAI Packet Validation

Command:  
`ip arp inspection validate [src-mac] [dst-mac] [ip]`

Checks:

- **src-mac**: Ethernet header vs ARP body sender MAC
- **dst-mac**: Ethernet header vs ARP body target MAC
- **ip**: Invalid IPs (e.g., 0.0.0.0, multicast)

_Note_: Only one `validate` command allowed → use all checks in a single line.

---

## **11.5 Mitigate STP Attacks**

### STP Manipulation

- Attacker sends spoofed **BPDU** to become root bridge.
- Alters topology → forwards traffic through malicious device.

---

### Mitigation Tools

**1. PortFast**

- Bypasses STP listening/learning on **access ports**.
- Brings port up immediately for end devices.

**Command**:

- Per-interface: `spanning-tree portfast`
- Global: `spanning-tree portfast default`

**⚠ Warning**: NEVER enable on trunk ports – creates loops.

---

**2. BPDU Guard**

- Used **with PortFast**.
- Disables (err-disables) any port that **receives a BPDU**.
- Prevents rogue switches on access ports.

**Command**:

- Per-interface: `spanning-tree bpduguard enable`
- Global (for all PortFast): `spanning-tree portfast bpduguard default`

---

### Verification Commands

| Command                                     | Function                     |
| ------------------------------------------- | ---------------------------- |
| `show spanning-tree summary`                | PortFast & BPDU Guard status |
| `show spanning-tree interface [int] detail` | Per-interface state          |
| `show running-config interface [int]`       | Config review                |
