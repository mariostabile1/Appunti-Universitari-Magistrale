## Ethernet as the de facto standard

The starting point is recognising the current role of the network switch: it is no longer simply a way to connect computers to the Internet, but is the mechanism through which computational elements are aggregated into larger blocks. In this context, the dominant traffic in data centers is the so-called **east-west traffic** — that is, traffic between servers within the same data center — rather than the traditional north-south traffic towards the outside.

Ethernet became the standard for local connection not because it was technically the best, but because it was the most widely adopted. Mass adoption generated economies of scale that lowered hardware costs and stimulated the development of an enormously rich software ecosystem. Originally Ethernet was designed for a single shared coaxial cable — a physical bus — but this physical architecture no longer exists: today it is emulated virtually, and all the software infrastructure built on top of it is the real reason we continue to use it.

> [!tip] Key intuition
> 
> Ethernet did not win for absolute technical merits, but through network effects: more users → higher volumes → lower costs → even more users. This dynamic repeats itself in many computing technologies.

When requirements change, the standard is extended rather than replaced. VLANs are the most important example: they add a tag to the Ethernet header to allow logical network segmentation.

---

## MTU and packet size

> [!definition] MTU — Maximum Transmission Unit
> 
> The maximum size of a frame (packet) that a network device is configured to accept. The default value in Ethernet is **1.5 KB** (more precisely 1500 bytes).

Knowing how large a network packet is is fundamental to understanding performance. Every network processing unit implements a loop of the type _"for each packet, do something"_: if packets are many and small, the number of iterations grows, increasing CPU load. If packets are large, the number of iterations decreases but more memory is required for buffers.

This trade-off has a practical consequence: **Jumbo frames**. In data centers it is possible to configure switches to accept frames of **9 KB** (a factor of 6 compared to the default). This was introduced when Ethernet began to be used for storage traffic, where transferring large amounts of data with low overhead is critical.

> [!warning] Caution: MTU consistency in the network
> 
> In a multi-switch network (e.g. a three-tier topology), all devices along a path must have the same MTU configured. If a 9 KB frame arrives on a link with MTU 1.5 KB, the receiver must fragment it into 6 smaller packets — negating the advantage of the jumbo frame and adding unnecessary work. MTU is one of the most underestimated configuration parameters and a source of difficult-to-diagnose problems.

---

## Control plane, CLI and command line philosophy

Parameters such as MTU are configured through the switch's **control plane**, typically via a **CLI** (Command Line Interface). The CLI of network switches has a modal structure: one enters a context (e.g. `vlan 3`), issues specific commands, then exits. This philosophy has spread well beyond networking: Linux's `ip` command, Windows' `netsh`, and even `docker` and other modern tools adopt the same paradigm of _verb + subcommand_.

> [!note] Philosophical note: Unix vs modern CLI
> 
> The traditional Unix philosophy envisaged small, atomic commands (`cp`, `ls`, `sort`) composed through pipes to build complex behaviours. The modern philosophy, probably born with Docker, prefers a single "fat" command with many subcommands (`docker ps`, `docker image`, `docker exec`). This reduces composability but simplifies management and versioning when the number of resources to manage is very high.

---

## VLAN: logical segmentation of the broadcast domain

> [!definition] VLAN — Virtual LAN
> 
> A VLAN is a tag inserted in the Ethernet header that identifies the **broadcast domain** to which a frame belongs. It allows a physical network to be logically divided into multiple separate networks, without modifying the cabling.

The mechanism is simple: when an untagged frame arrives on a port configured as _untagged_ for a certain VLAN, the switch rewrites it by adding the corresponding VLAN ID. All frames inside the switch are then treated with their VLAN ID, guaranteeing isolation between different broadcast domains.

### Access and trunk ports

When a port is associated with a single VLAN ID with untagged traffic, it is called a port in **access mode**: typically used to connect a single host that does not need to worry about tagging. When instead a port is associated with multiple VLAN IDs (with tagged traffic), it is called a port in **trunk mode**: used for inter-switch connections, where it is necessary to carry the traffic of multiple broadcast domains on the same physical cable.

> [!example] Practical example
> 
> A data center has two racks connected by a single cable between their respective leaf switches. That cable must carry VLAN 2 and VLAN 3 traffic simultaneously. The port on both switches is configured as trunk with VLAN 2 and 3 tagged. In this way a separate cable for each VLAN is not required.

### VLANs in spine-leaf topology

In a spine-leaf architecture, the ports on leaf switches that connect towards the spine are configured with all VLAN IDs in use. Every frame arriving on a leaf tagged with a certain VLAN ID is forwarded towards the spine only if the uplink port includes that VLAN ID in the trunk. This allows a logically rich network on a very regular physical topology.

> [!warning] Fundamental limitation of VLANs
> 
> The 802.1Q standard supports a maximum of **4096 VLAN IDs** (12 bits). For a public cloud with millions of tenants, this number is absolutely insufficient.

---

## VXLAN and overlay networks

The 4096 VLAN limit, combined with the difficulty of manually managing the configuration of every switch, led to the development of **VXLAN** (_Virtual Extensible LAN_).

> [!definition] VXLAN
> 
> VXLAN is an **overlay network** protocol: it encapsulates layer 2 (Ethernet) traffic inside layer 3 (IP) UDP packets. This allows arbitrary layer 2 networks to be simulated on an existing layer 3 infrastructure.

### Why working at layer 3 is advantageous

Operating at the IP level (layer 3) offers critical advantages over pure layer 2:

- **Much simpler debugging**: at layer 3 every packet has well-defined source and destination; it is possible to trace the path and isolate problems. At layer 2, a broadcast loop can bring down entire broadcast domains without it being easy to understand where the problem is.
- **Scalability**: VXLAN network identifiers are 24 bits wide, meaning approximately **16 million** virtual segments — orders of magnitude more than the 4096 VLANs.
- **Automation**: virtual segments are created and managed via software, without manual intervention on every physical switch.

> [!example] Real example: University of Pisa
> 
> The university's core network uses VXLAN to connect the different departments. When there was an incident at the engineering faculty, it was necessary to quickly restore connectivity by creating temporary layer 2 links. This introduced a loop that caused service interruption. The lesson learned: extended layer 2 networks are difficult to debug; working at layer 3 with VXLAN drastically reduces this risk.

### VXLAN architecture in production

In Azure (and similar providers), each data center rack is a layer 2 domain. All traffic exiting the rack is immediately encapsulated in layer 3. This makes troubleshooting much more effective: every flow has a unique IP identifier and can be traced independently.

> [!abstract] VLAN vs VXLAN summary
> 
> VLANs segment a physical broadcast domain using tags in the Ethernet header (layer 2). VXLAN creates virtual layer 2 networks by encapsulating traffic in UDP/IP (layer 3). VXLAN requires a control plane: this is provided by **EVPN** (_Ethernet VPN_), the protocol that manages the distribution of reachability information between VXLAN nodes.

---

## Protocols implemented by a switch

A modern switch is much more than a simple forwarding device. It implements dozens of protocols, including:

**Switching and VLAN:** IEEE 802.1Q (VLAN tagging), 802.1ad (Q-in-Q, for encapsulating VLANs inside other VLANs), Spanning Tree Protocol and variants (to prevent loops).

**Quality of Service:** ETS (_Enhanced Transmission Selection_) for flow control, PFC (_Priority Flow Control_) for traffic prioritisation. These standards, born for storage traffic (where disks cannot wait), allow bandwidth to be divided among up to 16 traffic classes with different priorities.

> [!note] The myth of Ethernet without QoS
> 
> It is often said that Ethernet does not support Quality of Service. This is not true: the pure IEEE 802.3 standard has no flow control, but the ETS and PFC extensions, present in all modern data center switches, provide exactly this. What is "slow" is not Ethernet, but TCP.

**Overlay and virtualisation:** VXLAN, EVPN, MAC-in-UDP for layer 2 tunnelling over layer 3.

**Routing:** OSPF, BGP, ARP, multicast protocols.

**Management:** SNMP for monitoring, syslog, NetFlow for traffic analysis, LLDP/CDP for topology discovery.

---

## OpenFlow and Software Defined Networking

### The networking research problem

Around 2008, Stanford researchers identified a problem: network administrators did not allow experimentation on real switches because they were too critical. Networking research was blocked.

### The flow table

To understand OpenFlow it is necessary to understand how a switch works internally: modern devices maintain a **flow table**, a data structure that associates traffic patterns with actions. Rather than re-executing all the configuration logic for every packet, the switch builds rules of the type:

> If (source MAC = X) AND (destination IP = Y) AND (VLAN = Z) → output port 2

Once a flow is known, this rule is executed directly in hardware, without recalculating everything from scratch. The flow table also includes counters, useful for debugging and monitoring.

> [!definition] OpenFlow
> 
> OpenFlow is a **standard API** that allows an external controller to read and modify the flow table of a switch. The controller — a software process on a normal server — can add, modify or remove rules in response to network events.

### Practical applications

**Firewall sandwich:** a Layer 7 firewall is inserted between two switches. The first switch redirects every new flow through the firewall; when the firewall approves the flow, the OpenFlow controller updates the switch's flow table to allow subsequent packets of the same flow to bypass the firewall. Result: only the beginning of the flow is inspected, the rest transits at full speed.

**Mirroring and honeypot:** OpenFlow allows selectively copying traffic to a monitoring port (for an IDS system) or redirecting suspicious traffic to a honeypot — a decoy system that simulates a real target to capture and study attackers.

> [!note] Current state of OpenFlow
> 
> Born with great ambitions of "open networking", 15 years later OpenFlow is primarily a research tool and is used in specific cases. Most professionals never use it. It is nonetheless important to know of its existence and principle.

---

## Latency in Ethernet

Bandwidth and latency are the two fundamental parameters governing network performance. Latency is measured in several ways:

- **Serialisation latency**: time to put bits on the cable. With 1 Gigabit it was ~12 µs; with modern technologies we are in the order of **300 ns**.
- **Propagation latency**: time for signal propagation along the cable. At 10 metres it is ~50 ns, negligible in most cases.
- **Switch latency** (cut-through): time to traverse a switch. Broadcom Trident/Tomahawk chipsets are typically between **400 and 700 ns**.
- **End-to-end with TCP**: the TCP protocol adds framing overhead, congestion windows, ACKs, etc. Typical latency of a TCP stream is about **70 µs** — orders of magnitude greater than the hardware alone.

> [!tip] Key intuition
> 
> Ethernet itself is **low latency**. TCP is not. When one hears "Ethernet is not suitable for HPC because it has too much latency", the problem is almost always the TCP protocol on top, not the physical medium.

---

## InfiniBand and RDMA: introduction

### Why InfiniBand

When 1 Gigabit Ethernet had latency of 12 µs, it was clearly inadequate for HPC (High Performance Computing) applications where the computational pattern is _compute → communicate → compute → communicate_ in very tight cycles. InfiniBand was born to meet this need.

> [!definition] InfiniBand
> 
> InfiniBand is an interconnection standard designed for HPC, with **1-2 µs** end-to-end latency (compare with the 70 µs of TCP over Ethernet). The MTU of an InfiniBand packet is **2 GB** — enormously larger compared to the 9 KB of Ethernet jumbo frames. Many functions that in Ethernet are implemented in software are implemented directly in silicon.

InfiniBand was long dominated by **Mellanox**, an Israeli company acquired by **Nvidia** in 2020. Nvidia now uses this technology for high-speed connection between GPUs in AI training clusters (NVLink/NVSwitch fabric).

### RDMA and RoCE

> [!definition] RDMA — Remote Direct Memory Access
> 
> RDMA allows a network host to write directly into the memory of another host **without involving the destination CPU**. Analogous to local DMA (the mechanism by which a disk writes to RAM without engaging the CPU), but across the network.

This drastically eliminates software latency: no interrupt, no context switch, no data copy. The resulting latency is comparable to that of the network hardware.

Since InfiniBand was expensive and required dedicated hardware, Mellanox proposed **RoCE** (_RDMA over Converged Ethernet_):

> [!definition] RoCE — RDMA over Converged Ethernet
> 
> RoCE implements RDMA over Ethernet, exploiting the QoS extensions (ETS and PFC) to guarantee the necessary reliability. It is natively supported on Linux and Windows, and is today the standard for HPC over Ethernet.

> [!abstract] Where we are today
> 
> InfiniBand remains in use in traditional HPC clusters for its latency and optimised topology (fat tree without oversubscription). RoCE/RDMA has eroded part of its advantage by bringing similar latencies to Ethernet. In the next lecture the **fat tree** (_full fat tree_) topology used in InfiniBand will be examined in more detail, which guarantees constant bandwidth regardless of the number of hops in the network.

---

## Roadmap of upcoming lectures

After InfiniBand and HPC topologies, the course will continue with:

- Storage, drives and redundancy (RAID and distributed storage)
- Climbing up the stack towards virtualisation and hypervisors
- Network security and Layer 7 firewalls

> [!question] Possible exam questions
> 
> - Why is Ethernet the de facto standard for LANs? What economic and technological factors determined its success?
> - What is MTU? What problems can an inconsistent MTU configuration cause in a multi-switch network?
> - What is the difference between an access port and a trunk port in a VLAN configuration?
> - Why is VXLAN preferable to VLANs in a cloud infrastructure? What limitations of VLANs does it resolve?
> - What is OpenFlow? Describe a practical application (e.g. the "firewall sandwich").
> - Why is TCP latency much greater than the hardware latency of Ethernet? What is the important distinction to make?
> - What is RDMA and what distinguishes it from normal TCP communication? Why is it important for HPC?
> - What is RoCE and in what context was it introduced?
