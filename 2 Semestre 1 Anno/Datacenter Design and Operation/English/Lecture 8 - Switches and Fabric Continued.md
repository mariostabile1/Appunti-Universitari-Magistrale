## Why the Network Changed

To understand the current architecture of switches it is necessary to understand _why_ it changed compared to the past. The lecturer traces a concise but illuminating periodisation of the evolution of IT workloads.

For almost three decades, the application landscape was divided into two categories: **HPC** (High Performance Computing), where computation exceeded the capacity of a single node and clusters were used, and **enterprise applications** — web servers, databases, middleware — where the available computing capacity far exceeded what was needed and it was preferred to concentrate multiple services on a single dense-core node. In both cases, the network was a secondary factor: enterprise applications lived predominantly on north-south traffic (client to server), while in HPC nodes communicated over specialised interconnects.

Everything changed around **2013**, with the explosion of the Big Data paradigm (Hadoop and similar). The mass adoption of SSDs, which lowered storage access latency from milliseconds to microseconds, made it convenient to move large quantities of data within the data center. So-called **East-West traffic** emerged: server-to-server communication within the same datacenter. This type of traffic demands low-latency, high-bandwidth networks between nodes — very different requirements compared to traditional traffic.

The final blow to the old order came with **generative AI**. For the first time a workload emerged that simultaneously requires: a lot of computation, a lot of fast memory, many GPUs and very high-bandwidth interconnects. The system balance breaks, and all components — CPU, memory, storage, network — must be rethought.

> [!tip] The balancing principle
> 
> A system is efficient only when all its components are dimensioned coherently. Having a 200 Gbps network with CPUs that process at a maximum of 80 Gbps is a waste. Data center design is always an exercise in balancing.

### The End of Chassis and the Fixed Form Factor

On the networking side, another change driver was the speed of technology evolution. In the past it was worthwhile to invest in expensive modular chassis (a chassis switch with replaceable cards), since the useful life of the investment was long. But when the capacity of port ASICs doubles every few years, the chassis became obsolete before its cards. The cost of a long-lived chassis was no longer justified.

The response was the **fixed form factor switch**: 1 or 2 rack units, not modular, but aggregable via software in spine-leaf topologies. This approach makes east-west traffic more predictable — both in latency and bandwidth — and allows updating the network by replacing entire switches (commodity hardware) instead of investing in proprietary chassis.

---

## Internal Architecture of the Switch

A modern data center switch is not a special appliance: it is fundamentally a **general purpose computer** to which specialised hardware for packet forwarding has been added. The lecturer insists on this point because it breaks the common perception of the switch as a networking "black box".

> [!definition] Control Plane and Data Plane
> 
> The **network switch** — like any complex communication device — divides into two distinct functional planes:
> 
> - The **Control Plane** is responsible for _configuration_, _management_ and _monitoring_. Policy logic, routing protocols, port counters and the network operating system reside here.
> - The **Data Plane** is responsible for _execution_: it receives packets on ingress ports, determines the egress port by consulting the tables populated by the control plane, and routes them at silicon speed.

![Control plane and data plane architecture in a network device](https://www.router-switch.com/faq/wp-content/uploads/2019/09/Management-Control-and-Data-Planes-in-Network-Devices-and-Systems.jpg) _Fig. — The three functional planes of a network device: management, control and data plane. In the modern data center switch, management typically coincides with the control plane._

### The Data Plane: Silicon Commodity

The data plane is where the true forwarding work is done, and today it is dominated by **ASICs** (Application-Specific Integrated Circuits) provided by a small number of vendors. Broadcom is the most cited name in this sector: chips like the **Jericho** are designed to store the complete routing table of the public Internet (and double, according to the specifications) and for terabit-per-second line-rate forwarding. Anyone wishing to build a switch buys the Broadcom catalogue, selects the chip based on desired performance and designs the board around it.

This has a fundamental consequence: **the data plane has become commodity**. The differentiation between vendors no longer lies in the silicon but in the control plane software and the management ecosystem.

> [!note] Non-blocking switching
> 
> A switch is said to be **non-blocking** when it can simultaneously switch all traffic on all its ports without any flow having to wait. With 48 ports at 25 Gbps + 8 uplinks at 100 Gbps, the backplane must handle: $(48 \times 25 + 8 \times 100) \times 2 = 3.6 \text{ Tbps}$ The $\times 2$ factor is because every port works in full-duplex (RX + TX). Modern data center switches are designed to be non-blocking, and the cost of this capacity is reflected in the power supply: they are 700 W machines, compared to 200 W in the past.

### The Control Plane: Linux inside the Switch

Above the data plane is the control plane, which is — surprisingly for those who have never worked with it — a **Linux** running on an Intel Celeron CPU or equivalent. There is RAM, flash storage, and a PCIe connection towards the data plane ASIC. The PCIe bus in a switch does not have the same bandwidth as that in a server: its function is to send configuration commands to the ASIC (update forwarding tables, change port policies), not to transport packet data.

> [!warning] PCIe between control and data plane: not a data pipe
> 
> The PCIe bus connecting control plane and data plane in a switch is dimensioned for _configuration_, not for _forwarding_. In principle, packets could be processed by the control plane CPU, but the available bandwidth is not sufficient to sustain the volume of traffic that the ASIC handles. This limits the type of functions implementable in software compared to hardware.

---

## SONiC and Open Networking

The fact that the silicon is commodity has opened the door to hardware/software disaggregation, a trend that in the server world happened decades ago (BIOS + generic OS) and that in the network was delayed for mixed reasons: technical and commercial.

**ONIE** (Open Network Install Environment) is the standard bootloader for switches that supports this disaggregation. Exactly like GRUB loads an OS onto a PC, ONIE loads a Network Operating System image onto the switch. Whoever buys an ONIE-compatible switch can choose which NOS to install, as one does with a server.

**SONiC** (Software for Open Networking in the Cloud) is the open-source NOS originally developed by Microsoft for its own Azure data centers and then donated to the community through the Linux Foundation. It is built on Linux, based on Docker containers for modularity, and uses Redis as an in-memory database for communication between components. Today it is co-maintained by Microsoft, Arista, Dell and many other vendors.

![SONiC — high-level architecture with containerised components](https://i.imgur.com/TAHKyAT.png) _Source: Azure/SONiC (GitHub) — SONiC architecture: each network function (BGP, LLDP, SNMP, DHCP relay) runs in a separate Docker container and communicates via Redis DB._

> [!note] Two OS images on the switch
> 
> Modern switches maintain two images of the operating system on disk: the _current_ running one and a _backup_ one. When updating the NOS, the new image is downloaded into the backup slot, the device is rebooted, and if something goes wrong, the previous image is restored with a command. This rollback pattern is fundamental for network resilience.

The lecturer recalls personally contributing to **Dell Networking OS 10** in 2014, when the ability to open a bash shell directly on the control plane was introduced. He went so far as to compile software directly on the switch's CPU using Visual Studio Code — which gives a concrete idea of what "the control plane is a PC" means.

In the market there is an ecosystem of NOSes: besides SONiC (open source), there are Arista's EOS, Dell's DNOS, and proprietary NOSes from other vendors. Some vendors offer SONiC with an optional proprietary layer, sold as an additional licence for those wanting professional support.

---

## The Switch CLI: a Modal Language

Switch configuration is done through a **modal CLI** — a command-line interface in which the available commands depend on the current _context_ (mode). This design stems from practical needs: early switches were configured by physically connecting a laptop to the serial port in the server room, in noisy environments, and therefore the CLI had to be as efficient to type as possible. The convention born at Cisco has become the de facto standard, imitated by almost all subsequent vendors.

The main modes are three, in hierarchical order:

1. **User EXEC** (prompt `>`): read-only. One can inspect but not modify.
2. **Privileged EXEC** (prompt `#`): accessed with the `enable` command. Allows advanced diagnostic operations.
3. **Global Configuration** (prompt `(config)#`): accessed with `configure` or `conf`. Allows configuration modification.

From Global Configuration one descends into specific contexts by typing an identifier (e.g. `interface Ethernet 1/1/3` enters the context of that port), and returns to the parent with `exit`.

> [!example] Typical CLI session on a switch
> 
> ```
> Switch> enable
> Switch# configure
> Switch(config)# interface Ethernet 1/1/3
> Switch(config-if-Et1/1/3)# no shutdown
> Switch(config-if-Et1/1/3)# switchport
> Switch(config-if-Et1/1/3)# exit
> Switch(config)# do show interface status
> Switch(config)# do write
> ```
> 
> The `do` prefix allows executing parent context commands without leaving it. The CLI also accepts unambiguous abbreviations: `conf` for `configure`, `sh run` for `show running-configuration`.

### Running Configuration and Startup Configuration

One of the most peculiar aspects of switch management — which surprises those coming from the server world — is the separation between **running configuration** (in-memory configuration, active) and **startup configuration** (on-disk configuration, loaded at boot).

Every CLI modification acts _only on RAM_. If the switch is rebooted without writing the configuration to disk, all modifications are lost. The command to persist is `write` (or `copy running-config startup-config`). This separation is intentional: it guarantees that the switch always returns to a deterministic state on reboot. It is not possible for an incremental software installation (as happens on a desktop OS) to leave the switch in an inconsistent state: it reboots and one gets exactly the saved configuration.

> [!warning] Configuration loss on reboot
> 
> If ports, VLANs or routing are modified on a switch and `write` is not executed, on the next reboot (planned or caused by a fault) _all modifications will be lost_. In production it is common practice to issue `write` (or `copy run start`) after every verified modification.

### Port Naming

Port names on modern switches follow a **three-level scheme**:

```
<stack_unit>/<slot>/<port>
```

- The first number identifies the **stack member**: when two or more physical switches are aggregated into a single logical switch (stacking), each unit has its own number.
- The second number identifies the **slot** or module in the chassis (for modular switches) or is always 1 for fixed switches.
- The third number is the **physical port number**.

There is also an optional fourth level for breakouts: a QSFP port (e.g. 100 Gbps) can be split into 4 SFP28 ports (25 Gbps each), in which case a suffix `/1`, `/2`, `/3`, `/4` is added. The port type name includes the speed for historical reasons (e.g. `Ethernet` for 1G, `TenGigabitEthernet` for 10G, simplified in the CLI with abbreviations).

---

## VLAN: Virtual Broadcast Domains

One of the most important concepts of the lecture concerns **VLANs** (Virtual LANs). To understand what a VLAN is, one must first be clear about what a LAN is.

> [!definition] LAN as Broadcast Domain
> 
> A **LAN** (Local Area Network) is a **broadcast domain**: all connected devices receive broadcast packets sent by any member. The medium access mechanism (CSMA/CD) implies that all nodes "listen" to the channel. This is what defines a LAN at the physical and Layer 2 level.

A **VLAN** extends this concept virtually: it allows **multiple independent broadcast domains** to be created on the same physical infrastructure. Instead of having one switch per network segment, a single switch can serve N logically separate segments thanks to VLAN tagging.

### The IEEE 802.1Q Tag

VLAN implementation is based on the **IEEE 802.1Q** standard, which introduces an additional **12-bit** field in the Ethernet frame header to carry the **VLAN ID** (VID). With 12 bits one can represent $2^{12} = 4096$ values, of which 4094 are usable (VID 0 is reserved and VID 1 is the default).

$$ \text{VLAN ID} \in [1, 4094], \quad \text{bits: } 12 $$

> [!definition] Tagged vs Untagged Port
> 
> - An **untagged** port (or _access port_) connects to end-host devices that do not know about VLANs. The switch adds the tag when the frame enters and removes it when it exits. The assigned VLAN is configured per port.
> - A **tagged** port (or _trunk port_) connects to another switch or to a VLAN-aware host. Frames transit with the 802.1Q tag intact. A trunk port can carry frames from multiple VLANs.

![Ethernet frame format with 802.1Q VLAN tag](https://upload.wikimedia.org/wikipedia/commons/0/0e/Ethernet_802.1Q_Insert.svg) _Source: Wikimedia Commons — The 802.1Q tag (4 bytes, TPID 0x8100) is inserted in the Ethernet header between Source MAC and EtherType. The 12 VLAN ID bits allow up to 4094 distinct VLANs._

Inside the switch, every packet is _always_ tagged: if it arrives untagged, the ASIC's first step is to assign it to the default VLAN of the ingress port. Packets travel internally with the tag, and only on exit through an untagged port is the tag removed.

> [!example] VLAN forwarding behaviour
> 
> A frame enters on port `Eth1/1/1` configured as untagged VLAN 10 → the switch adds tag VID=10. The ASIC looks up the destination MAC in the forwarding table filtering for VID=10 → finds port `Eth1/1/5` configured as trunk (tagged). The frame exits with the 802.1Q tag intact. If the destination MAC were instead on an untagged VLAN 10 port, the frame would exit without a tag.

### Why VLANs Are Critical

VLANs solve two fundamental data center problems:

**Security and segregation**: VLANs separate traffic as if they were physically distinct networks. For this reason even military forces — which traditionally used three separate physical networks (confidential, restricted, top secret) — have accepted the use of three VLANs on the same infrastructure. The guarantee is that a packet in VLAN A can never, by virtue of how the ASIC works, reach a host in VLAN B, unless an explicit routing decision is made.

**Topological flexibility**: physical cabling is expensive and difficult to modify. VLANs allow the logical network segmentation to be redesigned simply by reconfiguring port tagging, without touching a single cable. This has been instrumental in making data centers agile: subnets can be created or deleted in minutes.

> [!note] VLANs and VRFs
> 
> In advanced configurations, VLANs are associated with **VRF** (Virtual Routing and Forwarding) to also create distinct virtual routing tables. This completes the logical separation at the entire Layer 3, not just Layer 2.

---

## Resilience and the Complexity of Network Debugging

The lecture concludes with a reflection on why the network is designed as it is — functional, deterministic, resilient — and why network debugging is particularly difficult.

The lecturer brings two anecdotes from direct experience with the University of Pisa network:

1. A **network loop** (cycle in the Layer 2 topology) had caused an outage. The loop involved thousands of devices and ports, making the identification process extremely long — at the time of the lecture the loop was still present and being searched for.
    
2. An **intermittent fault** on a 15 km optical fibre link between two sites had caused a two-to-three-day downtime. The cause was probably physical damage to the fibre (an agricultural machine had driven over the buried cable). The LACP link used two fibres: one functioning and one degraded. The result was intermittent behaviour very difficult to diagnose — half the traffic worked, half generated errors — with the appearance of a software or configuration problem.
    

These examples illustrate a general principle:

> [!tip] The network is simple but its emergent behaviours are complex
> 
> A single switch does one thing only: receive a packet and decide on which port to forward it (store-and-forward). The algorithm is trivial. But when billions of packets per second traverse dozens of switches, the emergent behaviours are enormously complex. There is no "debugger" for the network because stopping execution _changes_ the behaviour of the network itself. For this reason everything in networking — from software design to CLI to the dual boot image — is conceived to reduce the probability of inconsistent states.

---

## Conclusions and Key Points

This lecture covered the entire hardware-to-software chain in modern network switches:

- The switch is a **general purpose computer** with an x86 CPU (control plane, Linux) and a specialised ASIC (data plane, silicon commodity from Broadcom et al.) connected via PCIe.
- The **control plane / data plane** distinction is the fundamental conceptual framework for understanding any network device.
- **SONiC** and **ONIE** represent the market's direction towards hardware/software disaggregation, with interchangeable open-source NOSes like server OSes.
- The **modal CLI** — Cisco heritage — is still the primary configuration tool, with the running/startup config mechanism guaranteeing determinism at boot state.
- **VLANs** (IEEE 802.1Q, 12 bits of VLAN ID, 4094 possible VLANs) are the fundamental tool for creating virtual broadcast domains on the same physical infrastructure, enabling both security and topological flexibility.
- Network debugging is intrinsically difficult due to traffic volume and the dynamic nature of the system: the functional approach (switches always returning to the same state on reboot) is the architectural response to this problem.

> [!question] Possible exam questions
> 
> - What is the difference between control plane and data plane in a switch? What type of CPU and bus connects them?
> - What is ONIE and why is it important for open networking?
> - What is meant by "non-blocking switch"? Make a calculation of the bandwidth required for a 48×25G + 8×100G.
> - Why does the switch have two configurations (running and startup config) instead of one?
> - Define LAN as a broadcast domain. What does the VLAN concept add? How many VLANs can be created and why?
> - What does tagged vs untagged port mean? What happens to an untagged packet when it enters a switch with VLANs configured?
> - Explain the `1/1/3` naming of a port on a stack switch.
