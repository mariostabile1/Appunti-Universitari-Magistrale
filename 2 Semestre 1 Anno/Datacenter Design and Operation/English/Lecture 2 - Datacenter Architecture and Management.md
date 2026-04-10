# Datacenter Architecture and Management

This lecture explores the fundamental infrastructure of modern datacenters: not just the technological components, but the engineering, physical and legal challenges tied to their design. The running thread is the permanent tension between two opposing needs — the total **redundancy** demanded by the user and the **economic efficiency** demanded by the provider — which drives every design decision, from the floor to the copper cables.

## Fundamentals and Design Constraints

### The Complexity of the Datacenter and the Pursuit of Efficiency

A datacenter is not a complex environment because of the high number of different components; at its core, the "ingredients" are few: a building housing a set of server cabinets (**racks**), electrical power, network equipment, cabling and a **cooling** system. The true complexity stems from the enormous resources required to run it and the constant pursuit of efficiency.

Until recently, air was used as the primary cooling medium not because it was the most efficient, but because it was cheap and immediately available. Today this choice is being questioned as the energy density of racks grows.

> [!tip] Efficiency as an economic metric
>
> Intel demonstrated that changing the operating design temperature of CPUs by just one degree Celsius could generate energy savings worth millions of dollars. More recently, Dell is bringing to market **AI racks** of 200 kW designed to operate at 40°C: the elevated threshold drastically reduces the energy spent on cooling, and since the rack is physically sealed, operators do not have to work in those temperatures.

### The "Always On" Paradigm and the Three Pillars of Security

Cloud service providers cannot afford to shut down datacenters for maintenance: users demand 24/7 availability. Service interruptions are perceived as exceptional events — if Google's servers go offline for even half an hour, the news makes television broadcasts worldwide.

Creating a perfect and infallible datacenter is, however, impossible, because redundancy comes at a very high cost. Companies must therefore evaluate how many copies of data are necessary based on the risk they are willing to tolerate. A momentary disconnection generates frustration but is tolerated; losing even a single file in storage is considered an unforgivable error.

> [!definition] CIA Triad — Availability, Integrity, Confidentiality
>
> Service design is based on three normative and technical pillars:
> - **Availability**: the service must always be reachable.
> - **Integrity**: data must not be corrupted or altered.
> - **Confidentiality**: data must not be accessible to unauthorised parties.
>
> The [[GDPR]] sanctions violations of **all three** pillars: not only data leaks, but also corruption and unavailability.

> [!example] La Sapienza University — violation without data theft
>
> Following a serious cyberattack that blocked services for a week without actually extracting data, La Sapienza still had to report the violation to the authorities, precisely because of the **unavailability** of its systems. In the digital world, the inability to access one's data or payment systems is equivalent to a tangible economic loss.

### Disasters, Redundancy and Geographic Regulations

To ensure continuity, infrastructures must be updatable at every point without ever interrupting the service. Regulations impose strict physical constraints: datacenters must be kept at a safety distance of at least **10 kilometres** from one another, to protect against catastrophes affecting a specific area.

> [!warning] The Brussels case
>
> A provider had placed two "distinct" datacenters within the same city square. A large fire razed both to the ground, forcing the company to inform customers of the **total loss** of their data. Geographic distance between sites is not a design preference, it is a survival requirement.

Additional legal obligations on positioning apply: the **GDPR** requires data to be kept within the boundaries of the European Union. Large cloud platforms mitigate risks by investing in their own infrastructure: Microsoft, for example, owns a global network of private oceanic cables, reducing technical dependence on third parties. To avoid *single points of failure*, these companies enter contracts with five different network operators, requiring that they do not share even a single infrastructure segment; similarly, they engage with energy suppliers who, by contract, must draw from strictly separate power plants.

> [!example] Netflix and the New Jersey tornado
>
> Despite every precaution, global-scale disasters do happen. Netflix was inaccessible for nearly a week when a devastating tornado levelled Amazon's facilities in New Jersey, depriving the entire macro-area of electricity and destroying the physical facilities.

---

## Physical Constraints and Technological Legacy

### The Physical Limits of Design (Hard Thresholds)

Unlike the work of a computer scientist — where one applies the principle of mathematical induction to scale algorithms infinitely — the engineering world involves intrinsic limitations that cannot be bypassed.

> [!definition] Hard Threshold
>
> Physical threshold beyond which a design cannot simply be "scaled up", but must be radically redesigned from scratch using different approaches.

A concrete example: in electronics a **relay** is useful for switching current on or off for common devices, but is never used in heavy systems like lifts. The enormous number of amperes required would melt the component's copper coil in seconds. This logic repeats at every scale in datacenter design: electrical cabling, cooling, load-bearing structure — each has thresholds beyond which the technology must change.

### The Legacy of Standards and the Floating Floor

Many of today's datacenter engineering solutions are based on physical standards decades old, kept alive to avoid the enormous costs of redesigning and mass-replacing them. In networking, having already laid over 70 billion metres of Ethernet (RJ45) cable, the industry preferred to push legacy wired architectures to 2.5 or 5 Gbps rather than rewiring entire buildings for modern high-speed Wi-Fi antennas. This stalemate produced a split: "campus networks" remained on standard Ethernet (often powering devices via **PoE**, *Power over Ethernet*), while the core of datacenters migrated towards faster and more stable dedicated standards.

Among the adapted technological legacies is the **floating floor**: a raised grid on crossed metal pedestals, with removable tiles laid on top.

> [!definition] Floating Floor
>
> Raised datacenter structure composed of metal pedestals and removable tiles. In old offices tiles could bear ~500 kg/m²; in modern datacenters, where storage weighs much more, panels made of compacted marble powder are used and structured to bear **1 to 1.5 tonnes per square metre**.

![Datacenter floating floor with tile lifted](https://upload.wikimedia.org/wikipedia/commons/a/a8/Tile-lifter-in-use-raised-floor.jpg)
*Source: Wikimedia Commons (Jonathan Lamb, Public Domain) — Floating floor tile lifted with a suction cup; cables and conduits are visible underneath.*

Maintaining the floating floor offers two fundamental engineering advantages:

1. **Immediate access** to all electrical and hydraulic infrastructure hidden beneath the floor, without having to move racks.
2. **Cooling conduit**: the cavity under the floor becomes an isolated plenum to push large volumes of cold air upward through perforated tiles positioned strategically in front of racks.

Keeping pathways separate beneath the floor also avoids potential short circuits: the risk of starting fires is serious, especially in the presence of lithium batteries — notoriously impossible to extinguish with standard procedures and strictly prohibited in aircraft holds for that very reason.

---

## Cooling Strategies

### Air, Water and Oil

To maximise thermal efficiency, the entire datacenter building is extremely insulated from the external environment. Creative exceptions exist: the Aruba complex, for example, freely admits cold external air in winter and expels the warm flow to the outside, saving energy.

The fundamental principle of rack cooling is the physical separation between the **cold aisle** — from which servers draw in cool air — and the **hot aisle** — from which they expel heat. This separation prevents cold and hot air from mixing before reaching the servers, maximising thermal efficiency.

![Hot aisle / cold aisle diagram with cold aisle containment](https://upload.wikimedia.org/wikipedia/commons/a/a7/Cold_Aisle_Containment.svg)
*Source: Wikimedia Commons (CC BY-SA 3.0 DE) — Cold aisle containment diagram: the blue flow (cold air) enters from below, passes through the servers and exits as a red flow (hot air) on the opposite corridor.*

Since hydraulic pipe logistics are becoming the central element of modern server farms, water is increasingly favoured for its exceptional thermal properties. Mirroring the biological mechanism of human sweating, the largest datacenters actively employ **evaporative dispersal**. Despite being an incredibly efficient technique, its environmental cost is high: about **80% of the injected water is evaporated**. Although this water returns to the atmosphere as rain without being destroyed, the massive local dispersal creates artificial pockets of humidity that risk altering the surrounding microclimate.

> [!warning] The water problem in the USA
>
> Massive agro-industrial and datacenter water use has substantially depleted resources based on the Colorado River. This has pushed some companies to plan migrating their operations to South America — Argentina in particular — in search of untouched water reserves. The water used must be desalinated and thoroughly purified, adding further logistical complexity.

A cutting-edge alternative is **total immersion cooling**: servers are submerged in tanks filled with mineral or vegetable oil. Unlike water, oil chemically isolates contacts and does not conduct electricity, dispersing heat very effectively — a technique also used for extreme overclocking in gaming PCs.

> [!note] Limits of immersion cooling
>
> - **Vegetable oil** quickly tends to go rancid, releasing sharp odours.
> - **Mineral oil** is very expensive to procure in industrial quantities.
> - Every maintenance operation on submerged motherboards becomes a messy and complex procedure.
>
> For these reasons, the industry has preferred to refine localised cooling via anti-spill pure water pipelines, accepting their technical complexity.

---

## Electrical Infrastructure

### Scale Up vs Scale Out

> [!definition] Scale Up vs Scale Out
>
> - **Scale Up**: the capacity of a single device is increased (e.g. RAM is added to an existing server).
> - **Scale Out**: the number of separate machines is replicated to distribute the load (e.g. new servers are added to the cluster).

In the context of power supply, an entire building is **forced to operate in Scale Up logic**: as infrastructure consumption increases, ever larger and thicker copper cables are required. Scaling electrically is painfully expensive, especially because of the price of copper. As an example, the cabling to support just 80 kW of peak load in a small academic complex — pulling the bundle from the external substation to the servers — cost approximately **€25,000**. To minimise intrinsic electrical resistance with very high currents, technicians replace braided cable bundles with **solid copper bars** (*busbars*), single and straight.

### Electrical Fundamentals: Power, Alternating Current and Distribution

A datacenter requires an enormous concentration of energy, which can be provided by renewables, fossil fuels or nuclear. In expanding American macro-corporate thinking, the use of latest-generation reactors is envisaged — technologically purer and safer, with radioactive materials such as thorium — generating about one-tenth of the waste compared to traditional uranium.

To manage this flow, the engineer must master the concept of **real power**, which is not simply derived from Volts, but from the product:

$$
P_{\text{apparent}} = V \times I \quad [\text{VA}]
$$

$$
P_{\text{real}} = V \times I \times \cos\phi \quad [\text{W}]
$$

where $\cos\phi$ (**power factor**) is the cosine of the phase angle between voltage and current. Modern *switching* transformers — present both in server farms and in a laptop's small power supply — have a variable power factor: it drops to about **0.56** when components are idle, but rises to **0.96** under intensive computational load.

> [!tip] Why the power factor is crucial in procurement
>
> During procurement processes for industrial equipment, demanding transformers with a high *power factor* is an economically discriminating choice: devices with a low power factor dissipate a significant portion of the energy drawn from the grid, increasing long-term operating costs.

Despite digital electronics working exclusively in **Direct Current** (*DC*), all global datacenters receive energy in **Three-Phase Alternating Current** (*AC*), standardised at **380 V** in Europe. The reasons are historical and physical: passing enormous flows of direct current heats conductors much more dangerously, and DC shocks are physiologically more lethal than AC ones at equivalent energy. Every individual server, at the last link in the chain, will convert alternating current to direct current through its own internal power supply.

> [!definition] UPS (Uninterruptible Power Supply)
>
> Component at the top of the datacenter's electrical distribution pyramid. These are not the small domestic anti-blackout devices, but **entire industrial cabinets and batteries** intended to stabilise the line and ensure continuity in the event of external grid interruption. The isolation switches — so powerful that they cannot be operated bare-handed — are activated by **pre-compressed metal springs**, which the mechanism releases automatically to guarantee maximum intervention speed and protect personnel.

> [!definition] PDU (Power Distribution Unit)
>
> Management units that progressively subdivide the electrical load along the distribution hierarchy. The power grid follows a tree architecture: from the external substation one descends through floor-level PDUs, down to **Rack PDUs** where each server plugs in its power cable.

The electrical distribution architecture therefore follows this hierarchical path:

**External Three-Phase Grid → UPS → Building PDU → Floor PDU → Rack PDU → Server**

> [!abstract] Summary — Electrical Infrastructure
>
> The electrical design of a datacenter is constrained by unavoidable physical hard thresholds: larger, more expensive and harder-to-manage cables. Current arrives from outside in three-phase AC at 380 V, is stabilised by the UPS and distributed hierarchically via PDUs to the servers, which convert it to DC internally. The power factor is a critical economic metric to evaluate when procuring equipment.

---

## Glossary

> [!definition] Scale Up / Scale Out
>
> Techniques for responding to growing resource needs. **Scale Up** increases the capacity of a single node (more RAM, more powerful CPU); **Scale Out** distributes the load across a greater number of separate nodes. In electrical terms, datacenters are forced to operate in Scale Up mode, with all the costs that entails.

> [!definition] CIA Triad (Availability, Integrity, Confidentiality)
>
> The fundamental triad on which the protective design of digital services is based. Ensuring **Availability** means the service is always reachable; **Integrity** means the data is correct and untampered; **Confidentiality** means only authorised parties can access it. The GDPR sanctions the violation of each of the three pillars.

> [!definition] Power Factor (Cosfi, $\cos\phi$)
>
> Ratio between real power (W) and apparent power (VA): $\cos\phi = P / S$. Quantifies how effectively the energy drawn from the grid is converted into useful work. A low power factor indicates energy waste. In server switching transformers, it varies between ~0.56 (idle) and ~0.96 (under load).

> [!definition] Floating Floor
>
> Raised datacenter structure on metal pedestals, with removable tiles bearing up to 1–1.5 t/m². It serves a dual purpose: easy access to infrastructure hidden beneath the floor, and creation of a plenum for distributing cold cooling air towards the racks.

> [!definition] UPS (Uninterruptible Power Supply)
>
> First stage of the power grid entering the datacenter. A large industrial system — not the small domestic device — composed of batteries and automatic switching mechanisms, designed to ensure continuity of supply and protect the entire infrastructure from grid fluctuations and blackouts.

> [!question] Possible exam questions
>
> - What are the three normative pillars (CIA Triad) and how do they apply to the datacenter context? Give an example of a violation for each.
> - Why do datacenters use alternating current (AC) despite digital electronics operating in DC? Describe the conversion path.
> - What is the power factor ($\cos\phi$) and why is it relevant when choosing equipment?
> - What are the engineering advantages of the floating floor in a datacenter?
> - Compare the three cooling approaches (air, water, oil): advantages, disadvantages and use cases.
> - Explain the difference between Scale Up and Scale Out and why the electrical infrastructure imposes a forced Scale Up approach.
> - Why must datacenters be geographically separated? What happens if this constraint is ignored?
