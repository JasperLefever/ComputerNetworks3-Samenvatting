# SRWE Module 13: WLAN Configuration

## **13.1 Remote Site WLAN Configuration (SOHO Routers)**

### **SOHO Router Overview**

A SOHO (Small Office/Home Office) router typically integrates:

- **Switch (LAN ports)** – for wired clients
- **WAN port** – for ISP connectivity
- **Wireless AP** – for wireless clients
- **DHCP Server** – assigns IPs automatically
- **NAT** – translates internal private IPs to the public IP
- **Firewall & Security features** – protects LAN
- **QoS, Port forwarding, MAC filtering** – traffic management

---

### **Accessing the Router GUI**

- **Default login info** (IP, user/pass) is **widely known**, hence:

  - **Change default credentials immediately**

- **Steps:**

  1. Connect to router (wired or wireless)
  2. Open browser → enter router IP (e.g., `192.168.0.1`)
  3. Log in with default credentials (commonly `admin/admin`)
  4. Change:

     - Admin password
     - Default IP range
     - DHCP settings

---

### **Basic Network Setup**

- **Renew IP address** after changing LAN IP
- Confirm connection and router access with new IP
- **DHCP Configuration**:

  - Set the start and end IP range
  - Define subnet mask, default gateway
  - Configure DNS servers (optional)

---

### **Basic Wireless Setup**

Configure wireless features:

- **Network mode**: Select appropriate 802.11 standard (b/g/n/ac)

  - 802.11ac is preferred for higher speed and dual-band support

- **SSID**: Name of your wireless network
- **Channel selection**:

  - Manual channel config helps avoid interference
  - 2.4 GHz: Use channels 1, 6, or 11 to avoid overlap
  - 5 GHz: More non-overlapping channels available

- **Security settings**:

  - Prefer **WPA2 Personal (AES)**
  - Set **strong passphrase**
  - Avoid WEP/Open networks for security

---

### **Wireless Mesh Network (WMN)**

- Used to **extend Wi-Fi range**
- **Mesh APs** use same SSID and security settings
- Use **non-overlapping channels**
- Modern mesh systems are **plug-and-play**, configurable via **smartphone apps**

---

### **NAT (Network Address Translation)**

- Translates **private IPv4 addresses** (e.g., `192.168.1.x`) to a single **public IP**
- Enables multiple devices to share a single ISP IP
- Tracks sessions using **source port numbers**

---

### **Port Forwarding vs. Port Triggering**

- **Port Forwarding**: Static rule mapping external port → internal IP\:port (e.g., for game servers, CCTV)
- **Port Triggering**: Dynamic port opening when outbound traffic triggers it (used for apps like VoIP)

---

### **QoS (Quality of Service)**

- Prioritizes bandwidth usage:

  - Voice/Video > Gaming > Web browsing > Background data

- Can be based on:

  - Application type
  - Port number
  - MAC address

---

## **13.2 Configure a Basic WLAN on a WLC (Controller-Based APs)**

### **WLC + Lightweight APs (LAPs)**

- LAPs have **no config interface**—they boot, find WLC, and download config
- Use **LWAPP or CAPWAP** to communicate with WLC
- WLC centralizes management for:

  - SSID broadcasting
  - Security enforcement
  - VLAN tagging
  - Monitoring and troubleshooting

---

### **Logging into the WLC**

- GUI login via IP address
- Access to **Network Summary** dashboard:

  - APs status
  - WLANs configured
  - Connected clients
  - Rogue devices (unauthorized APs)

---

### **Configure a WLAN**

#### Steps:

1. **Create the WLAN**: Choose SSID, Profile name, WLAN ID
2. **Enable it**
3. **Assign interface (VLAN)**
4. **Configure Security**:

   - WPA2-Personal (AES)
   - WPA2-Enterprise (with RADIUS)

5. **Verify Status**:

   - View under **WLANs menu**

6. **Monitor Clients**:

   - Use **Clients tab** to see MAC addresses, IPs, signal strength

---

## **13.3 Configure WPA2 Enterprise WLAN on WLC**

### **SNMP & RADIUS Setup**

- **SNMP**: Sends logs (traps) to SNMP monitoring server
- **RADIUS**: Central authentication server for **802.1X** (AAA services)
- **WPA2 Enterprise requires**:

  - External RADIUS server
  - Username/password authentication
  - Strong mutual authentication

---

### **Configuring SNMP on WLC**

- **MANAGEMENT > SNMP > Trap Receivers**
- Add:

  - Community string (e.g., "public")
  - Server IP

---

### **Configuring RADIUS**

- **SECURITY > RADIUS > Authentication**
- Add RADIUS server:

  - IP address
  - Shared secret (used by WLC, not end-users)

---

### **VLAN Interface for WLAN**

Each WLAN on WLC needs its own **virtual interface (VLAN)**

#### Steps:

1. **CONTROLLER > Interfaces > New**
2. Name: e.g., `vlan5`, VLAN ID: `5`
3. IP address: `192.168.5.254`
4. Gateway: `192.168.5.1`
5. DHCP Server IP: Gateway address (DHCP enabled on router)

---

### **Internal DHCP Scope**

If WLC is the DHCP server:

- **INTERNAL DHCP SERVER > DHCP Scope > New**
- Scope config:

  - Pool: e.g., `192.168.200.240 – .249`
  - Default router: `192.168.200.1`
  - Enable scope

---

### **Final WLAN Configuration for WPA2 Enterprise**

1. **Create new WLAN**
2. Assign to **VLAN5**
3. Confirm defaults: **AES encryption**, **802.1X key management**
4. Under **AAA Servers tab**, select RADIUS server
5. Enable WLAN and verify it is **listed and active**

---

## **13.4 Troubleshooting WLAN Issues**

### General Troubleshooting Method

1. **Identify the problem**
2. **Establish a theory**
3. **Test the theory**
4. **Create action plan**
5. **Implement and test**
6. **Verify and document**

---

### Client Not Connecting

- Check:

  - IP address config (`ipconfig`)
  - Ping gateway or another device
  - NIC driver status
  - Wireless security settings
  - Distance from AP (out of coverage?)
  - Interference on 2.4 GHz (microwaves, cordless phones)

- Physical checks:

  - Are APs online?
  - Bad cables or power loss?

- Ping AP directly if reachable

---

### Network is Slow

- **Upgrade clients** to same standard (avoid slow 802.11b/g clients)
- **Split-band strategy**:

  - 2.4 GHz → general devices
  - 5 GHz → media-heavy devices

- **Rename SSIDs** to identify bands
- **Optimize AP placement**:

  - Minimize obstruction
  - Elevate APs

- **Add extenders** or Powerline Wi-Fi kits if needed

---

### Firmware Updates

- **WLCs can push updates** to all APs
- On Cisco WLC:

  - Go to **WIRELESS > Access Points > Global Configuration**
  - Use **AP Image Pre-download** to update firmware across APs

---

## Key Exam Concepts and Technologies

| Concept             | Description                        |
| ------------------- | ---------------------------------- |
| **SSID**            | Wireless network name              |
| **WLC**             | Central controller for APs         |
| **LAP**             | Lightweight AP (no local config)   |
| **NAT**             | Private ↔ Public IP translation    |
| **QoS**             | Prioritize time-sensitive traffic  |
| **SNMP**            | Network monitoring protocol        |
| **RADIUS**          | AAA protocol for WPA2-Enterprise   |
| **802.1X**          | Authentication framework           |
| **DHCP Scope**      | IP address pool assignment         |
| **VLAN**            | Segregates traffic logically       |
| **WPA2 Enterprise** | Secure authentication using RADIUS |
