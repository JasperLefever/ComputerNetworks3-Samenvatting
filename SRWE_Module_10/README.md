# SRWE Module 10: LAN Security Concepts

## **10.1 Endpoint Security**

### Modern Network Threats:

- **DDoS (Distributed Denial of Service)**: Uses botnets/zombie devices to overwhelm services (web, DNS, etc.) via massive traffic, disrupting access.
- **Data Breach**: Unauthorized access to sensitive data (customer records, trade secrets); commonly via unpatched systems or social engineering.
- **Malware**: Includes viruses, worms, spyware, ransomware. **WannaCry** encrypts files and demands payment. Often enters via phishing emails or malicious downloads.

### Security Devices at Network Edge:

- **VPN-Enabled Router**: Encrypts data across public networks. Secure remote access.
- **NGFW (Next-Generation Firewall)**:
  - **Stateful inspection**: Tracks active sessions.
  - **Application visibility**: Identifies app-layer traffic (e.g., Skype, Facebook).
  - **NGIPS**: Blocks intrusions.
  - **AMP**: Blocks malware based on behavior analysis.
  - **URL filtering**: Blocks harmful domains.
- **NAC (Network Access Control)**:
  - Manages **AAA** (Authentication, Authorization, Accounting).
  - Controls user/device access based on posture (e.g., device type, compliance).
  - Example: Cisco ISE enforces access policies dynamically.

### Endpoint Protections:

- **Endpoints** = Laptops, phones, printers, IP phones, BYOD.
- Vulnerable to email/web-borne malware (common initial attack vector).
- Traditional tools: Antivirus, host-based firewalls, HIPS (Host Intrusion Prevention System).
- **Modern endpoint protection stack**:
  - **NAC**
  - **AMP (Cisco or other)**
  - **ESA (Email Security Appliance)**
  - **WSA (Web Security Appliance)**

### Cisco Email Security Appliance (ESA):

- Monitors **SMTP traffic**.
- Uses **Cisco Talos** feeds for live updates (every 3–5 mins).
- Functions:
  - Block known threats
  - Detect stealth malware (e.g., polymorphic)
  - Filter malicious links
  - Prevent access to newly infected sites
  - Encrypt outbound content to avoid **data loss**

### Cisco Web Security Appliance (WSA):

- Protects web access; sits in path of user → internet.
- Capabilities:
  - Malware scanning
  - URL categorization + blacklisting
  - Control user access to web services (e.g., allow YouTube, block adult content)
  - Inspect encrypted traffic (SSL/TLS)
  - Enforce **acceptable use policies** (by user, time, app)

---

## **10.2 Access Control**

### Authentication Methods:

- **Console/VTY/AUX passwords**: Basic, no encryption.
- **SSH**: Secure remote access with encrypted credentials.
- **Local database**: User credentials stored on each device.
  - Good for small-scale use only.
  - **Not scalable**, no centralized management or fallback.

### AAA (Authentication, Authorization, Accounting):

- **Authentication**: Verifies identity (who).
- **Authorization**: Defines permissions (what they can do).
- **Accounting**: Logs activity (what they did).

### AAA Authentication Methods:

- **Local**:

  - Usernames/passwords stored on local device.
  - Simple but manual; best for small networks.

- **Server-Based**:
  - Centralized database (e.g., RADIUS or TACACS+ server).
  - Better scalability and security.
  - **RADIUS**: Encrypts password only; combines AAA.
  - **TACACS+**: Encrypts entire packet; separates AAA functions.

### Authorization:

- Triggered **after** authentication.
- AAA server checks user attributes (group, time, IP) → enforces policies.
- Determines CLI access level, allowed protocols, bandwidth usage, etc.

### Accounting:

- Tracks:
  - Connection times
  - Commands entered (especially config mode)
  - Byte/packet counters
- Useful for audits, forensics, and billing.

### IEEE 802.1X Authentication:

- **Port-based control**: Blocks network access until device is authenticated.
- **Roles**:
  - **Supplicant**: The client (e.g., laptop) with 802.1X agent.
  - **Authenticator**: Switch or AP controlling port access.
  - **Authentication Server**: Validates identity (e.g., RADIUS).

---

## **10.3 Layer 2 Security Threats**

### Why Layer 2 is Critical:

- Often overlooked, but fundamental. A compromise here compromises all upper-layer defenses (VPN, firewall, etc.).
- LANs now host **untrusted** devices (BYOD, guests), making Layer 2 security essential.

### Secure Management Guidelines:

- Use **encrypted management protocols** (SSH, SCP, SFTP).
- **Out-of-band management**: Use isolated network/VLAN.
- **Dedicated management VLANs**: Carry only management traffic.
- Apply **ACLs** to restrict access to management interfaces.

---

## **10.4 MAC Address Table Attack**

### MAC Table Operation:

- Switch learns MACs from source addresses of frames.
- Maps MACs to ports → enables efficient forwarding.

### MAC Flooding Attack:

- Attacker floods switch with thousands of fake source MACs.
- Table fills up → switch **fails open**, floods unknown traffic to all ports.
- Attacker sniffs traffic on VLAN → eavesdrops.

### Impact:

- Breaks confidentiality.
- Affects local VLAN only, but can cascade to other switches if flooded traffic propagates.

### Mitigation:

- **Port Security**:
  - Limits # of allowed MACs per port.
  - Can shut down, restrict, or protect if limit is exceeded.

---

## **10.5 LAN Attacks**

### VLAN Hopping Attacks:

#### Classic VLAN Hopping:

- Host pretends to be a switch.
- Spoofs **DTP and 802.1Q** to force trunking.
- Gains access to **all VLANs** on the trunk.

#### Double-Tagging:

- Sends frame with **two VLAN tags**:
  - Outer tag = native VLAN (gets stripped at first switch).
  - Inner tag = target VLAN.
- Second switch forwards to target VLAN.
- Works **only when attacker is on native VLAN**.

**Mitigation**:

- Disable trunking on access ports (`switchport mode access`).
- Manually configure trunk ports.
- Use dedicated VLAN for native VLAN, not shared with users.

### DHCP Attacks:

#### Starvation:

- Attacker uses tools (e.g., **Gobbler**) to lease all IPs using fake MACs.
- Causes **DoS** for legitimate users.

#### Spoofing:

- Rogue DHCP server provides incorrect:
  - Gateway IP → MITM
  - DNS IP → redirect to malicious sites
  - Client IP → invalid IPs (DoS)

**Mitigation**:

- **DHCP Snooping**:
  - Trust only DHCP responses from authorized ports.
  - Builds MAC-to-IP mapping database for use in DAI and IPSG.

### ARP Attacks:

#### Spoofing/Poisoning:

- Attacker sends **gratuitous ARP** replies linking their MAC to gateway IP.
- Creates **MITM** position.
- Tools: Ettercap, Cain & Abel, etc.

**Mitigation**:

- **DAI (Dynamic ARP Inspection)**:
  - Validates ARP packets against DHCP Snooping table.

### Address Spoofing:

- **IP Spoofing**: Uses a valid or fake IP to evade security.
- **MAC Spoofing**: Forwards traffic intended for another device.

**Mitigation**:

- **IP Source Guard (IPSG)**:
  - Matches MAC/IP against DHCP snooping database.
  - Blocks unverified hosts.

### STP Attack:

- Broadcasts **BPDUs** with low priority to become root bridge.
- Changes traffic paths → intercepts traffic.

**Mitigation**:

- Enable **BPDU Guard**:
  - Disables port if BPDU is received unexpectedly (e.g., on access port).

### CDP Reconnaissance:

- **CDP (Cisco Discovery Protocol)**:
  - Periodic, unencrypted, unauthenticated broadcast.
  - Reveals IP, OS version, VLAN info → used for further attacks.

**Mitigation**:

- Disable CDP on untrusted ports or globally (`no cdp run`).
- **LLDP** (Link Layer Discovery Protocol) should also be disabled on edge interfaces.
