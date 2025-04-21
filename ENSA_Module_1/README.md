# ENSA Module 1: Single-Area OSPFv2 Concepts

## **1.1 OSPF Features and Characteristics**

### **Routing Protocol Overview**

- **Routing protocols** are used to find the best path for data packets dynamically.
- They fall into two broad categories:
  - **Interior Gateway Protocols (IGPs)**: Used **within** a single organization.
    - Examples: OSPF, RIP, EIGRP
  - **Exterior Gateway Protocols (EGPs)**: Used **between** organizations.
    - Only one main protocol: **BGP (Border Gateway Protocol)**.
      - Used to exchange routing information between different autonomous systems (ASes).

### **Types of Routing Protocol Algorithms**

- **Distance Vector**: Based on hop count (e.g., RIP).
- **Link-State**: Based on link cost and network topology (e.g., OSPF, IS-IS).
- **Path Vector**: Uses policies and path attributes (e.g., BGP).

---

### **OSPF Characteristics**

- Developed as an improvement over RIP.
- Based on **link-state technology**.
- Key Benefits:
  - **Faster convergence**
  - **Better scalability**
- OSPF divides the network into **areas** to reduce update traffic and complexity.

### **Understanding a Link**

- A **link** can be:
  - A router interface,
  - A network segment between two routers,
  - A **stub network** (e.g., a LAN with one router).

### **Link-State Information Includes:**

- Network prefix
- Prefix length
- Cost (used to calculate best paths)

---

### **Components of OSPF**

All routing protocols share 3 core components:

1. **Routing Messages** – exchanged information between routers.
2. **Data Structures** – store received data.
3. **Routing Algorithm** – calculates best paths based on the data structures.

OSPF uses 5 types of **packets** for communication:

- **Hello**: To discover and maintain neighbors.
- **Database Description (DBD)**: Summary of the LSDB. Contains LSA headers. Based on this information, routers decide which LSAs to exchange.
- **Link-State Request (LSR)**: Request for specific LSA details.
- **Link-State Update (LSU)**: Contains actual LSA data.
- **Link-State Acknowledgment (LSAck)**: Confirms receipt.

---

### **OSPF Databases**

1. **Neighbor Table / Adjacency Database**
   – who has a bidirectional connection with the router in question.
   - Unique for each router.
2. **Link-State Database (LSDB)** – topological view of the network.
   - All routers which are in the same area have the same LSDB.
   - Contains **Link-State Advertisements (LSAs)**, which describe the state of the links and their costs.
3. **Routing Table / Forwarding Database** – best paths calculated from LSDB.
   - Each router has its own unique routing table, which is built from the LSDB.

OSPF uses **Dijkstra’s SPF algorithm** to:

- Build a **SPF tree** with the router at the root.
- Determine the **shortest (lowest cost) paths** to all destinations.

### **Link-state routing steps**

1. **Discovery**: Routers discover neighbors using Hello packets.
2. **Exchange**: Routers exchange Link-State Advertisements (LSAs) to build a complete view of the network topology.
3. **Building the LSDB**: Each router builds its own Link-State Database (LSDB) based on received LSAs.
4. **SPF Calculation**: Each router runs the SPF algorithm on its LSDB to calculate the shortest path to each destination.
5. **Routing Table Creation**: The router creates its routing table based on the SPF calculation results.

---

### **Single-Area vs. Multiarea OSPF**

#### **Single-Area OSPF**

- All routers in one area, typically **area 0**.
- Simpler configuration and suitable for smaller networks.

#### **Multiarea OSPF**

- Divides the network into multiple areas connected through **backbone area (area 0)**.
- Benefits:
  - **Smaller routing tables**
  - **Less frequent SPF recalculations**
  - **Reduced LSA flooding**
- Introduces **Area Border Routers (ABRs)**.

---

### **OSPFv3 (for IPv6)**

- Equivalent to OSPFv2 but supports **IPv6**.
- Uses **separate processes** for IPv4 and IPv6.
- Works similarly, with Hello, DBD, LSR, LSU, and LSAck packets.
- **OSPF Address Families** (IPv4 and IPv6 in one process) are supported but **not part of this curriculum**.

---

## **1.2 OSPF Packets**

### **Packet Types:**

1. **Hello** – establishes and maintains neighbor relationships.
   - Advertise paramters routers must agree on to form adjacencies.
   - Elect Designated Router (DR) and Backup DR (BDR) on multiaccess networks.
2. **Database Description (DBD)** – lists known LSAs in summary.
3. **Link-State Request (LSR)** – requests specific LSA details.
4. **Link-State Update (LSU)** – contains one or more LSAs.
5. **Link-State Acknowledgment (LSAck)** – confirms receipt of LSAs.

### **LSU & LSA Relationship:**

- **LSUs** carry **LSAs**.
- There are 11 types of LSAs in OSPFv2.

---

## **1.3 OSPF Operation**

### **7 Operational States of OSPF:**

1. **Down** – No OSPF activity.
   - Router sends Hello packets to discover neighbors.
   - If no response, it remains in this state.
   - If a neighbor is detected, it moves to the Init state.
2. **Init** – Hello packets sent; router is waiting for a response.
   - Transition to **Two-Way** state after receiving a Hello packet from the neighbor.
3. **Two-Way** – Bidirectional communication established.
   - multiaccess networks: DR/BDR election occurs.
   - Transition to **ExStart** state.
4. **ExStart** – Routers decide which one starts DBD exchange.
   - Master/Slave relationship is established. (point-to-point links only).
   - The router with the higher Router ID becomes the master.
5. **Exchange** – Routers exchange DBD packets.
   - Routers send and receive DBDs.
   - If additional information is needed, they transition to the Loading state.
6. **Loading** – Routers send LSRs and receive LSUs.
   - SPF algorithm is run to calculate the best path.
7. **Full** – LSDBs are fully synchronized.
   - Adjacency is fully established.
   - Routers are in a stable state.

---

### **Establishing Neighbor Adjacencies**

- Routers send **Hello packets** with their **Router ID** (formatted like IPv4).
- Sent to **224.0.0.5** (All OSPF Routers multicast).
- If a new neighbor is detected, they attempt to form an adjacency.

---

### **Database Synchronization (Three Steps):**

For multiaccess networks, the process is as follows after the Two-Way state:

1. **Elect a master** – The router with the higher Router ID goes first.
2. **Exchange DBDs** – Routers send and acknowledge summaries.
3. **Request missing info** – Via LSRs, followed by full database sync.

Updates (LSUs) are sent:

- When a change is perceived (incremental updates)
- Every 30 minutes

---

### **Why DR/BDR are Needed**

In Ethernet/multiaccess networks:

- If every router talks to every other router, LSA flooding becomes overwhelming.
- Solution: Elect a **Designated Router (DR)** and **Backup DR (BDR)**.
  - DR is central hub for LSA distribution. Meaning that only the DR and BDR will send LSAs to all other routers. And the other routers will only send LSAs to the DR and BDR.
  - **DROTHERs** are regular routers not elected as DR/BDR.

| Role    | Forms Adjacency With |
| ------- | -------------------- |
| DR      | All DROTHER routers  |
| BDR     | All DROTHER routers  |
| DROTHER | Only DR and BDR      |

DROTHERs initiate the DBD process with the DR and BDR. But the DR / BDR takes the lead in the process.

So in a real OSPF multiaccess network:

- DR/BDR don’t start the DBD process.
- DROTHERs start it.
- DR/BDR take control of the exchange (Master role).
- DBDs are exchanged in both directions, but the flow is managed by the Master.

---

## Summary Takeaways:

- OSPF is a **link-state IGP** designed for scalability and fast convergence.
- **Single-area OSPF** is simple and ideal for smaller networks.
- Routers exchange **Hello** packets to form neighbor relationships.
- The **SPF algorithm** builds a network tree from which best paths are derived.
- **DR/BDR elections** help reduce overhead on multiaccess networks.
