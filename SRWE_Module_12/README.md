# SRWE Module 12: WLAN Concepts

### **12.1. WLAN Concepts and Operation**

#### Definition and Role

- **Wireless LAN (WLAN)**: A LAN that uses **radio waves** instead of cables for connectivity.
- Extends wired networks to wireless clients through **Access Points (APs)**.
- Common in enterprise, public, and home networks for mobility and flexibility.

#### WLAN Architecture

- **Ad Hoc Mode**:

  - Peer-to-peer connection without centralized infrastructure.
  - Rarely used in modern deployments due to lack of scalability/security.

- **Infrastructure Mode**:

  - Devices connect through a central AP.
  - AP connects to distribution system (DS), typically via Ethernet, to bridge wireless and wired segments.
  - Most common WLAN deployment.

#### Key Terms

- **SSID (Service Set Identifier)**: Logical name of a WLAN; can be broadcast or hidden.
- **BSSID (Basic Service Set Identifier)**: MAC address of the AP’s radio interface.
- **ESSID**: When multiple APs share the same SSID to provide extended coverage.
- **Client**: Any wireless-capable device (e.g., phones, laptops, IoT).

---

### **12.2. IEEE 802.11 Standards and Wireless Fundamentals**

#### Standards Overview

| Standard           | Frequency | Max Speed | Channel Width | Key Features                      |
| ------------------ | --------- | --------- | ------------- | --------------------------------- |
| 802.11a            | 5 GHz     | 54 Mbps   | 20 MHz        | Less interference                 |
| 802.11b            | 2.4 GHz   | 11 Mbps   | 22 MHz        | High range, interference-prone    |
| 802.11g            | 2.4 GHz   | 54 Mbps   | 20 MHz        | Backward-compatible with b        |
| 802.11n            | 2.4/5 GHz | 600 Mbps  | 20/40 MHz     | MIMO support, improved throughput |
| 802.11ac           | 5 GHz     | 1.3+ Gbps | up to 160 MHz | MU-MIMO, beamforming              |
| 802.11ax (Wi-Fi 6) | 2.4/5 GHz | >9.6 Gbps | OFDMA support | Improved efficiency & performance |

#### Frequency Bands

- **2.4 GHz**:

  - Longer range but prone to interference (microwaves, Bluetooth).
  - Only **3 non-overlapping channels** (1, 6, 11) in North America.

- **5 GHz**:

  - More available channels (up to 23 non-overlapping).
  - Shorter range, less wall penetration, lower interference.
  - DFS (Dynamic Frequency Selection) required for radar channel detection.

#### Channel Width

- Wider channels (e.g., 40 MHz, 80 MHz, 160 MHz) increase speed but reduce available non-overlapping channels.

---

### **12.3. Wireless Security**

#### Encryption and Authentication

| Security Mode | Encryption | Auth Method | Notes                                |
| ------------- | ---------- | ----------- | ------------------------------------ |
| Open          | None       | None        | Public networks, no confidentiality  |
| WEP           | RC4 (weak) | Shared key  | Deprecated, easily cracked           |
| WPA           | TKIP       | PSK/802.1X  | Legacy, transitional                 |
| WPA2          | AES (CCMP) | PSK/802.1X  | Industry standard                    |
| WPA3          | AES-GCMP   | SAE/802.1X  | Stronger encryption, forward secrecy |

#### Authentication Options

- **PSK (Pre-Shared Key)**: Common for small networks; all users share same key.
- **802.1X / RADIUS** (Enterprise):

  - Client credentials are authenticated by a RADIUS server.
  - More secure, enables per-user access control and logging.

#### Other Security Features

- **MAC Filtering**: Only allows whitelisted MACs; can be spoofed.
- **Hidden SSID**: Prevents SSID from being broadcast, minor security by obscurity.
- **Client Isolation**: Prevents wireless clients from communicating directly.

---

### **12.4. Wireless Router Configuration (Home/Small Office)**

#### Interface Setup

- Access router via web GUI (e.g., 192.168.0.1).
- Set **SSID**, **wireless mode** (e.g., b/g/n/ac), **channel**, and **security settings**.

#### Channel Selection

- Auto vs Manual: Manual allows tuning based on site survey.
- Use channels 1, 6, 11 for 2.4 GHz to avoid overlap.

#### Network Mode

- Mixed mode supports older clients but can slow network.
- Prefer “n only” or “ac only” if all clients support it.

---

### **12.5. Wireless LAN Controller (WLC) and Lightweight APs**

#### WLC Role

- Centralized management of multiple APs.
- Enables unified policies, configuration, firmware updates.

#### Lightweight AP vs Autonomous AP

- **Lightweight AP**: Relies on WLC for control plane; CAPWAP tunnel established.
- **Autonomous AP**: Standalone, configured individually.

#### CAPWAP Protocol

- **Control And Provisioning of Wireless Access Points**
- Encapsulates data/control traffic between WLC and AP.
- UDP ports 5246 (control) and 5247 (data).

#### AP Registration Process

1. AP powers on and gets IP via DHCP.
2. AP discovers WLC (via DHCP Option 43, DNS, broadcast).
3. AP joins WLC via CAPWAP.
4. Firmware check/update occurs.
5. Configuration is downloaded from WLC.

---

### **12.6. WLAN Configuration on WLC (GUI)**

#### WLAN Creation

- Create a **WLAN profile** (Name, SSID, ID).
- Configure **VLAN ID** to bind WLAN to a specific network segment.
- Set **security policies** (e.g., WPA2-PSK, WPA2-Enterprise).
- Apply **QoS, bandwidth limits, client policies** if needed.

#### Interface Mapping

- Management Interface: For WLC administration and AP discovery.
- Dynamic Interface: Used by WLANs for client data traffic.

#### Advanced Features

- **Band Select**: Encourages clients to use 5 GHz.
- **Load Balancing**: Distributes clients evenly across APs.
- **Client Exclusion**: Blocks misbehaving clients temporarily.
- **Coverage Hole Detection**: WLC detects areas with poor signal.

---

### **12.7. WLAN Troubleshooting**

#### Common Wireless Problems

- Low signal strength: Far from AP, obstructions.
- Interference: From other wireless devices or overlapping channels.
- Authentication failures: Incorrect PSK or RADIUS issues.
- DHCP issues: Client not getting IP.
- Excessive retries/packet loss: Signal quality, congestion.

#### Tools and Techniques

- **Wireless Survey Tools**: Analyze signal strength, interference (e.g., Ekahau, NetSpot).
- **WLC Monitoring**: View client association, signal, logs.
- **Ping/Traceroute**: Test connectivity to gateway or external sites.
- **Wireshark**: Packet analysis for deep inspection.

#### Client Troubleshooting

- Check SSID, security type, and password.
- Ensure correct IP/DNS configuration.
- Observe signal strength and speed.
- Use **ipconfig/ifconfig**, **netsh wlan show interfaces** (Windows) for diagnostics.

---

### **12.8. Deployment & Design Best Practices**

#### Site Survey

- Perform **pre-deployment survey** to measure RF environment.
- Identify:

  - Interference sources
  - Dead zones
  - Optimal AP placement

#### AP Placement Guidelines

- Mount high and central in coverage area.
- Avoid metal objects, thick walls.
- Overlap coverage between APs \~15–20% for roaming.
- Consider directional antennas for specific areas (e.g., corridors).

#### Segmentation

- Use **VLANs** to isolate guest, internal, and management traffic.
- Apply **ACLs** for access control between segments.
- Use **802.1Q trunking** for multiple SSIDs per AP via different VLANs.

#### Capacity Planning

- Estimate users per AP (ideally <25 per AP).
- Ensure adequate bandwidth per user based on use case (e.g., voice/video).
