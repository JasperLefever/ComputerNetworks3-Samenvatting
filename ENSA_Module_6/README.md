# ENSA Module 6: NAT for IPv4

## 6.1 NAT Characteristics

### Why NAT Exists

- IPv4 address exhaustion led to the need for mechanisms like NAT.
- **Private IPs** allow enterprises to deploy **internal networks** without needing public IPs.
- NAT allows **internal clients** to reach the internet while **hiding** the real internal network structure.

### RFC 1918 - Private IP Address Spaces

- **10.0.0.0/8** (Class A)
- **172.16.0.0/12** (Class B)
- **192.168.0.0/16** (Class C)

These IP addresses **cannot be routed** directly on the public Internet.

---

### Detailed NAT Operation (Packet Flow)

When an internal device initiates traffic to the internet:

1. Packet is sent with **source IP = private IP**, **destination IP = public server**.
2. Router with NAT inspects the packet:
   - Matches source IP to an inside local address.
   - Assigns a public IP (inside global).
   - Records the mapping in the **NAT translation table**.
3. Router rewrites packet:
   - Changes **source IP** to inside global address.
   - Leaves **destination IP** unchanged.
4. Packet is forwarded to the public network.
5. Response from the public server:
   - Destination IP is the inside global address.
6. Router checks NAT table:
   - Finds the original inside local mapping.
   - Rewrites the destination IP back to the original private IP.
7. Packet is forwarded internally.

---

### Critical NAT Terminology

| NAT Term           | Detailed Meaning                                                                      | Key Example                       |
| :----------------- | :------------------------------------------------------------------------------------ | :-------------------------------- |
| **Inside Local**   | The private source IP assigned to a host inside the network.                          | `192.168.10.10`                   |
| **Inside Global**  | The public IP assigned to the same internal device, visible to the outside world.     | `209.165.200.226`                 |
| **Outside Global** | The public IP address of the external destination (e.g., a web server).               | `209.165.201.1`                   |
| **Outside Local**  | The destination IP address as seen internally (rarely different from outside global). | Typically same as outside global. |

---

# 6.2 Types of NAT

---

### Static NAT

- **Manual 1:1 mapping** between a private IP and a public IP.
- Consistent translation — IP never changes.
- Used for:
  - Internal servers needing **inbound access** from the internet (e.g., email servers, web servers).
  - Devices that need to be **reached** from outside.

🔵 **Key Considerations**:

- **Public IPs must be sufficient** for every device that needs static mapping.
- If a public IP address changes, it must be manually updated.

---

### Dynamic NAT

- **Dynamically** assigns a public IP from a **predefined pool** when needed.
- Mapping is **temporary** and exists only during the session.
- Once the session is finished, the IP is **returned to the pool**.
- Configured with:
  - A **pool of public IPs**.
  - An **access control list (ACL)** to define which inside IPs are eligible.

🔵 **Key Characteristics**:

- Still requires **enough public IPs** for concurrent users.
- **First-come, first-served** basis — if all public IPs are used, new sessions fail.

---

### PAT (Port Address Translation) / NAT Overload

- **Many-to-one** or **many-to-few** mappings.
- All inside hosts share **one or a few public IPs**.
- Differentiation is achieved by **modifying the Layer 4 source port numbers**.

🔵 **How PAT works under the hood**:

- Each internal session gets a **unique source port number** at the NAT router.
- NAT builds a table based on:
  - **Inside Local IP:Port** ↔ **Inside Global IP:Port**.

🔵 **Real-world behavior**:

- Thousands of internal clients can connect simultaneously using **one public IP**.
- Source ports are carefully selected to avoid collisions. If a collision occurs, a new port is picked.

| Feature                      | NAT     | PAT       |
| :--------------------------- | :------ | :-------- |
| Address translation only     | ✅      | 🚫        |
| Address and port translation | 🚫      | ✅        |
| Public IP conservation       | Limited | Excellent |

---

# 6.3 Advantages and Disadvantages of NAT

---

### Advantages

- **Address conservation**: Reuses private IPs internally.
- **Flexibility**: Changing ISPs only requires modifying NAT settings, not internal IPs.
- **Security by obscurity**: Internal network structure is hidden from outsiders.
- **Port multiplexing**: Multiple connections over single public IP (via PAT).

---

### Disadvantages

- **Performance overhead**: Packets must be modified → slight delays.
- **Breaks end-to-end principle**:
  - NAT complicates or breaks protocols like **IPsec VPNs**, **SIP VoIP**, and **peer-to-peer** apps.
- **NAT Traversal protocols** (e.g., STUN, TURN) are often needed.
- **Some protocols require extra configurations** to work properly behind NAT (e.g., FTP, SIP).

---

# 6.4 Static NAT - Full Details

---

### Use Cases

- External services (e.g., web, mail servers).
- Remote access to internal hosts.

---

### Static NAT Packet Flow

- Internal server sends or receives packets.
- Router always maps internal IP to static global IP.
- No dynamic lookup — translation is preconfigured.

---

### Verification Tools

- `show ip nat translations` — Static mappings are **always listed**, even if no traffic is present.
- `show ip nat statistics` — Tracks hits, misses, active translations.

---

# 6.5 Dynamic NAT - Full Details

---

### Dynamic NAT Workflow

1. Packet from internal device arrives at NAT router.
2. ACL determines eligibility for NAT.
3. If eligible:
   - A public IP is dynamically selected from the pool.
   - Mapping recorded in NAT table.
4. Packet source IP is translated.
5. When the session ends (timeout or connection close):
   - Mapping is deleted.
   - IP is returned to the pool.

---

### Dynamic NAT Key Points

- **Stateful** — Requires router memory to track active translations.
- Mapping disappears once session ends or expires.
- **Verification commands**: `show ip nat translations`, `show ip nat statistics`.

---

# 6.6 PAT (Overload) - Full Details

---

### How PAT Handles Collisions

- If the same port is already used for another session, PAT **finds an unused port**.
- If **all ports are exhausted** (rare in large NAT pools), new sessions are dropped.

---

### PAT Internal Tables

PAT tracks each translation with 5-tuple information:

- Source IP
- Source Port
- Destination IP
- Destination Port
- Protocol (TCP/UDP)

This allows PAT to **multiplex thousands of sessions**.

---

### PAT Special Behaviors

- Some legacy applications **hard-code** specific source ports. PAT must adjust mappings dynamically.
- When NAT is overwhelmed (e.g., gaming servers, heavy UDP use), NAT "pinhole" techniques or firewall rules are required.

---

# 6.7 NAT64 for IPv6

---

### Role of NAT64

- Bridges communication between **IPv6-only clients** and **IPv4-only servers**.
- Typically used with **DNS64**, which synthesizes fake AAAA records pointing to NAT64 gateway.

---

### NAT64 Important Notes

- NOT meant to allow private IPv6 to public IPv6 translations.
- Temporary solution until full IPv6 adoption.
- Adds **protocol translation overhead** (IPv6 ↔ IPv4).

---

## Exam Tips Summary

- Always visualize packet flows (inside → outside and vice versa).
- Understand when **static NAT** vs **dynamic NAT** vs **PAT** is appropriate.
- Know that PAT modifies **IP + port**, NAT modifies **IP only**.
- Remember the **impact of NAT on tunneling protocols** (e.g., IPsec, VoIP).
- Understand how **address pools** and **overloading** are configured.
- **Static entries** always visible; **dynamic entries** time out.
- NAT64 only handles **IPv4–IPv6 translation**, NOT private-to-global IPv6.
