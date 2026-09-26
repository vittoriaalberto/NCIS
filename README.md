# Network Slicing in SDN with Ryu and Mininet  
### Course Project – Network and Cloud Infrastructures  
M.Sc. in Computer Engineering — University of Naples Federico II (A.Y. 2024/2025)

This repository contains the project developed for the *Network and Cloud Infrastructures* course.  
The work implements static, service-based, and dynamic network slicing in an SDN environment using Mininet as the network emulator and Ryu as the SDN controller. 
In addition, this project extends the base requirements with a real-time monitoring dashboard and a Machine Learning-based traffic classifier.

---
## Technologies Used
- Python 3
- Ryu SDN Framework (OpenFlow 1.3)
- Mininet
- Open vSwitch (OVS)
- Streamlit (for the dashboard)
- scikit-learn, pandas (for ML classification)
- iperf

---

## 📡 Architecture and Base Topology
The SDN network consists of **4 hosts** and **6 OpenFlow switches (OVS)** organized into two parallel paths:

```text
        H1 --+                              +-- H3
             S1 -- S2 -- S4 -- S6  (10 Mbps | Upper Slice - Video)
             S1 -- S3 -- S5 -- S6   (1 Mbps | Lower Slice - Data)
        H2 --+                              +-- H4

        Controller: Ryu (OpenFlow 1.3) -- Dynamic Slicing Enabled
```

A logical separation enforces isolation between the two slices:
- **Upper Slice:** H1 ↔ H3 (10 Mbps)
- **Lower Slice:** H2 ↔ H4 (1 Mbps)

The controller enforces isolation, MAC learning, and OpenFlow 1.3 flow modification.

---

## 🚀 Extensions Overview

The implementation focuses on three advanced extensions:

### 1. Real-time Dashboard
A live monitoring dashboard developed with Streamlit that provides real-time visualization of network statistics directly from the Ryu controller. 

**Modules:**
- **Live Monitor:** Real-time metrics for both slices (estimated bandwidth, cumulative volume), Dynamic Slicing status banner (Fast Lane ON/OFF), and area charts for traffic trends.
- **ML Analysis & Classification:** Deep dive into the K-Means unsupervised classification, showing cluster distribution and comparing L1 vs L2 accuracy.
- **Network Topology:** Interactive SVG visualization of the network with adaptive colors and switch details.

### 2. ML-based Traffic Classification
Traffic is classified using an unsupervised Machine Learning approach (K-Means algorithm).
Instead of relying on simple port numbers, the classifier uses packet statistics to dynamically group flows.

- **Level 1 - Port/MAC-based (Deterministic):** The controller uses eth_src/eth_dst to route flows. It acts as the ground truth.
- **Level 2 - K-Means (Unsupervised):** Analyzes `byte_count` and `packet_count` via StandardScaler. It has no knowledge of MACs, IPs, or ports, autonomously discovering high-bandwidth (video) and low-bandwidth (data) categories.

### 3. Dynamic Slicing
Non-video traffic from the Lower Slice is dynamically rerouted to the Upper Slice when video traffic is inactive, ensuring resource optimization while keeping video priority.
The controller periodically measures the bandwidth used by video flows:

```text
bandwidth_video = (delta_byte * 8) / (10s * 1_000_000)  ->  Mbps

if bandwidth_video < 0.5 Mbps:
    Move Lower Slice to Upper path (10 Mbps) -- Fast Lane active
else:
    Move Lower Slice back to Lower path (1 Mbps) -- Video priority
```

---
## How to Run

1. **Start the Mininet Topology:** `sudo python3 slicing_topo.py`
2. **Start the Ryu Controller:** `ryu-manager main_controller.py`
3. **Generate Traffic (Mininet CLI):** `source video_traffic.sh` and `source normal_traffic.sh`
4. **Launch the Dashboard:** `streamlit run app.py`
