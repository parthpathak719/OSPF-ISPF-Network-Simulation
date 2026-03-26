# 🌐 OSPF Packet Visualizer (iSPF vs Traditional OSPF)

This project demonstrates and compares **traditional OSPF (Open Shortest Path First)** with **iSPF (Incremental Shortest Path First)** using a realistic network simulation.

The goal is to show how **iSPF improves convergence time and reduces CPU usage** by updating only affected parts of the network instead of recomputing everything.

---

## 🚀 Features

- 🧠 Implementation of **OSPF vs iSPF behavior**
- 🔁 Simulation of **real network events**
  - Link failure
  - Router failure
  - Link cost change
- 📊 Performance comparison using:
  - Convergence time
  - CPU usage
- 🖥️ Realistic simulation using **Cisco IOSv routers**
- 🌐 Multi-area OSPF topology (Area 0, 1, 2)

---


---

## ⚙️ Tech Stack

- GNS3 (v2.2+)
- VMware Workstation Pro
- Cisco IOSv (15.9)
- Networking concepts (OSPF, SPF, iSPF)

---

## 🧠 Concepts Used

- OSPF (Open Shortest Path First)
- Dijkstra’s Algorithm (SPF)
- Incremental SPF (iSPF)
- Network convergence
- Link State Advertisements (LSA)

---

## 📊 Results Summary

| Scenario              | OSPF Time | iSPF Time | CPU Impact |
|----------------------|----------|----------|------------|
| Link Failure         | 10 ms    | 7 ms     | Lower in iSPF |
| Router Down          | 7 ms     | 4 ms     | Faster recovery |
| Link Cost Change     | 3 ms     | 3 ms     | No CPU spike in iSPF |

💡 iSPF avoids unnecessary recalculations → **faster + more efficient**

---

## 📁 Included Files

- GNS3 project file (`.gns3`)
- Router configurations
- Simulation results (CPU graphs, logs)

For detailed explanation refer:
- 📄 Report: :contentReference[oaicite:0]{index=0}  
- 📊 Presentation: :contentReference[oaicite:1]{index=1}  

---

## 🧪 How to Run the Simulation

### 1️⃣ Install Required Tools

- Install **GNS3**
- Install **VMware Workstation Pro**
- Import **GNS3 VM into VMware**

---

### 2️⃣ Setup Cisco IOSv

- Download Cisco IOSv image (15.9 recommended)
- Add it in GNS3:
  - Edit → Preferences → QEMU VMs
  - Import IOSv image

---

### 3️⃣ Open Project

- Open `ispf.gns3` file in GNS3
- Ensure all routers are properly mapped to IOSv image

---

### 4️⃣ Start the Network

- Click **Start All Devices**
- Open router consoles

---

### 5️⃣ Verify OSPF

Run on routers:

```bash
show ip ospf neighbor
show ip route
