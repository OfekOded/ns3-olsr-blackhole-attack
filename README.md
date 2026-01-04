# 🛰️ NS-3 OLSR Blackhole Attack Implementation

![NS-3 Version](https://img.shields.io/badge/ns--3-3.40%2B-blue.svg)
![Protocol](https://img.shields.io/badge/Protocol-OLSR-green.svg)
![Attack Type](https://img.shields.io/badge/Attack-Blackhole-red.svg)

---

## 📌 Project Overview

This repository contains a modified implementation of the **OLSR (Optimized Link State Routing)** protocol for the **NS-3 Network Simulator**.

The project focuses on **Network Layer Security** within **Mobile Ad-hoc Networks (MANETs)**. It implements a stealthy **Blackhole Attack** directly within the protocol's source code, allowing a specific node to manipulate network topology and silently discard traffic.

This implementation serves as a research tool for analyzing protocol vulnerabilities, evaluating the impact of insider threats, and testing Intrusion Detection Systems (IDS).

---

## 🏴‍☠️ Attack Mechanism (Theoretical Background)

A Blackhole Attack is a Denial-of-Service (DoS) attack where a malicious node uses the routing protocol to advertise itself as having the shortest path to the destination. Once the traffic is routed through it, the packets are dropped.

This implementation modifies both the **Control Plane** (to attract traffic) and the **Data Plane** (to destroy traffic).

### 1. Willingness Manipulation

In OLSR, nodes select **MPRs (Multi-Point Relays)** to forward broadcast messages.

- **Modification:**  
  The malicious node sets its `Willingness` field to `WILL_ALWAYS` (value 7).

- **Impact:**  
  According to RFC 3626, neighbors are forced to select this node as their MPR, ensuring it becomes a central hub for routing traffic.

---

### 2. Topology Poisoning (ANSN)

Topology Control (TC) messages carry the **Advertised Neighbor Sequence Number (ANSN)**.

- **Modification:**  
  The attacker artificially increments the ANSN by `+200` (modulo sequence limit) before sending updates.

- **Impact:**  
  The network interprets the attacker's information as newer or fresher than legitimate updates, causing valid routes to be overwritten by the malicious path.

---

### 3. Link Spoofing

- **Modification:**  
  The node generates fake `HELLO` messages claiming to have symmetric links with non-existent (phantom) neighbors.

- **Impact:**  
  This artificially increases the node's degree (connectivity), making it appear as a highly connected hub, further incentivizing shortest-path algorithms (like Dijkstra) to route traffic through it.

---

### 4. Silent Packet Drop (Blackhole)

- **Modification:**  
  In the packet forwarding logic (`RouteInput`), data packets destined for other nodes are intercepted.

- **Impact:**  
  The protocol signals successful processing (`return true`) but never invokes the forwarding callback. The packets are deleted from memory without generating ICMP error messages, making the packet loss difficult to trace.

---

## 📂 Repository Structure

The file structure mirrors the standard NS-3 source tree for easy integration.

```tree
├── src/
│   └── olsr/
│       └── model/
│           ├── olsr-routing-protocol.cc  # Core logic (Attack implementation)
│           └── olsr-routing-protocol.h   # Header file (Attributes & Definitions)
└── README.md
File Descriptions
olsr-routing-protocol.h
Defines the new IsMalicious and SpoofedLinksCount attributes.

olsr-routing-protocol.cc
Implements the logic for SendHello (Spoofing & Willingness), SendTc (ANSN poisoning), and RouteInput (Packet Drop).

🛠️ Installation & Integration
This project is not a standalone application; it is a patch for the NS-3 Simulator source code.

Prerequisites
A working installation of NS-3
(Recommended: ns-3-dev or ns-3.35+)

Git

Step 1: Clone the Repository
bash
Copy code
git clone https://github.com/YourUsername/ns3-olsr-blackhole-attack.git
Step 2: Backup Original Files
bash
Copy code
cd ~/workspace/ns-3-dev/src/olsr/model/
mv olsr-routing-protocol.cc olsr-routing-protocol.cc.bak
mv olsr-routing-protocol.h  olsr-routing-protocol.h.bak
Step 3: Apply the Patch
bash
Copy code
cp /path/to/ns3-olsr-blackhole-attack/src/olsr/model/* \
   ~/workspace/ns-3-dev/src/olsr/model/
Step 4: Recompile NS-3
bash
Copy code
cd ~/workspace/ns-3-dev/
./ns3 build
Once the build finishes successfully, your NS-3 environment is ready to simulate Blackhole attacks using the standard OLSR helper.