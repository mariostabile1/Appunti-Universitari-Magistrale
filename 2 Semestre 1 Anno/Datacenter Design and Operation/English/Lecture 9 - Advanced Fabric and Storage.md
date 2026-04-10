
# Advanced Fabric and Introduction to Storage

## Workloads and Datacenter Architecture

Before discussing high-performance fabrics, it is necessary to understand *why* they exist. The choice of a datacenter's network architecture — bandwidth, latency, topology — is not universal: it is driven by the type of workload the system must execute. The lecturer distinguishes four fundamental categories.

**HPC** (*High Performance Computing*) is the oldest category: these are computationally enormous problems — physical simulations, fluid dynamics, seismic modelling — that a single computer cannot solve in useful time. The solution is to decompose the problem into sub-problems assigned to homogeneous nodes (all with the same resources), which collaborate by communicating frequently. The classic example is ENI simulating the propagation of seismic waves in the subsoil via finite elements to identify oil deposits without drilling: the result arrives in six months using thousands of CPUs, while a single computer would require centuries.

**Cloud** is the inverse model: here the problem is small — a single web application, a microservice — and the server is powerful enough to host dozens or hundreds of virtual machines simultaneously. The relationship is one-to-many: one computer, many problems. This model was possible because for twenty years computing power grew much faster than software demand.

**Big Data** is oriented towards processing enormous quantities of data, typically with mechanical disks: storage access latency (milliseconds) is so dominant that network latency becomes irrelevant. It makes no sense to optimise the network if the bottleneck is the disk.

**AI/ML** is the category that has recently broken the cloud equilibrium: these are workloads that, like HPC, require one or more dedicated servers for a single computation. The lecturer distinguishes two sub-categories with very different requirements — **training** and **inference** — and notes that even modern laptops now mount an NPU (*Neural Processing Unit*) optimised for local inference.

> [!tip] Key intuition
>
> Network latency only matters when computation is *memory-bound* and *synchronised*. In HPC and AI training every node must wait for the others at each step: even 2 ms of network latency multiplied by millions of iterations becomes the bottleneck. In Big Data the disk is so slow (ms) that the network ceases to matter.

---

## InfiniBand

### Historical context and motivation

When InfiniBand was introduced, Ethernet was still at 1 Gigabit per second. To aggregate 5 Gbps on a server, 4-5 Ethernet cables were used. InfiniBand offered ~80 Gbps in a single cable — hence the name, which evoked *infinite bandwidth*. Today the comparison with Ethernet is much less clear-cut from a bandwidth perspective, but InfiniBand maintains a significant advantage in **latency**.

> [!definition] InfiniBand
>
> High-performance network fabric designed for HPC environments. Physically similar to Ethernet (SFP/QSFP connectors), radically different at the protocol level. Originally developed by Mellanox, now owned by NVIDIA.

### Protocol characteristics

The crucial difference from Ethernet is not in the cables, but in the protocol:

- **Packets up to 2 GB**: from the host software's perspective, a 2-gigabyte packet can be sent as an atomic unit. The fabric handles segmentation and reassembly internally, but the application does not need to worry about it.
- **16 priority classes** (native *QoS*): traffic can be classified with fine granularity.
- **IP over IB**: there is an adapter for running TCP/IP over InfiniBand, useful for *north-south* communication with the outside, but nobody uses it for internal cluster communications.

The typical topology of an InfiniBand cluster is: **east-west over InfiniBand** (communication between cluster nodes), **north-south over Ethernet** (communication with the outside). The cluster head node (*head node*) is the only one connected to both networks.

An InfiniBand switch can reach 128 ports in a single chassis: this allows a completely flat interconnect to be implemented in hardware with guaranteed latency below 3 microseconds between any pair of ports.

### Top500: the state of the art

The *Top500* is the ranking of the world's 500 most powerful supercomputers. All systems at the top use InfiniBand or Slingshot. The lecturer cites as examples:

| System | Power | Notes |
|---------|---------|------|
| Frontier | ~29 MW, ~11M cores | AMD + Instinct GPU, Slingshot (HPE) |
| Eagle | — | Microsoft Azure, NVIDIA H100, InfiniBand — reference architecture for AI |
| Leonardo | — | Italy, top 10 |
| Jupiter | — | Germany, first European exascale system |

**Slingshot** (HPE/Cray) is Ethernet "on steroids": Ethernet-compatible at the physical level, but with protocol optimisations for low latency. **InfiniBand** (NVIDIA) is the native protocol for HPC. The two dominate the supercomputer segment.

---

## MPI and RDMA

### MPI: Message Passing Interface

Before discussing RDMA, it is fundamental to understand the HPC software context. Almost all distributed HPC code is written using **MPI** (*Message Passing Interface*), a C library of ~35 years that expresses distributed computation as message exchange between nodes identified by a sequential number (not by network addresses).

> [!tip] Key intuition
>
> MPI completely abstracts the underlying fabric. The application sends "a message to node 4" and MPI handles translating this into calls to the specific network primitives of the available fabric — InfiniBand, Slingshot, Ethernet. This is the reason why the HPC community has been able to change fabrics multiple times without rewriting the application code.

### RDMA: Remote Direct Memory Access

**RDMA** extends the concept of DMA (*Direct Memory Access*) — the mechanism that allows a peripheral to read/write directly in RAM without involving the CPU — across the network.

The mechanism exploits *converged NIC* network cards, connected to the PCIe bus: these cards can read and write in the RAM of the remote server directly, without going through the destination's operating system.

> [!warning] Security implications
>
> RDMA bypasses the operating system: the network card has direct access to memory. This introduces non-trivial security risks that must be considered in the system design.

RDMA has become the *de facto* standard for low-latency communication in distributed environments. Since 2016, Windows also implements **RoCE** (*RDMA over Converged Ethernet*), bringing RDMA to standard Ethernet as well.

### Latency comparison

| Technology | Typical latency |
|-----------|---------------|
| Native InfiniBand (RDMA) | 1–3 μs |
| RoCE (RDMA over Converged Ethernet) | 1.3–5 μs |
| IP over InfiniBand | 5–15 μs |
| TCP/IP over Ethernet | 20–70+ μs |

The most significant datum: adding the IP protocol over InfiniBand **multiplies latency by 4-5x** compared to native RDMA. TCP/IP is even worse. This explains why for HPC nobody uses TCP/IP internal to the cluster.

> [!note] The inventor of TCP/IP and buffers
>
> The lecturer cites an anecdote: one of the authors of TCP/IP, analysing the slowness of his own home network, discovered that the problem was the buffering introduced in every switch and router. TCP/IP had been designed to be *adaptive* and without buffers: the accumulation of buffers in the modern network distorts its behaviour. The author commented that, if he were to design the protocol today, he would do it differently.

---

## Overbooking and Full Fat Tree Topology

### Overbooking

In any switched network the concept of **overbooking** exists: the aggregate traffic generatable by access nodes exceeds the capacity of the link towards the upper layer.

> [!example] Concrete example
>
> A switch with 48 ports at 25 Gbps and 6 uplink ports at 100 Gbps. Maximum east-west traffic: 48 × 25 = 1,200 Gbps. Total uplink: 6 × 100 = 600 Gbps. The overbooking ratio is **2:1**: the nodes can generate double what the fabric can transmit towards the spine. This is normal and expected — the network adapts.

### Full Fat Tree

For HPC applications where non-blocking latency and throughput are requirements, a specific topology exists: the **Full Fat Tree**.

> [!definition] Full Fat Tree
>
> Network tree topology in which the uplink bandwidth of each node equals the *sum* of the bandwidths of the links in the subtree it connects. The result is a **non-blocking** network: each node can transmit at full speed to any other node without contention.

![Full Fat Tree topology: non-blocking tree with uplink equal to sum of downlinks](https://commons.wikimedia.org/wiki/Special:FilePath/Fat-tree.svg)
*Source: Wikimedia Commons — Fat Tree topology: each intermediate node has an uplink bandwidth equal to the sum of the bandwidths of the links towards the leaves. No chokepoint in the network.*

In practice, with modern link aggregation, it is implemented by summing multiple standard cables into a logical link. The rule is simple: if half the ports serve the servers (east-west) and half go towards the spine (north-south), the bandwidth of both sides is identical — no overbooking.

The Full Fat Tree has a high cost (switches at higher layers must handle increasing bandwidths), but it is the topology used for supercomputers and HPC clusters where non-blocking throughput is non-negotiable.

> [!note] Fat Tree vs Spine-and-Leaf
>
> The Spine-and-Leaf architecture seen in previous lectures *resembles* the Fat Tree, but typically introduces overbooking. The Fat Tree is the non-blocking version of Spine-and-Leaf, applied when the cost is justified by the required performance.

---

## Fibre Channel: the Storage Fabric

Beyond Ethernet and InfiniBand, in datacenters there exists a third type of fabric dedicated to storage: **Fibre Channel**.

> [!definition] Fibre Channel
>
> Optical network fabric dedicated to host-storage connections. Physically similar to Ethernet and InfiniBand (optical fibre, analogous connectors), but with completely different protocols oriented towards storage access.

The typical architecture is: server with **HBA** card (*Host-Based Adapter*, the Fibre Channel equivalent of a NIC) → Fibre Channel fabric (FC switches) → storage array (a system with many internal disks). The HBA emulates a local disk, but the data resides on the storage array.

The protocol transported by Fibre Channel is **SCSI** (or more precisely FCP — Fibre Channel Protocol encapsulating SCSI), which leads to an important historical curiosity.

**FCoE** (*Fibre Channel over Ethernet*) also exists, conceived as a transition mechanism to avoid maintaining a separate FC network. The lecturer defines it as "terrible" — it does not work well and has been substantially abandoned.

Fibre Channel is today **legacy**: many existing installations still use it, but new projects prefer Ethernet or InfiniBand even for storage.

---

## The History of Storage Protocols

To understand why Fibre Channel still uses SCSI, the history of disk access protocols must be retraced.

### SCSI: 1979

**SCSI** (*Small Computer System Interface*) was born in 1979 as a parallel bus (multi-connector flat cable) for connecting multiple storage devices to a single controller, sharing the cost of the controller among multiple disks. The master/slave logic allowed managing 2 disks per controller (ATA) or many more (SCSI).

Although the physical technology has changed radically, the **SCSI protocol has survived for 40 years** thanks to the principle of not having to rewrite the software: every new technology has implemented SCSI as an adapter over new physical media.

| Variant | Physical medium | Notes |
|---------|-------------|------|
| Parallel SCSI | Flat cable | Original 1979 |
| SAS | Serial | Serial Attached SCSI |
| iSCSI | Ethernet/TCP | SCSI over Internet — high overhead |
| FCP | Fibre Channel | SCSI over optical fibre |
| SRP | InfiniBand | SCSI over RDMA |

### ATA → SATA

In parallel with SCSI, the PC world developed **ATA** (cheaper) and then **SATA** (Serial ATA), dominant in consumer drives until a few years ago.

### NVMe: the revolution

**NVMe** (*Non-Volatile Memory Express*) breaks the paradigm: it is not a bus protocol, it is a protocol directly on top of **PCIe** (*PCI Express*). An NVMe SSD is not "a disk connected to a controller" — it is a PCIe card that behaves like any other peripheral in the system.

> [!tip] Why NVMe changed everything
>
> For decades software — databases, filesystems, operating systems, networks — was designed with the assumption that the disk was **slow** (milliseconds). With NVMe, latency drops to microseconds. The bottleneck shifts: SCSI, TCP/IP and other protocols that introduced negligible latency compared to a mechanical disk suddenly become the limiting factor. This required rethinking almost all storage software.

---

## Storage Hierarchy

The modern datacenter maintains a layered architecture, in which each level is a trade-off between speed, capacity, cost and durability.

| Level | Technology | Typical latency | Use |
|---------|-----------|---------------|-----|
| Primary | NVMe SSD | Microseconds | Active data, cache |
| Secondary | Mechanical HDDs | Milliseconds | Hot data, primary backup |
| Tertiary | Magnetic tapes | Seconds | Disaster recovery, archive |
| Future | Glass (Project Silica) | Seconds+ | Long-term storage |

### Magnetic tapes: why still in use

Tapes seem anachronistic, but have unique properties:
- They can be physically removed and stored in secure locations (resistant to physical attacks, EMP, disasters).
- They do not require power to preserve data.
- The cost per GB is much lower than disks.

Modern tape archives use **robots** that automatically manage the inventory, retrieval and insertion of cartridges into reading units. The appearance recalls science fiction, but it is well-established technology.

### The data durability problem

All storage media have a finite life. The lecturer introduces a little-known problem even with SSDs: every NAND block must be **rewritten every ~2 weeks** by an internal firmware thread on the drive, otherwise data corrupts due to charge dissipation.

> [!note] Project Silica (Microsoft)
>
> Microsoft is experimenting with writing data onto **glass** via laser pulses, with reading through optical microscopy and AI decoding. Data is estimated to last from a thousand to a million years, resistant to EMP and water. Current density is ~75 GB per piece of glass (expected to scale to TB), latency is in the order of seconds. Suitable only for long-term *cold storage*.

---

## Where We Are in the Course

> [!abstract] Upcoming topics
>
> The next lecture (**Lecture 10**) will deepen storage: what happened to server architecture with the advent of NVMe SSDs, how the memory hierarchy changed, and how software adapted. Server architectures will follow, then the software side of the datacenter.

> [!question] Possible exam questions
>
> - What are the four workload categories? How do they differ in terms of network requirements?
> - Why is latency critical for HPC but not for Big Data?
> - What is InfiniBand? How does it differ from Ethernet at the protocol level?
> - What is RDMA? How does it differ from MPI? What are the security implications?
> - Compare the latencies of native IB, RoCE, IP-over-IB and TCP/IP.
> - What is overbooking in a switched network? How is it calculated?
> - Define the Full Fat Tree topology and its fundamental property.
> - What is Fibre Channel and what is it used for? Why does it use the SCSI protocol?
> - Why did NVMe represent a break from previous storage protocols?
> - Describe the storage hierarchy in a modern datacenter and the rationale for each level.
