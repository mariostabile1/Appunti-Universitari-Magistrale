# Lecture 1 — Introduction to Data Centers

> Course: *Data Center Design and Operation* — Prof. **Antonio Cisternino**, University of Pisa
> Horizontal view of the infrastructure behind Cloud, VMs and HPC systems.

---

## From Server Rooms to Gigafactories

Twenty years ago a data center was a room with an air conditioner, a few racks and desktop PCs without monitors. Today it is critical national-level infrastructure.

> [!info] Legislative definition
> The Italian parliament is working on a **framework law** to regulate data centers. A facility is considered of **national relevance** when it exceeds **0.5 MW** of power draw.

**Historical references (University of Pisa):**
- **2013** — Cisternino's first data center (near the entrance café): **80 kW**
- **Today** — San Piero campus: **250–300 kW**

With the explosion of **AI factories**, we are entering the era of **Gigafactories**: there are projects for data centers drawing **2.5 GW**, a value that imposes extreme constraints on the national power grid (in Italy very few sites can deliver that much power at a single point).

---

## Network Infrastructure and Switching

| Context | Technology | Speed |
|---|---|---|
| Campus / DC Management | Ethernet RJ45 | < 1 Gbps |
| Main DC Traffic | Optical fibre | 10–25 Gbps |

**Modern switch (< $10,000):**
- 48 ports × 25 Gbps
- **Non-blocking** architecture → no overbooking, zero bottlenecks
- Switching capacity: **1.2–2.4 Tbps** (up to 24 Tbps in TX+RX)

> [!note] Non-Blocking Switch
> All nodes can transmit and receive simultaneously without slowdowns. The switch itself becomes an intensive computing system.

---

## The Paradigm Disrupted by AI

### Before AI: virtualisation and Cloud

Modern laptops and servers operate at **10% of their potential** for 90% of the time → starting in **2008**, the enterprise industry exploited **virtualisation** to recover this idleness, giving rise to the **Cloud**.

### AI breaks the balance

AI is the **first workload that fully saturates physical hardware**. Consequences:

- Virtualisation makes no sense in the AI stack
- Distributed architectures across **multiple GPUs** are used
- **Communication latency** is a critical constraint → cables must be as short as possible

### Computational cost of tokens

> [!example] Example: 7B-parameter LLM
> - 1 token (≈ 3 characters) → **7 billion multiplications**
> - 1,000 tokens → **~7 trillion operations**

For models like **GPT-4** (estimated ~600B parameters), the cost would be unworkable without the **Mixture of Experts** technique: the network divides into specialised layers, limiting active multiplications to **1–3 billion per expert**.

> [!note] Mixture of Experts (MoE)
> Computational paradigm that dynamically activates only a small portion of the parameter matrix for each inference, drastically reducing the per-token cost.

### Required hardware (2024–2025)

To run open-source models with **120B parameters** (e.g. GPT-OSS, Deepseek R1):
- 2× **Nvidia RTX 6000** → ~**€50,000 each**

---

## Physics of Data Centers: Power and Cooling

### Evolution of power density per rack

| Period | Power per rack |
|---|---|
| Early 2000s | 3 kW |
| 2013–2016 (San Piero) | 15 kW |
| Today (AI hardware) | ~200 kW |
| Near-term forecast | 500 kW – 1 MW |

> [!warning] Scale: 200 kW per rack
> Equivalent to the maximum power of **70 apartments** connected simultaneously to a single cabinet.

### Physical implications

- **Cabling:** at 200 kW traditional copper is unusable → a **rigid power bar** in the rear wall of the rack is used
- **Cooling:** **liquid cooling** has gone from a futuristic option to an **absolute necessity** — an HPC system cannot be powered on without active liquid cooling

> [!tip] Why do chips generate heat?
> Boolean algebra is intrinsically dissipative. In logic gates (e.g. AND with two `1` inputs), the excess voltage cannot accumulate and is released as **heat**. **Fredkin Gates** (trinomial reversible, non-dissipative gates) would have solved the problem, but were never commercially realised due to manufacturing limits.

---

## Supercomputers and Geopolitical Tensions

### Top500 & Green500

- **Top500**: semi-annual ranking (since 1993) based on linear algebra benchmarks
- **Green500**: efficiency in **petaflops/Watt**
- Participation is not mandatory (e.g. NSA, Ferrari in 2003 did not certify their systems)

### Top systems

| System | Notes |
|---|---|
| Frontier | USA |
| Aurora | USA |
| Jupiter Boost | Europe |
| Fugaku | Japan (former champion) |
| **Eagle** (Microsoft Azure) | 2M cores, AI-oriented, **virtually distributed** |
| **Leonardo** (CINECA) | 2M cores, **10th place**, Italy |

### European geopolitics

- **2019**: post-Brexit, the European Commission (dir. **Roberto Viola**) invested **~€2 billion** over 7 years to fill the gap left by the United Kingdom
- Italy launched a public tender (managed by CINECA) for a **national AI Factory**: **€500 million**

> [!info] Scale of US investments
> **Project Stargate** (Oracle/OpenAI/Ellison): **$500 billion**
> **Nvidia**'s market capitalisation exceeds the GDP of Italy, France and Germany combined.

---

## Location: Geographic and Fiscal Constraints

### Italy is an unfavourable country for data centers

> [!danger] Italian structural problems
> 1. **Seismic risk** — Delicate hardware is vulnerable to tremors. Example: the Lazio Regional data center is built on the first floor on **anti-seismic damping pillars**.
> 2. **Thermal excursion** — Oscillations of up to **50°C** between summer and winter destabilise cooling systems (the same phenomenon that cracks Italian asphalt).

Same problem: southern Spain, Barcelona, Greece.

### Solutions adopted worldwide

| Approach | Example |
|---|---|
| Cold deserts at night | Nevada (night ice for daytime cooling) |
| Nordic countries | Scandinavia, Iceland (but: risk of volcanic eruptions) |
| Alpine caves | ENI — supercomputer in the Alps at constant temperature |
| Free winter cooling | **Aruba** (Arezzo) — reversible turbine: cold air in winter, extraction in summer |
| Seabed | **Microsoft Project Natick** — five-year cycles; risks: salt corrosion, difficult servicing |

### Tax advantages: the Irish example

Google, Apple, Microsoft and others chose **Ireland** primarily for **tax incentives** linked to EU membership, not just for the favourable climate.

---

## Acoustic, Fiscal and Management Issues

### Acoustic impact

**Chillers** (cooling pumps) are loud and continuous. The IT department data center at the University of Pisa was dismantled following a **court ruling**: despite being below the legal decibel threshold, a nearby professor filed a lawsuit and won, forcing its relocation to an out-of-town campus (with further bureaucratic problems related to protected woodland areas).

### Electrical infrastructure

In Pisa, the substation near the café was rated up to **2 MW**, but the urban grid limited the installation to **80 kW**.

### Large-scale management

> [!important] Documentation and management
> In a server farm with **700+ machines** interconnected via optical fibre, physically tracing a cable in the event of a fault is not feasible.
> Every component, logical port and cabinet must be catalogued and traceable via **centralised management software**.
> Energy efficiency is measured with **PUE (Power Usage Effectiveness)**.

---

## Glossary

| Term | Definition |
|---|---|
| **Framework Law** | Legislative process to regulate nationally relevant data centers |
| **Non-Blocking Switch** | Switch without overbooking: all nodes transmit/receive without queuing |
| **Mixture of Experts (MoE)** | AI paradigm that dynamically activates only a fraction of parameters to reduce per-token computational cost |
| **Top500 / Green500** | Semi-annual HPC performance rankings (since 1993) and energy efficiency rankings (petaflops/W) |
| **Fredkin Gates** | Theoretical trinomial reversible logic gates, thermally non-dissipative — never commercially realised |
| **PUE** | Power Usage Effectiveness: ratio of total DC power to IT equipment power consumption |
| **Liquid Cooling** | Liquid-based cooling, now mandatory for high-density HPC hardware |
