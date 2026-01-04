# 🛰️ NS-3 OLSR Blackhole & Link Spoofing Attack Implementation

![NS-3 Version](https://img.shields.io/badge/ns--3-3.40%2B-blue.svg)
![Protocol](https://img.shields.io/badge/Protocol-OLSR-green.svg)
![Attack Type](https://img.shields.io/badge/Attack-Blackhole-red.svg)

---

## 📌 Project Overview

This repository contains a specialized implementation of the **Optimized Link State Routing (OLSR)** protocol for the **NS-3 Network Simulator**. It is designed for **Network Layer Security** research within **Mobile Ad-hoc Networks (MANETs)**.

Modified by **Oded Ofek (2025)**, this version introduces a sophisticated **Blackhole Attack** and **Link Spoofing** capabilities directly into the core protocol logic. By manipulating both the **Control Plane** and the **Data Plane**, a malicious node can effectively position itself as a central routing hub and silently discard network traffic.

---

## 🏴‍☠️ Attack Mechanisms

The implementation utilizes four distinct techniques to compromise the network topology and disrupt data delivery:

### 1. Willingness Manipulation (Control Plane)
In standard OLSR, nodes select **Multi-Point Relays (MPRs)** based on their advertised willingness to forward traffic.
* **Modification**: When the `IsMalicious` attribute is enabled, the node overrides its default willingness.
* **Implementation**: The node sets its `Willingness` field to `WILL_ALWAYS` (value 7) in all outgoing `HELLO` messages.
* **Impact**: According to RFC 3626, neighbors are forced to prioritize this node as an MPR, ensuring the attacker is included in nearly all routing paths.

### 2. Topology Poisoning via ANSN (Control Plane)
The **Advertised Neighbor Sequence Number (ANSN)** is used by nodes to verify the freshness of topology information.
* **Modification**: The attacker artificially manipulates the sequence number arithmetic.
* **Implementation**: In the `SendTc` function, the attacker increments the ANSN by a large offset (`+200`) before broadcasting updates.
* **Impact**: Neighboring nodes perceive the attacker's topology information as significantly newer than legitimate updates, causing valid routing table entries to be overwritten by the malicious path.

### 3. Link Spoofing (Control Plane)
* **Modification**: The node advertises symmetric links with non-existent (phantom) neighbors.
* **Implementation**: The node generates `HELLO` messages containing fake IP addresses (starting from `200.0.0.1` / `0xC8000001`) with a `SYM_LINK` / `SYM_NEIGH` status.
* **Impact**: This artificially inflates the node's degree of connectivity. Pathfinding algorithms like Dijkstra will perceive the attacker as the most efficient "short-cut" for traffic, attracting flows from across the network.

### 4. Silent Packet Drop / Blackhole (Data Plane)
* **Modification**: Interception and destruction of transit traffic.
* **Implementation**: Within the `RouteInput` function, if the node is malicious, it intercepts unicast packets destined for other nodes.
* **Impact**: The protocol returns `true` (signaling successful processing) but intentionally fails to invoke the `UnicastForwardCallback`. Packets are dropped from memory without generating ICMP error messages, making the attack difficult to detect through standard diagnostic tools.

---

## 📂 Repository Structure

The source files are organized to mirror the standard NS-3 module tree for seamless integration:

| File | Description |
| :--- | :--- |
| `olsr-routing-protocol.h` | Defines the `m_isMalicious` flag and `m_spoofedLinksCount` attribute. |
| `olsr-routing-protocol.cc` | Implements the attack logic in `SendHello`, `SendTc`, and `RouteInput`. |
| `README.md` | Comprehensive documentation of the implementation. |

---

## 🛠️ Installation & Integration

### Prerequisites
* A working installation of **NS-3** (v3.35 or higher recommended).
* Basic knowledge of compiling NS-3 using the `./ns3` build system.

### Step-by-Step Setup
1.  **Backup** your original OLSR files located in `src/olsr/model/`.
2.  **Copy** the modified `olsr-routing-protocol.h` and `olsr-routing-protocol.cc` from this repository into `src/olsr/model/`.
3.  **Rebuild** the simulator:
    ```bash
    ./ns3 build
    ```

---

## 🚀 Simulation Usage

You can dynamically enable or disable the attack behavior in your NS-3 simulation scripts using the attribute system:

```cpp
Ptr<Node> node = nodes.Get(5);
Ptr<olsr::RoutingProtocol> protocol = node->GetObject<olsr::RoutingProtocol>();

protocol->SetAttribute("IsMalicious", BooleanValue(true));
protocol->SetAttribute("SpoofedLinksCount", UintegerValue(15));

### Configurable Attributes
* **IsMalicious**: Boolean flag to toggle the Blackhole, ANSN poisoning, and Willingness manipulation.
* **SpoofedLinksCount**: The number of fake symmetric neighbors to advertise in HELLO messages.

### ⚠️ Research Disclaimer
This implementation is intended strictly for academic and research purposes, such as evaluating protocol vulnerabilities or testing Intrusion Detection Systems (IDS). Unauthorized use of these techniques in real-world environments is prohibited.

---

**Would you like me to create a C++ simulation script (scenario) that demonstrates this attack and calculates the resulting packet delivery ratio (PDR)?**