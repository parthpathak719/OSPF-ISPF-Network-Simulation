# 🌐 OSPF Packet Visualizer - iSPF vs Traditional OSPF

This project simulates and compares **Traditional OSPF** (full Dijkstra recomputation) and **Incremental Shortest Path First (iSPF)** using real Cisco IOSv routers in GNS3.

The goal is to demonstrate how iSPF reduces CPU overhead and convergence time by recomputing only the **affected subtree** of the shortest-path tree instead of recalculating everything from scratch.

---

## 🚀 Features

- 🔁 Side-by-side simulation of Traditional OSPF and iSPF in GNS3
- 🖥️ Real Cisco IOSv 15.9 routers - not just Python scripts
- 📊 Performance comparison using **SPF Calculation Time** and **CPU Spike %**
- 🧪 Three real-world failure scenarios: link failure, router down, cost change
- 🏗️ Multi-area OSPF topology (Area 0, Area 1, Area 2)

---

## 🗂️ Topology

**GNS3 Network - 5 Routers, 3 Areas**

| Parameter | Value |
|---|---|
| Routers per topology | 5 (Cisco IOSv) |
| Total topologies | 2 (one OSPF, one iSPF) |
| Areas | Area 0 (backbone), Area 1, Area 2 |
| Simulator | GNS3 v2.2.54 |
| Router OS | Cisco IOSv 15.9(3)M8 |

---

## 🛠️ Tech Stack

- GNS3
- Cisco IOSv (v15.9)
- OSPF Process (IOS CLI)
- iSPF (`ispf` IOS command)

---

## 📈 Results

| Model | Scenario | SPF Time | CPU Spike |
|---|---|---|---|
| Traditional OSPF | Link Failure | 10 ms | ~30% |
| iSPF | Link Failure | 7 ms | ~20% |
| Traditional OSPF | Router Down | 7 ms | - |
| iSPF | Router Down | 4 ms | - |
| Traditional OSPF | Link Cost Change | 3 ms | Visible |
| iSPF | Link Cost Change | 3 ms | **None** |

iSPF outperformed Traditional OSPF in every scenario - most strikingly in the link cost change test, where iSPF detected the change was irrelevant to the best path and **skipped the SPF run entirely**.

---

## ⚙️ Simulation Walkthrough

### Prerequisites

| Tool | Version | Notes |
|---|---|---|
| [GNS3](https://www.gns3.com/software/download) | 2.2.54 | Desktop + GNS3 VM |
| GNS3 VM | Matching GNS3 version | Run in VMware / VirtualBox |
| Cisco IOSv image | `15.9(3)M8` | Requires a valid Cisco account |
| RAM | ≥ 8 GB | For running 10 router VMs simultaneously |

> **Note:** The Cisco IOSv `.qcow2` image is **not** included due to licensing. Obtain it via [Cisco DevNet](https://developer.cisco.com/) or a CML subscription.

---

### Step 1 - Install GNS3 and the GNS3 VM

Download both the **GNS3 desktop app** and the **GNS3 VM** from [gns3.com](https://www.gns3.com/software/download).

In GNS3 → **Edit → Preferences → GNS3 VM**, point it to your VMware/VirtualBox instance and verify the green "connected" status before proceeding.

---

### Step 2 - Add the Cisco IOSv Image

1. In GNS3 go to **Edit → Preferences → QEMU VMs → New**
2. Name it `IOSv` and upload your `vios-adventerprisek9-m.vmdk.SPA.159-3.M8` image
3. Set RAM to **512 MB**, adapters to **4**, adapter type to **e1000**
4. Under **Advanced**, enable *"Use as a linked base VM"* for faster cloning

---

### Step 3 - Import the Topology

```bash
git clone https://github.com/<your-username>/ospf-packet-visualizer.git
cd ospf-packet-visualizer
```

Then in GNS3: **File → Open Project** → select `ispf.gns3`

The project loads two parallel topologies side by side:

```
         [Area 0 - Backbone]
IOSv1 ──────── IOSv2 ──────── IOSv3
  |         (ABR)          (ABR)|
  | [Area 1]              [Area 2]
IOSv5 ──────────────────── IOSv4

(Topology B is identical with iSPF enabled)
```

---

### Step 4 - Configure the Routers

> Skip this step if you imported `ispf.gns3` - configs are pre-loaded.

**Topology A - Traditional OSPF** (example for IOSv1):

```
hostname IOSv1
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.0
 no shutdown
!
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 1
```

**Topology B - iSPF** (same config + one extra line):

```
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 1
 ispf
```

The single `ispf` command is all it takes to enable Incremental SPF on a Cisco router.

---

### Step 5 - Verify Full Convergence

Start all routers and wait ~60 seconds. Confirm all adjacencies are up before running any scenario:

```
IOSv1# show ip ospf neighbor
IOSv1# show ip route ospf
```

All routers should show **FULL** state in the neighbor table.

---

### Step 6 - Reset Stats Before Each Test

Run this on the router you'll be measuring **before each scenario**:

```
IOSv2# clear ip ospf process      ! type 'yes' to confirm
IOSv2# clear counters
```

---

### 🧪 Scenario 1: Single Link Failure

Shut down the link between two Area 1 routers on **both topologies simultaneously**.

**Topology A (OSPF) - IOSv2:**
```
IOSv2# conf t
IOSv2(config)# interface GigabitEthernet0/1
IOSv2(config-if)# shutdown
IOSv2(config-if)# end
```

**Topology B (iSPF) - IOSv7:** run the same `shutdown` command on the equivalent interface.

Then immediately [collect metrics](#step-7--collect-metrics).

---

### 🧪 Scenario 2: Router Down

Simulate a complete router failure in Area 1.

**Topology A:** Right-click **IOSv3** in GNS3 → **Stop**

**Topology B:** Right-click **IOSv8** in GNS3 → **Stop**

Collect metrics from the ABR routers (IOSv2 / IOSv7) immediately after.

---

### 🧪 Scenario 3: Link Cost Change

Increase the OSPF cost on one link from 1 to 200 to test if iSPF is smart enough to skip an irrelevant SPF run.

**Topology A — IOSv2:**
```
IOSv2# conf t
IOSv2(config)# interface GigabitEthernet0/1
IOSv2(config-if)# ip ospf cost 200
IOSv2(config-if)# end
```

**Topology B — IOSv7:** same command on the equivalent interface.

This is the most revealing scenario — traditional OSPF runs a full Dijkstra anyway, iSPF does not.

---

### Step 7 — Collect Metrics

Run both commands immediately after triggering each scenario event.

**CPU Usage:**
```
IOSv2# show processes cpu history
```
Look at the peak `CPU%` value in the last-60-seconds graph. A cluster of `*` characters indicates a spike from the SPF run.

**SPF Calculation Time:**
```
IOSv2# show ip ospf statistics
```
Record the **`Total`** time in milliseconds from the most recent SPF entry:
```
  Area 1: SPF algorithm executed 3 times
   Last execution took 0.007 seconds
