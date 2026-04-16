
# Storage — Technologies and Fundamental Principles

This lecture opens the second great pillar of a datacenter: **storage**. After studying the fabric (internal networking), and before addressing compute, the lecturer devotes this session to understanding how drives are built, how the technology has evolved, and what the fundamental principles are that guide the design of reliable storage systems.

---

## The Three Pillars of a Datacenter

A datacenter is defined by the flexible aggregation of three fundamental resources:

1. **Networking** (the fabric) — already covered
2. **Storage** — subject of this lecture
3. **Compute** — future lecture

The fabric is the glue that holds everything together: if it breaks, the system is down. Storage is instead the place where data survives the loss of power. This characteristic — **persistence** — is at the centre of every architectural choice in the storage domain.

---

## HDD vs SSD: The Silent Revolution

### The mechanical disk and its physical limits

For decades, the generic term "disk" designated **HDDs** (*Hard Disk Drives*): devices with rotating magnetic platters, read/write heads, sectors and cylinders. Physics imposes an insurmountable limit: access time depends on mechanical movement, both rotational (the platter must bring the correct sector under the head) and linear (the head must move to the right track). This time, called *seek time* and *rotational latency*, brings total latency to the order of **3–5 milliseconds**.

> [!tip] Why HDD latency is a physical wall
>
> No matter how sophisticated the electronics: as long as the system depends on a motor rotating a platter at 7,200 or 15,000 revolutions per minute, the minimum latency is imposed by mechanics. No algorithm can eliminate rotation time.

The dominant interface for HDDs was for years **SATA** (*Serial ATA*), born for desktop PCs and then adopted in servers for the lower cost. In the server domain **SAS** (*Serial Attached SCSI*) also existed: a more performant protocol, typically used with 15,000 RPM drives, with slightly lower latency and multipath support. Today, with HDDs relegated primarily to cold storage (backups and archives), SAS has lost much of its practical relevance: SATA is preferred for cost reasons, since performance is no longer the primary objective of a modern HDD.

> [!note] HDD capacities today
>
> Modern mechanical drives reach capacities of **30–36 TB** (Samsung, Seagate, Western Digital, 2024). This progress exclusively reflects their use as *cold storage*: density is maximised because throughput is not critical.

![Inside a Hard Disk Drive: rotating magnetic platters and read/write heads](https://upload.wikimedia.org/wikipedia/commons/3/38/Seagate_ST33232A_hard_disk_inner_view.jpg)
*Fig. — The internal anatomy of a Seagate HDD: the rotating magnetic platters and the head arm make the mechanical constraint that limits latency to 3–5 ms visible.*

---

### The transition to SSD: 30x in a single leap

SSDs entered the consumer market around **2010**, initially as expensive devices of small capacity, in the 2.5" form factor to be able to use the same mechanical infrastructure as HDDs. The change was not gradual: the first generations of SSDs on the SATA interface offered a latency reduction of approximately **30x** compared to HDDs — from milliseconds to hundreds of microseconds. The result was a "silent revolution" that few fully understood at the time.

> [!warning] Cascading impact on the entire stack
>
> The acceleration of storage was not confined to hardware. The Linux operating system (and others) was hiding concurrency bugs latent for over twenty years in the I/O subsystem, never triggered because the disk was too slow. With the arrival of SSDs, some race conditions became manifest. Parts of the kernel had to be rewritten and decades of assumptions about disk slowness had to be revised.

---

### NVMe: a protocol, not a device

The second major breakthrough was the shift of SSDs from the SATA controller to direct connection to the **PCIe** (*PCI Express*) bus via the **NVMe** (*Non-Volatile Memory Express*) protocol.

It is fundamental to understand that **NVMe is not a hardware component**: it is a **protocol standard** that defines how a block storage device must behave on the PCIe bus. In practice, an NVMe drive presents itself to the system exactly like any other PCIe card — a network card, a GPU — and the operating system recognises it automatically without proprietary drivers.

The effect on the bottleneck was immediate:

| Interface | Typical latency | Typical bandwidth |
|---|---|---|
| HDD SATA/SAS | 3–5 ms | 100–250 MB/s |
| SATA SSD (2015) | 50–200 µs | 500–550 MB/s |
| NVMe Gen 3 SSD (2016+) | 20–100 µs | ~3.5 GB/s |
| NVMe Gen 4 SSD (2019+) | 10–50 µs | ~7 GB/s |
| NVMe Gen 5 SSD (2022+) | 5–20 µs | 10–14 GB/s |

![NVMe M.2 2280 SSD 1 TB](https://upload.wikimedia.org/wikipedia/commons/e/ed/1TB_2280_NVME_SSD.jpg)
*Fig. — An NVMe SSD in M.2 2280 form factor: direct connection to the PCIe bus eliminates the SATA controller and brings bandwidth above 7 GB/s.*

> [!tip] The controller as bottleneck
>
> The SATA/SAS controller was a hardware chip interposed between the drive and the CPU, useful when the drive was slow and needed buffering and command queuing. With SSDs, the controller became the **bottleneck**: by removing it and connecting the drive directly to the PCIe bus, a gain of **4–5x in bandwidth** and **3–4x in latency** was obtained at the same flash technology.

Each NVMe drive typically uses **4 PCIe lanes**. The current latency of the fastest enterprise NVMe drives — about 5 µs — is a step away from the 2 µs of InfiniBand, the best HPC fabric. Storage, from a slow component, is becoming "slow persistent memory".

---

## The Systemic Consequence: PCIe as Bottleneck

With the removal of the controller and direct connection to the PCIe bus, the entire ecosystem had to adapt. PCIe Gen 3 had held for about 10 years, but suddenly became the new bottleneck.

The response was a rapid evolution of standards:

- **PCIe Gen 4** (2019): doubles bandwidth compared to Gen 3
- **PCIe Gen 5** (2021): doubles again — ~14 GB/s per drive
- **PCIe Gen 6 and 7**: in development

> [!example] AMD EPYC "Naples" (2017) and PCIe lanes
>
> When AMD returned competitively to the server segment with the EPYC series, one of its key value propositions was the offering of **128 PCIe lanes** per socket. The choice was not casual: the sector had already understood that the CPU needed to be able to directly feed NVMe drives without intermediaries, and more lanes meant more drives operating in parallel.

A typical server with 24 NVMe drives (24 × 4 lanes = 96 lanes) requires a number of PCIe lanes that only modern CPUs can provide. This pushed manufacturers to directly integrate the PCIe controller into the CPU die, eliminating the bridge chip that added latency.

---

## Memory Hierarchy and the New Position of SSDs

The traditional memory hierarchy has always been:

```
L1/L2/L3 Cache (ns) → DRAM (50–100 ns) → Disk (ms)
```

With modern NVMe SSDs, the hierarchy has transformed:

```
L1/L2/L3 Cache (ns) → DRAM (50–100 ns) → NVMe SSD (5–20 µs) → HDD (3–5 ms)
```

The jump between DRAM and SSD is no longer 6 orders of magnitude (as between ns and ms), but **2 orders of magnitude**. This has profound algorithmic consequences: data structures that previously had to reside entirely in memory — because serialisation to disk was prohibitive — can now be maintained directly on the SSD.

> [!example] RocksDB as a paradigmatic case
>
> RocksDB (Facebook/Meta) is a library that implements a *persistent hash table* optimised for SSDs. Before fast SSDs, any data that needed to survive reboots required serialisation to disk with intermediate buffers in memory. With RocksDB on NVMe, key-value pairs are written directly to persistent storage without explicitly managing buffers or double structures. The performance is good enough to make the pattern acceptable.

---

### Intel Optane (3D XPoint): The Persistent Memory That Wasn't

Intel Optane is a **non-NAND** technology that deserves mention, despite Intel cancelling it in 2022. It used a three-dimensional junction structure (*3D Cross-Point*) that allowed storing a bit at each intersection of a 3D lattice. The properties that distinguished it from NAND flash:

- **Nearly symmetric latency**: read and write had comparable times (factor 3–10x, versus the 6–100x of NAND)
- **Much superior endurance**: millions of PE cycles versus the thousands of TLC NAND
- **Memory bus connection**: installable in DIMM slots (*NVDIMM*), not just on PCIe

> [!definition] NVDIMM — Non-Volatile DIMM
>
> Electrical standard introduced by Intel around 2015 to allow the installation of persistent memory modules on the same bus as standard DRAM modules. Optane was available in 512 GB per slot modules, allowing servers with **terabytes of persistent RAM**.

The technology allowed the processor to perform automatic *tiering*: hot data resided in DRAM, warm data migrated to Optane. In terms of latency, Optane NVMe was around **10 µs** with minimal variation between read and write — the closest approximation to DRAM as persistent storage ever commercialised.

The reasons for failure were two:
1. **Intel was going through an internal crisis** and had to cut costly projects
2. **The software was not ready**: operating systems did not know how to exploit memory tiering with sufficient efficiency, and NVMe SSDs were meanwhile improving rapidly

> [!note] The security problem of persistent memory
>
> Persistence in DIMMs introduces an unforeseen attack vector: a physical attacker who removes the modules can read their content offline. Intel resolved this by generating a random cryptographic key at boot, stored in a volatile register. All data written to Optane was encrypted with that key. If the module was removed, the register was lost, rendering the content unusable — simulating the behaviour of traditional DRAM.

---

## Read/Write Asymmetry in SSDs

All NAND-based devices (unlike Optane) have a **fundamental asymmetry** between read latency and write latency.

> [!definition] PE Cycle — Program/Erase
>
> A NAND cell cannot be directly rewritten: it must first be **erased** (*erase*) and then **programmed** (*program*). Erasure occurs at the block level (thousands of cells), while writing occurs at the page level. This generates write amplification and asymmetric latency.

| Technology | Read latency | Write latency | Ratio |
|---|---|---|---|
| DRAM | ~50–100 ns | ~50–100 ns | ~1x (symmetric) |
| Intel Optane | ~6–10 µs | ~6–10 µs | ~1–3x |
| NVMe Gen 5 SSD | ~20–50 µs | ~50–200 µs | ~2–7x |
| SATA SSD (2015) | ~50–100 µs | ~500 µs–5 ms | ~10–100x |
| HDD | ~3–5 ms | ~3–5 ms | ~1x (symmetric, but slow) |

This asymmetry is relevant in architectural choices: write-intensive workloads require drives with better endurance (SLC or MLC), and storage systems must provide appropriate write buffers.

---

## Types of NAND Cells and Endurance

Not all SSDs are equal: the density of NAND cells — measured in bits per cell — determines a trade-off between capacity, cost, performance and useful life.

> [!definition] PE Cycles (Program/Erase)
>
> Each NAND cell can be erased and rewritten a finite number of times. This number, called *PE cycles* or *endurance*, decreases as bits per cell increase: more states to distinguish means narrower electrical margins and faster wear.

| Type | Bits/cell | Typical PE Cycles | Write latency | Cost/GB | Use case |
|---|---|---|---|---|---|
| **SLC** | 1 | ~50,000–100,000 | ~200 µs | Very high | Cache, high-frequency write storage |
| **MLC (eMLC)** | 2 | ~10,000–30,000 | ~600 µs | High | Enterprise, mixed read/write workloads |
| **TLC** | 3 | ~1,000–3,000 | ~1–3 ms | Medium | Standard consumer and enterprise today |
| **QLC** | 4 | ~100–1,000 | ~5–10 ms | Low | Cold storage, archive |
| **PLC** | 5 | <100 | ~10–20 ms | Very low | Pure archiving (*write once*) |

> [!warning] SLC as internal cache
>
> Many TLC and QLC drives include a portion of cells configured in SLC mode as an internal **write buffer**. Initial writes go into the SLC cache (very fast), then are migrated to TLC/QLC in the background. When the SLC buffer is exhausted (drive nearly full), write performance drops sharply.

---

### Over-provisioning and preconditioning

A 4 TB SSD might internally contain 6–8 TB of NAND cells. The "hidden" capacity beyond what is exposed serves two purposes:

1. **Wear levelling**: the controller distributes writes evenly across all cells to prevent some from exhausting before others. Having spare cells multiplies the effective drive life.
2. **Performance**: the controller uses the free space to optimise garbage collection and internal reorganisation operations.

> [!warning] "Preconditioning" and the red bar
>
> When a drive exceeds approximately **90% fill**, performance degrades significantly. The controller has less free space for wear levelling and optimisation. macOS and Windows show the usage bar in red at exactly this point — not to indicate that the disk is nearly full in an absolute sense, but to signal that **performance is about to worsen**. Practical rule: keep an SSD below 75–80% to preserve performance.

---

## The Unforgivable Sin: Losing Data

Before addressing technical solutions, the lecturer states the cardinal principle of the entire storage domain:

> [!warning] The fundamental rule of storage
>
> In IT there are serious but recoverable events — a server that breaks, a network that goes down. There is however **one unforgivable sin**: **losing data**. Everything else can be restored; lost data does not come back.

This is not a technological fact, it is an organisational and cultural one. Any storage system must be designed starting from this axiom: the disk *can* and sooner or later **will** fail. The question is not "if" a drive will break, but "when".

> [!example] The Brussels datacenter
>
> A cloud provider installed two datacenters in the same Brussels city block, in separate buildings with independent power lines. The two datacenters were configured as backups of each other. An exceptionally large fire destroyed both buildings. Result: total loss of data. European regulations now require a minimum distance of **11–12 km** between datacenters used for data replication, precisely to mitigate risks of this type (floods, fires, local power grid failures).

---

## RAID: Redundancy Through Aggregation

**RAID** (*Redundant Array of Inexpensive Disks*) was born in the 1980s not as a backup solution, but as a response to an economic problem: a large-capacity drive cost much more than two smaller drives. The idea was to aggregate inexpensive drives into a single logical volume, compensating for their lower reliability with redundancy.

The mechanism common to all RAID levels is **parallelism**: if I have two drives and need to write two blocks, I can send them simultaneously to the two drives, performing two writes at the temporal cost of one.

---

### RAID 0 — Striping (no redundancy)

**RAID 0** is not properly a RAID in the sense of redundancy: it is **striping**, i.e. the distribution of data across multiple drives to present the system with a single larger logical volume. Blocks are alternated between drives: block 0 → drive A, block 1 → drive B, block 2 → drive A, etc.

The advantage is twofold: aggregated capacity and potential to read/write in parallel from the two drives. The disadvantage is a total absence of protection: the loss of any drive causes the loss of the entire volume.

---

### RAID 1 — Mirror

**RAID 1** is the most intuitive form of redundancy: every block is written identically on two drives. In the event of a drive failure, one operates on the second without interruption. The price to pay is **halved effective capacity**: two 4 TB drives give 4 TB usable.

In compensation, reads can be distributed across the two drives, improving read throughput. The *rebuild* phase (copying the functioning drive to the new drive) does not require taking the volume offline.

---

### RAID 5 — Distributed parity with XOR

**RAID 5** uses an elegant mathematical mechanism to achieve redundancy with an overhead of **1/N** rather than the 50% of RAID 1.

> [!theorem] The XOR property that makes RAID 5 possible
>
> The **XOR** (*exclusive OR*) operation has a fundamental property: if $A \oplus B = C$, then knowing any two of the three values one can reconstruct the third: $A = B \oplus C$, $B = A \oplus C$.
>
> In RAID 5 with 3 drives, data is stored on drives A and B, and their XOR on drive C (called the parity block). If any of the three drives fails, its content is reconstructed by calculating the XOR of the other two.

With $N$ drives in RAID 5:
- **Usable capacity**: $(N-1)/N$ of the total (with 3 drives: 66%, with 6 drives: 83%)
- **Tolerance**: loss of **1 drive**
- **Rebuild**: requires taking the volume offline, recalculating all parity blocks

The hidden cost of RAID 5 is **degraded mode**: when a drive fails and one awaits replacement, the system must perform on-the-fly reconstruction of every missing block, increasing read latency. If a second drive fails during the rebuild, everything is lost.

---

### RAID 6 — Double parity

**RAID 6** extends RAID 5 with **double parity** (using more sophisticated mathematical algorithms than simple XOR, based on Galois fields). This allows:

- Tolerating the loss of **2 drives** simultaneously
- Remaining **online** in degraded mode with 1 failed drive during rebuild

The capacity cost is slightly higher than RAID 5 (overhead of 2/N instead of 1/N), but the reliability gain is significant.

**RAID 6 is today the de facto standard** in storage systems, preferred over RAID 5 because the risk of a second drive failing during the long rebuild process of a modern array (hours on drives of tens of TB) is real.

---

### Summary RAID comparison

| Level | Min drives | Usable capacity | Drives tolerated | Online with 1 failure | Mechanism |
|---|---|---|---|---|---|
| RAID 0 | 2 | 100% | 0 | No | Striping |
| RAID 1 | 2 | 50% | 1 | Yes | Mirroring |
| RAID 5 | 3 | $(N-1)/N$ | 1 | Degraded | XOR parity |
| RAID 6 | 4 | $(N-2)/N$ | 2 | Yes (with 1) | Double parity |

> [!note] Who implements RAID?
>
> Historically, RAID was implemented by the disk's **hardware controller**: the chip interposed between the drives and the CPU exposed a single logical volume to the operating system. With the transition to NVMe (which eliminates the controller), RAID must be implemented by the **operating system** (Linux `mdadm`, Windows Storage Spaces) or by dedicated software. The OS is perfectly capable of doing it, but the transition required adaptation in administration workflows.

---

## Towards Distributed Storage Architectures

The lecture closes with a preview: the single server with RAID is not sufficient for the needs of modern datacenters. The natural evolution leads to **centralised and distributed storage architectures**, which will be covered in subsequent lectures:

- **SAN** (*Storage Area Network*): shared storage accessible via dedicated network
- **NAS** (*Network Attached Storage*): shared file system via standard network
- **HCI** (*Hyper-Converged Infrastructure*): distributed storage using local server drives, with software replication at **3 copies** (3x overhead, but superior availability properties)

> [!abstract] Lecture summary
>
> The transition from HDD to NVMe SSD is one of the most underestimated revolutions in computing of the last 15 years. It reduced storage latency from milliseconds to microseconds, transforming the disk from a slow component to near-persistent memory. This transformation forced the redesign of controllers, PCIe buses, CPUs, operating systems and software architectures. The guiding principle of all storage remains unchanged: **losing data is unforgivable**, and every technological choice — from the NAND cell type to the RAID level — revolves around maximising reliability at the lowest possible cost.

> [!question] Possible exam questions
>
> - What is the difference between HDD and NVMe SSD latency? Why does physics limit HDDs?
> - What does it mean that NVMe is a protocol and not a hardware device?
> - Why did the disk controller, born to simplify drive design, become a bottleneck with SSDs?
> - Explain the XOR property that makes RAID 5 possible and its single-fault tolerance limit.
> - Compare RAID 5 and RAID 6: in which scenario is one preferred over the other?
> - What is meant by "pre-conditioning" of an SSD and why does performance degrade beyond 90% fill?
> - What are PE cycles and how do they influence the choice between SLC, MLC, TLC, QLC?
> - Why does European regulation impose a minimum distance between replica datacenters?
