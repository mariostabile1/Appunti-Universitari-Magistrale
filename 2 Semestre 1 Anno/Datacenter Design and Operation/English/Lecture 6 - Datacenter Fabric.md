## Physical Cabling in the Data Center

### Why cabling is a design choice

When laying a cable in a data center, one is not performing a neutral mechanical operation: an architectural decision is being made. Physical cabling determines the geometry of communication between racks, and that geometry then becomes a hard limit on the performance of the applications running on top.

The reasons why cabling is relevant are essentially three.

**Signal loss.** Every connector introduced causes a small attenuation. Inside the data center, where paths are short, this loss is negligible and does not represent a practical constraint.

**Number of hops.** Every switch a packet traverses introduces latency. A single modern Ethernet switch introduces approximately **600 nanoseconds** of latency. It seems little, but there are workloads — GPU memory sharing, accessing distributed AI models — where even a few microseconds of addition make the system unusable.

**Topological flexibility.** The geometry of the cabling determines how many physical paths exist between two racks, how much east-west traffic can flow, and how difficult it is to reconfigure the network when machines are added or removed.

> [!tip] End-to-end latency: the hidden gap
> 
> A single Ethernet switch introduces ~600 ns, but as soon as one rises to the TCP/IP level the latency can reach **70 microseconds**. For context: a local DRAM access is in the order of 60–100 ns. Crossing even a single TCP hop introduces a penalty equivalent to hundreds of memory accesses. For workloads like LLM models distributed across GPUs — in the lecture GLM-4 with 750 billion parameters in a Mixture of Experts architecture is cited — this latency becomes the bottleneck of the entire system.

### Patch panels and functional zones

Since it is not practical — nor reconfigurable — to run direct cables from every rack to every other rack, the standard solution is the **patch panel**: a passive panel that serves as an intermediate switching point, allowing a connection to be logically "teleported" from one area of the data center to another simply by moving a cord, without redoing the structural cabling.

Structural cabling presupposes a division into **functional zones** of the data center. Since connecting everything to everything is physically impractical (a complete graph over N racks would require $\binom{N}{2}$ cables), roles are assigned to areas: telco zone for external links, compute zones, storage zones, and so on.

> [!warning] Consequences of incorrect cabling
> 
> If the geometry of the cabling does not reflect the actual communication pattern of the workloads, packets will be forced onto suboptimal paths: more hops, more latency, less bandwidth. This translates into visible performance limitations on VMs, even on public clouds like AWS or Azure — they are the direct result of physical choices made by Microsoft or Amazon when installing the data center.

### Physical organisation of racks: pods and corridors

In a data center, racks are typically organised in **distribution pods**: sets of physically adjacent racks, arranged in two rows facing a confined cold corridor for cooling. Physical proximity makes it acceptable, in some cases, to connect adjacent racks directly without a patch panel, removing the side walls of the racks and running cables internally from one rack to the next.

The fundamental distinction remains between **east-west** traffic (server-to-server within the data center) and **north-south** traffic (server towards the outside): they have radically different patterns and latency requirements, and every cabling choice impacts them differently.

---

## Transceivers: from Cable to Switch

### The optical connector problem

When network speeds exceeded the threshold at which RJ45 and copper twisted pair are sufficient, the move to optical fibre came. But here an economic problem emerges: single-mode OS2 fibre allows distances up to **40 km**, but inside a data center one does not exceed 300 metres. Purchasing industrial-power lasers designed for tens of kilometres is an enormous waste.

The solution was to separate the optical transmitter from the switch through a **modular transceiver**: a small active module that inserts into a dedicated slot of the switch, converts the electrical signal to optical and vice versa, and determines the type of fibre, the wavelength and the maximum supported distance.

> [!definition] Transceiver (SFP/QSFP module)
> 
> Hot-pluggable active module that inserts into a slot of the switch backplane. Converts electrical signals to optical and vice versa. The module's firmware exposes its characteristics to the switch (speed, fibre type, range). This allows **mix and match**: slot 1 with short-range multi-mode fibre, slot 3 with long-range single-mode, slot N with copper for debug — all on the same switch.

### The SFP family: evolution of standards

The transceiver ecosystem is organised around standardised families. Knowing these names is fundamental operational competence in a data center.

**SFP28** is the dominant standard for server-facing ports on leaf switches. It supports **25 Gbps** per lane and comes in distance variants: _SR (Short Range)_ at 850 nm on multi-mode fibre for distances up to ~100 m, _LR (Long Range)_ at 1310 nm on single-mode fibre for distances up to ~10 km, and _ER (Extended Range)_ for even greater reaches.

When SFP28 reached its physical limit (signal density would not scale beyond 25 Gbps with that form factor), the move was made to **QSFP** (_Quad SFP_): same concept but with **4 internal lanes**. The variants of the QSFP family are:

|Standard|Total bandwidth|Internal structure|Notes|
|---|---|---|---|
|QSFP+|40 Gbps|4×10G NRZ|First generation|
|QSFP28|100 Gbps|4×25G NRZ|Current standard for leaf↔spine uplinks|
|QSFP56|200 Gbps|4×50G PAM4|Spine-to-spine uplinks|
|QSFP-DD|up to 800 Gbps|8×100G PAM4|Latest generation, double density|

> [!note] The jump to PAM4 and the logic of the increment
> 
> The transition from QSFP28 to QSFP56 does not increase the number of lanes (they remain 4), but changes the modulation from NRZ to PAM4: instead of encoding 1 bit per symbol, PAM4 encodes 2 bits per symbol, doubling the bandwidth at the same frequency. QSFP-DD further doubles the lanes from 4 to 8, maintaining the same external physical dimensions but with a deeper contact board.

### Colour codes and module decoding

Transceivers use conventional **colour codes**: blue typically identifies single-mode fibre, black or grey multi-mode. The complete code is engraved on the module body: type (e.g. SFP28), speed (25G), wavelength (850 nm → multi-mode, 1310 nm → single-mode), range (SR/LR/ER).

**Why does multi-mode dominate the data center?** Because internal distances rarely exceed 100–300 metres, and multi-mode fibre is significantly cheaper both in cables and in lasers. In the lecture a practical comparison was made: an SFP28 SR (multi-mode) module costs about €45; two are needed (one for each end) plus the optical cable (~€10), for a total of ~€100. A DAC (_Direct Attach Cable_) copper cable has a similar cost, but the transceiver solution offers modularity: modules can be replaced to upgrade the speed without changing the cable.

> [!tip] Evolution of bandwidth per transceiver over time
> 
> 2005: 1 Gbps → 2010: 10 Gbps → 2015: 25–40 Gbps → 2020: 100–200 Gbps → 2024: 400–800 Gbps. In twenty years, a factor of 800×. Each generation required new form factors and new lasers, but the principle of slot modularity has remained unchanged.

---

## Switches: from Modular Chassis to Fixed Form Factor

### The chassis era

Until the early 2000s, the typical data center switch was a **modular chassis**: a large cabinet into which _line cards_ — boards with multiple Ethernet ports — and an **RPM** (_Routing Processing Module_) responsible for coordination through a shared _backplane_ were inserted.

These switches were very expensive, so typically a data center had **two** for redundancy, and all data center cables converged towards these two central switches. From this physical architecture, a star topology naturally emerged: a single central switching point around which everything was organised.

> [!note] Legacy in port naming
> 
> The `X/Y` notation of Ethernet ports (e.g. `0/1` = _line card 0, port 1_) is a direct legacy of the bidimensional structure of the chassis. On modern fixed form factor switches the line card no longer exists, but the convention has remained in all network management software.

### The backplane crisis and the transition

The chassis model made economic sense as long as bandwidth grew slowly. But when Ethernet speed began doubling every few years — from 1G to 10G to 25G — the chassis **backplane** could no longer keep up. Designing a backplane capable of supporting the current speed meant that in two years it would already be obsolete, and the expensive chassis purchased already had half-empty cards.

The response was the transition to **fixed form factor switches**: compact switches, typically 1U, with a fixed number of ports, mountable directly top-of-rack. These switches are cheaper, easier to replace, and bring the switching point closer to the servers, reducing cable length and number of hops.

---

## Network Topology: from 3-Tier to Spine-and-Leaf

### 3-tier architecture

Concentrating switching in a few central chassis produced the **3-tier** topology: _access layer_ close to the servers, _distribution layer_ for intermediate aggregation, _core layer_ with the large central chassis. All traffic had to pass through the core. This was acceptable as long as the dominant traffic was north-south.

![North-south traffic in 3-tier architecture](https://cdn.networkacademy.io/sites/default/files/2025-08/three-tier-architecture-traffic.svg) _Fig. 1 — In 3-tier architecture typical traffic is north-south: clients access servers from the outside. Core and distribution switches handle these flows efficiently._

With the advent of microservices, cloud and AI, east-west traffic has become dominant: every application request involves dozens of services communicating with each other inside the data center.

![3-tier inefficiencies with east-west traffic](https://cdn.networkacademy.io/sites/default/files/2025-08/three-tier-inefficiencies.svg) _Fig. 2 — With dominant east-west traffic, every server-to-server communication must climb access → distribution → core → distribution → access: variable latency, bottlenecks on uplinks._

### STP: necessary but costly solution

To ensure redundancy, multiple physical paths are needed, but more paths create loops in Ethernet.

> [!warning] Loops and broadcast storms
> 
> In Ethernet, a loop causes a **broadcast storm**: a broadcast frame bounces endlessly amplifying itself, saturates all bandwidth and renders the network unusable within seconds. It is a concrete problem that arises every time a cable accidentally creates a cycle in the topology.

> [!definition] Spanning Tree Protocol (STP / RSTP)
> 
> Layer 2 protocol (IEEE 802.1D / 802.1W) that dynamically builds a spanning tree of the network, **disabling links that would form cycles**. In the event of an active link failure, it recalculates the tree and re-enables a previously disabled link. STP converges in tens of seconds; RSTP in a few seconds.

![STP blocks redundant links in 3-tier topology](https://cdn.networkacademy.io/sites/default/files/2025-08/three-tier-spanning-tree.svg) _Fig. 3 — STP in action: redundant links (dashed/red) are disabled to eliminate loops. The physical cable is present and paid for, but carries no traffic._

The cost of STP is twofold. First: **resource waste** — the redundant link is physically present and paid for (cable, transceiver, switch port), but carries no traffic. Second: **slow convergence** — original STP takes up to **50 seconds** to converge after a fault, RSTP a few seconds. A 50-second interruption is acceptable for low-load web traffic; it is a disaster if the fabric connects VMs to their virtual storage units (the VM would lose the disk for the entire duration).

### Spine-and-Leaf: today's dominant topology

> [!definition] Spine-and-Leaf
> 
> Two-level network topology in which **leaf switches** collect the servers (typically top-of-rack, one per rack) and **spine switches** interconnect the leaves. Every leaf is connected to every spine, producing a regular bipartite graph. The path between any pair of servers is constant: leaf → spine → leaf, i.e. **2 physical hops, 1 logical hop**.

Leaf switches never talk to each other directly: all east-west traffic passes obligatorily through a spine. This guarantees **uniform and predictable** latency regardless of where the two servers are located.

![Spine-and-leaf architecture](https://cdn.networkacademy.io/sites/default/files/2025-08/leaf-spine-architecture.svg) _Fig. 4 — Spine-and-leaf: every leaf connects to all spines. Any server-to-server communication traverses exactly one spine, with constant latency. Adding capacity means adding leaves (horizontal scale-out)._

### Redundancy without STP: active-active link aggregation

With spine-and-leaf, redundancy is achieved through **link aggregation**: two physically distinct spine switches are made to operate as a single logical switch (through protocols like Cisco vPC, Arista MLAG, or the standard LACP protocol). Each leaf sees two cables towards the spine but treats them as a single logical high-bandwidth interface. The scheme is **active-active**: both links carry traffic simultaneously.

> [!example] Bandwidth aggregation with spine pair
> 
> If each leaf↔spine link is 25 Gbps, aggregating two spines gives 50 Gbps of available bandwidth between a leaf and the spine layer. A single TCP stream remains limited to 25 Gbps (a flow cannot be split across two links), but two parallel streams can use different links, summing to 50 Gbps total. If one spine falls, bandwidth degrades from 50 to 25 Gbps but connectivity does not break — no STP to wait for.

### 3-tier vs Spine-and-Leaf comparison

|Dimension|3-tier / Chassis|Spine-and-Leaf|
|---|---|---|
|Latency|Variable (depends on path)|Fixed and low (1 logical hop)|
|Bandwidth|Limited by hierarchy|Aggregated and scalable|
|Redundant links|Disabled by STP (wasted)|Active — active-active|
|East-west traffic|Suboptimal|Native and optimised|
|Convergence after fault|Seconds (RSTP)|Instantaneous (bandwidth degradation)|
|Scalability|Vertical (chassis upgrade)|Horizontal (add leaves)|

> [!abstract] Why spine-and-leaf won
> 
> Three reasons summarise the dominance of spine-and-leaf in modern data centers. First: **1-hop latency** — any server can reach any other with predictable latency. Second: **aggregated bandwidth** — no cost for unused links, all hardware works. Third: **east-west friendly** — internal data center traffic, fundamental for microservices, distributed storage and AI GPU clusters, no longer has to climb slow switch hierarchies.

---

## Data Center vs Campus Network

Spine-and-leaf is designed for the **east-west** traffic dominant in data centers. In **campus networks** — universities, offices, Wi-Fi networks — traffic is predominantly **north-south**: users access external resources. For this reason campus networks continue to use hierarchical 3-tier architectures with STP: slow convergence is acceptable, and the waste of redundant links is not critical.

> [!note] The campus Wi-Fi network
> 
> The university wireless network is a north-south architecture, not spine-leaf. This explains why in Wi-Fi one often cannot ping other devices connected to the same access point: the traffic is not optimised for lateral communication.

---

## Conclusions and Key Points

This lecture traced the path from the physical cable to the network architecture, showing how every low-level choice reflects in characteristics observable at the high level. Understanding these physical choices is essential to understanding why a VM on AWS has certain bandwidth limits, or why a distributed application on different racks behaves differently from one co-located on the same rack.

The course will continue with **converged and hyper-converged infrastructures** — technologies that directly depend on the ability of the spine-and-leaf fabric to manage east-west traffic with controlled latency and predictable bandwidth.

> [!question] Possible exam questions
> 
> - What are the three factors for which physical cabling is relevant in a data center?
> - What is the difference between SFP28 and QSFP28? When is it preferable to use one or the other?
> - Why were modular chassis switches abandoned in favour of fixed form factor?
> - What is the fundamental problem of STP in an east-west intensive environment?
> - How does redundancy work in spine-and-leaf without using STP?
> - Why does the latency of a single Ethernet switch (~600 ns) become critical for workloads like LLMs distributed across GPUs?
> - In which context is a 3-tier architecture still used? Why is it acceptable there but not in a data center?
