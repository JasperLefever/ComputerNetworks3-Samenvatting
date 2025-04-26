# ENSA Module 2: Single-Area OSPFv2 Configuration

## 2.1. OSPF Router ID (RID)

### What It Is:

- **Router ID** is a unique 32-bit identifier in an OSPF network, formatted like an IPv4 address.
- Used by routers to:
  - Identify themselves in OSPF messages (Hello packets, LSAs, etc.).
  - Determine DR/BDR elections in multiaccess networks.
  - Resolve DBD (Database Description) exchange order during synchronization (higher RID goes first).

### How It's Chosen (Order of Precedence):

1. **Manual assignment** with `router-id <id>`.
2. **Highest IP address of a loopback interface**.
3. **Highest IP address of any active physical interface**.

**Best Practice**: Always manually configure router ID for consistency and easy troubleshooting.

### Important:

- **Changing RID** after OSPF is running requires a **process restart** (`clear ip ospf process`) or router reload.
- Loopback interfaces are preferred because they are always up, unlike physical interfaces that may flap.

---

## 2.2. Point-to-Point OSPF Networks

### How OSPF Is Enabled:

- **Router mode**: `network <address> <wildcard> area <id>`.
  - All interfaces matching the network statement are included in OSPF.
- **Interface mode**: `ip ospf <process-id> area <id>`.
  - OSPF is enabled on the specific interface.

### Why Use Point-to-Point:

- No need for DR/BDR elections (only two routers).
- Faster convergence (no unnecessary elections).
- Simplifies neighbor relationships.

### Wildcard Masks:

- Wildcard mask is the **inverse** of the subnet mask.
- Calculation: `Wildcard = 255.255.255.255 - Subnet Mask`.

**Example**:

- Subnet Mask 255.255.255.0 ➔ Wildcard 0.0.0.255.

**Note**: You can simplify configuration by using exact IPs with wildcard 0.0.0.0.

---

### Passive Interfaces:

- Stops OSPF Hello packets out an interface (doesn't form neighbor relationships).
- Still **advertises** the network.
- Used typically on user-facing LANs, not on router-to-router links.
- **Benefits**:
  - Reduces unnecessary OSPF traffic.
  - Prevents rogue devices from participating in OSPF.
  - Enhances network security. Since messages cannot be intercepted by packet sniffers.

Conclusion: Only router to router links should be active OSPF interfaces.

### Default OSPF Behavior on Ethernet:

- Ethernet interfaces are treated as **broadcast networks** by default.
- DR/BDR elections occur, even if only two routers are connected.

NETWORK TYPE: **Broadcast** (default for Ethernet).
NETWORK TYPE: **Point-to-Point** (for serial links).

- Or use `ip ospf network point-to-point` to force Ethernet to behave like a serial link.

**Optimization**:  
Set `ip ospf network point-to-point` to make Ethernet behave like serial (point-to-point).

Only do this when you are sure there are only two routers on the link. Otherwise, it can cause issues.

---

## 2.3. Multiaccess OSPF Networks (Ethernet LANs)

### Concept:

- Multiaccess = Many routers share a broadcast medium (e.g., Ethernet).
- Without optimization, every router would need a full mesh of adjacencies: **n\*(n-1)/2** adjacencies.
- OSPF reduces overhead by electing:
  - **Designated Router (DR)** – central router for all link-state information.
    - Controls LSA flooding and database synchronization.
  - **Backup Designated Router (BDR)** – standby.

**DROTHERs** (other routers) form adjacencies only with DR/BDR.

---

### DR/BDR Election Details:

**Election Steps**:

1. Highest OSPF interface **priority** (1-255).
2. If tie ➔ Highest **router ID**.

**Important**:

- **Priority 0** ➔ Router cannot become DR/BDR.
- No preemption: New routers joining don't trigger a new election.

**Traffic Flow**:

- DROTHERs send LSAs only to DR/BDR, **not** directly to each other.

---

### Multicast Addresses Used:

- **224.0.0.5** ➔ All OSPF routers (for general messages).
- **224.0.0.6** ➔ Only DR/BDR (for updates from DROTHERs).

---

### Why Priorities Matter:

- If you want to manually control who becomes DR (example: most powerful router with best hardware), set higher OSPF priorities.

---

## 2.4. Modify Single-Area OSPFv2 (Advanced OSPF Tweaks)

---

### OSPF Metric (Cost):

**Purpose**:

- OSPF uses **cost** to select the shortest/best path to a destination.
- **Lower cost = better path**.

**Cost Calculation**:

- Default formula:
  ```
  Cost = 100,000,000 / Interface Bandwidth (bps)
  ```

**Issues**:

- Gigabit and FastEthernet links have **same cost** under default calculation (1).
- Leads to inaccurate path selection in modern high-speed networks.

---

### Solutions:

1. **Adjust reference bandwidth** (`auto-cost reference-bandwidth`):

   - Example: For Gigabit Ethernet, set to 1000 Mbps.
   - Must be configured consistently across all routers!

2. **Manually set OSPF cost per interface** (`ip ospf cost <value>`):
   - Useful for fine-tuning paths manually.
   - Good for influencing backup vs primary routes.

---

### OSPF Cost Accumulation:

- Total cost = Sum of costs along the path.
- E.g., R1 to R2 over a 1Gbps link (cost 1) and a loopback (cost 1) ➔ Total cost = 2.

**Verification**: Use `show ip route` to see OSPF path metrics.

---

### Failover Behavior:

- If a link goes down:
  - Routers recompute SPF.
  - New shortest path based on updated costs is selected immediately.

---

### OSPF Timers:

| Timer       | Default Value  | Purpose                        |
| ----------- | -------------- | ------------------------------ |
| Hello Timer | 10s            | Interval between Hello packets |
| Dead Timer  | 40s (4x Hello) | Time to declare neighbor down  |

**Important**:

- **Timers must match** between routers to form an adjacency.
- Can be manually tuned for faster convergence but at the cost of more overhead.

---

## 2.5. Default Route Propagation

**Steps**:

1. Create a default static route
2. Use `default-information originate` to inject it into OSPF.
3. Routers learn default route marked as `O*E2`.

**Real-world Example**:

- Used by routers that connect internal OSPF domains to the Internet.

---

## 2.6. Verifying Single-Area OSPFv2

---

### Key Commands:

| Command                        | Purpose                                 |
| ------------------------------ | --------------------------------------- |
| `show ip interface brief`      | See interface statuses and IP addresses |
| `show ip route`                | Check routing table for OSPF routes     |
| `show ip ospf neighbor`        | Verify adjacencies                      |
| `show ip protocols`            | See OSPF process details                |
| `show ip ospf`                 | See OSPF areas and SPF calculation info |
| `show ip ospf interface`       | Detailed OSPF info per interface        |
| `show ip ospf interface brief` | Summary of OSPF interfaces              |

---

### Neighbor Adjacency States:

| State            | Meaning                                                        |
| ---------------- | -------------------------------------------------------------- |
| FULL             | Full adjacency established; routers are synchronized.          |
| 2-WAY            | Hello packets received, bidirectional communication confirmed. |
| EXSTART/EXCHANGE | Database description being exchanged.                          |
| LOADING          | LSRs (Link State Requests) being sent.                         |
| INIT/DOWN        | Early stages; adjacency not yet formed.                        |

**Important**:

- If stuck in EXSTART, EXCHANGE, LOADING = troubleshooting needed.
- INIT/DOWN means routers are not properly communicating yet.

---

# Tips

- Always configure **loopbacks** and **router IDs manually**.
- Always adjust **reference bandwidth** in modern networks.
- Always control **DR/BDR elections** via **priority settings** on critical routers.
- Use **passive interfaces** on user-facing LANs.
- Match **Hello and Dead timers** exactly between routers.
- Know how to spot **adjacency problems** quickly (subnet mismatch, timer mismatch, network type mismatch).
