# SRWE Module 13: WLAN Configuration

## 13.1. **Introduction to WLANs**

- WLANs extend Layer 2 access to users via **radio frequency (RF)** instead of copper or fiber cabling.
- Provide **mobility**, **flexibility**, and **cost-effective deployment**, particularly in dynamic or mobile environments.
- Operate over the **OSI Layer 1 (Physical)** and **Layer 2 (Data Link)** layers with adaptations for wireless media.

### Key WLAN Definitions:

- **SSID (Service Set Identifier):** Logical identifier (network name) broadcast by APs.
- **BSS (Basic Service Set):** Single AP + all associated clients; identified by a unique BSSID (MAC address of AP).
- **ESS (Extended Service Set):** Two or more APs with the same SSID, providing seamless client roaming.
- **IBSS (Independent BSS):** Ad hoc, peer-to-peer wireless network with no central AP.

---

## 13.2. **IEEE 802.11 Wireless Standards**

### Frequency Bands:

- **2.4 GHz**: Better range, but more interference (11 channels, only 3 non-overlapping: 1, 6, 11).
- **5 GHz**: Higher bandwidth and more non-overlapping channels (\~23 usable in the US); reduced range.
- **6 GHz (Wi-Fi 6E)**: New spectrum with even less congestion, only available in Wi-Fi 6E devices.

### Major IEEE Standards:

| Standard           | Band      | Modulation      | Max Data Rate | Key Features                                |
| ------------------ | --------- | --------------- | ------------- | ------------------------------------------- |
| 802.11a            | 5 GHz     | OFDM            | 54 Mbps       | Shorter range, less interference            |
| 802.11b            | 2.4 GHz   | DSSS            | 11 Mbps       | Legacy; highly prone to interference        |
| 802.11g            | 2.4 GHz   | OFDM            | 54 Mbps       | Backward compatible with 802.11b            |
| 802.11n            | 2.4/5 GHz | OFDM + MIMO     | 600 Mbps      | Channel bonding (20/40 MHz), MIMO           |
| 802.11ac           | 5 GHz     | OFDM + MU-MIMO  | \~6.9 Gbps    | Wider channels (up to 160 MHz), beamforming |
| 802.11ax (Wi-Fi 6) | 2.4/5 GHz | OFDMA + MU-MIMO | 9.6 Gbps      | Improved efficiency, latency, capacity      |

---

## 13.3. **WLAN Architecture and Topologies**

### Infrastructure Mode:

- Clients associate with an AP; AP connects to LAN infrastructure.
- Preferred for **enterprise environments** with centralized control and scalability.

### Ad Hoc Mode (IBSS):

- Peer-to-peer connections; no APs.
- Used in temporary setups (e.g., file sharing, emergency comms).

### Mesh WLAN:

- APs (called **Mesh Points**) wirelessly connect to each other.
- One AP acts as a **Mesh Portal**, bridging to wired network.
- Ideal for **hard-to-wire environments** (e.g., outdoors, disaster zones).

---

## 13.4. **WLAN Components**

### Access Point (AP):

- Layer 2 device bridging wireless and wired networks.
- Types:

  - **Autonomous APs**: Manage themselves.
  - **Lightweight APs**: Rely on WLC for configuration and control plane functions.

- Uses radio interfaces to transmit/receive on defined channels.

### Wireless LAN Controller (WLC):

- Centralized control of multiple APs.
- Handles **RF management, security policies, firmware upgrades**, and **roaming**.
- Enables features like **load balancing**, **client steering**, and **radio resource management (RRM)**.

### Wireless Clients:

- Devices with wireless NICs.
- Must match **SSID**, **authentication**, **encryption**, and **frequency band** to associate.

---

## 13.5. **Wireless Channel Planning**

### 2.4 GHz Considerations:

- Channels overlap significantly; use only 1, 6, 11 in most countries.
- Affected by **microwaves**, **Bluetooth**, **cordless phones**.

### 5 GHz Considerations:

- Offers more non-overlapping channels.
- Some channels are subject to **DFS (Dynamic Frequency Selection)** due to radar interference concerns.

### Best Practices:

- **Minimize co-channel interference** by using non-overlapping channels.
- Use **site survey tools** to visualize and select optimal channels.

---

## 13.6. **Wireless Frame Structure**

### Differences from Ethernet:

- Uses **management**, **control**, and **data frames**.
- Includes fields like **Frame Control**, **Duration**, **Address 1–4**, and **Sequence Control**.
- **Management Frames**: Beacon, probe request/response, authentication, association.
- **Control Frames**: RTS, CTS, ACK.
- **Data Frames**: Encapsulate upper-layer data (e.g., IP packets).

---

## 13.7. **Association and Authentication Process**

1. **Scanning (Passive/Active):**

   - Passive: Listen for beacon frames.
   - Active: Send probe requests; receive probe responses.

2. **Authentication:**

   - Basic 802.11 authentication (open or shared key).
   - Determines if a client is allowed to proceed with association.

3. **Association Request/Response:**

   - Exchange of supported capabilities (e.g., data rates, security).
   - Establishes Layer 2 connection between AP and client.

4. **4-Way Handshake (WPA2):**

   - Secure key exchange between client and AP using PMK and PTK.
   - Ensures data encryption keys are agreed upon securely.

---

## 13.8. **WLAN Security Mechanisms**

### Encryption Protocols:

| Protocol | Encryption | Authentication                              | Vulnerabilities                                   |
| -------- | ---------- | ------------------------------------------- | ------------------------------------------------- |
| WEP      | RC4        | Shared/Open                                 | Weak IV, easily cracked                           |
| WPA      | TKIP/RC4   | PSK/802.1X                                  | Legacy, better than WEP                           |
| WPA2     | AES/CCMP   | PSK/802.1X                                  | Industry standard                                 |
| WPA3     | AES/GCMP   | SAE (Simultaneous Authentication of Equals) | Resistant to offline attacks, stronger encryption |

### Authentication Modes:

- **Open**: No authentication; anyone can connect.
- **PSK (Pre-Shared Key)**: Shared passphrase.
- **802.1X/EAP**: Enterprise-grade authentication; requires RADIUS server.

### Enterprise Considerations:

- Use **WPA2 or WPA3-Enterprise** with **RADIUS + EAP (PEAP, EAP-TLS)**.
- Implement **role-based access**, **guest VLANs**, and **segmentation** for enhanced security.

---

## 13.9. **WLAN Deployment and Site Survey**

### Site Survey Goals:

- Identify:

  - RF interference sources
  - Signal dead zones
  - Optimal AP placement

### Factors Affecting Coverage:

- Building materials (concrete, metal block RF).
- Number of users per AP (overloading causes drops).
- Signal attenuation due to distance and obstacles.

### Best Practices:

- Overlap AP coverage \~15–20% to support seamless roaming.
- Mount APs **below ceiling level** for indoor use, **above obstructions** outdoors.
- Use **channel planning tools** to minimize interference.

---

## 13.10. **Client Roaming Mechanisms**

- Enables seamless transition between APs within an ESS.
- Triggered by signal strength thresholds or client logic.
- Requires:

  - Consistent SSID
  - Identical security settings
  - Overlapping AP coverage

### Controller-Assisted Roaming:

- WLC uses features like **802.11r (Fast BSS Transition)** and **802.11k/v** to assist client handoff.
- Improves roaming speed and connection stability.

---

## 13.11. **Controller-Based vs Autonomous WLANs**

| Feature         | Autonomous AP           | Controller-Based WLAN               |
| --------------- | ----------------------- | ----------------------------------- |
| Management      | Per-AP configuration    | Centralized via WLC                 |
| Scalability     | Low                     | High                                |
| Roaming         | Basic, client-dependent | Seamless with WLC coordination      |
| Deployment size | Small/home offices      | Large enterprises                   |
| Feature Set     | Limited                 | Advanced: RRM, client steering, QoS |

---

## 13.12. **Monitoring and Troubleshooting**

### Key Metrics:

- **RSSI (Received Signal Strength Indicator)**: > -67 dBm for VoIP, > -70 dBm for general use.
- **SNR (Signal-to-Noise Ratio)**: > 20 dB recommended.
- **Channel Utilization**: High % may indicate interference or congestion.

### Troubleshooting Steps:

1. Verify physical environment and AP placement.
2. Check for overlapping SSIDs on same channels.
3. Inspect authentication logs (RADIUS, WLC).
4. Use tools:

   - **Wi-Fi analyzers** (NetSpot, Ekahau)
   - **Packet capture** (Wireshark)
   - **Ping/traceroute** to verify connectivity.
