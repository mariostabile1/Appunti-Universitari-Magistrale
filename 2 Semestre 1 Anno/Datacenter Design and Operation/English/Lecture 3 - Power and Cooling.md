# Power and Cooling Management in Data Centers

The infrastructure of a modern Data Center is an extremely complex ecosystem, comparable to an orchestra in which every element — power supply, networks, cooling and racks — must work in perfect harmony with the others. In this chapter we explore the physical, economic and managerial dynamics governing power and cooling: from the vendor market and maintenance criticalities, to the risk of overheating, to energy efficiency metrics such as **PUE** (*Power Usage Effectiveness*), and finally to the evolution of air conditioning architectures with **CRAC** systems.

---

## The Vendor Market and Procurement Strategy

Purchasing individual parts of a Data Center to assemble them independently is a considerable strategic mistake. Since every component directly affects the others, it is essential to rely on integrated ecosystems provided by specialised vendors. Today every part of the Data Center can be considered "active": even passive elements such as racks and control boards are equipped with displays and interfaces accessible via HTTP.

There are few suppliers in the world capable of managing the skills required for high-power installations. **Schneider Electric** dominates electrical installations and has acquired **APC** (a historic company specialising in racks and UPS units). Another giant is **Vertiv** (formerly Emerson), born as a manufacturer of *chillers* (industrial refrigeration units). In Italy the market is served by **Riello** for electrical systems and UPS, and **Climaveneta** for chillers.

---

## Maintenance and the Life Cycle

When purchasing a Data Center, one is not merely buying hardware: one is "marrying" the manufacturer through a maintenance contract. The business model is analogous to that of high-end cars — BMW or Mercedes — where access to control units and repairs require proprietary tools that only the manufacturer can provide.

> [!warning] Infrastructure costs
>
> Costs are substantial: the refurbishment of a small DC starts at approximately **€250,000–300,000**, while larger facilities range between **€2 and €5 million**. Buying a spare part to keep in stock — for example a chiller costing **€100,000** — is economically unsustainable and logistically impractical due to its physical size.

Maintenance contracts guarantee "Next Business Day" or within **4-hour** interventions, with 24/7 remote monitoring. The annual cost of these contracts ranges between **10% and 30%** of the original investment. If one decides not to renew, after several years vendors no longer guarantee parts availability — especially for machines over **12 years** old — effectively forcing the operator into a costly full upgrade. The typical life cycle of an entire Data Center is around **10 years**.

---

## Overheating: A Critical Threat to Silicon

Conceptually, a Data Center requires the same resources as a domestic dwelling: network, power, cooling. The difference is in scale: whereas a domestic meter delivers **3–5 kW**, a Data Center easily needs **1 Megawatt** to power thousands of servers.

This massive energy demand translates into an enormous release of heat. If the cooling system fails, the situation deteriorates rapidly: server fans, equipped with thermal sensors, detect the rising temperature and speed up to compensate. But by moving already warm air, their own friction introduces additional thermal energy into the room. An **exponential effect** is triggered: without cold air, the room temperature can reach **60–70°C** in a very short time, to the point where the only emergency solution is to physically open the building's doors.

> [!warning] Overheating vs blackout
>
> Overheating has historically been more dangerous than a simple power outage. A traditional blackout could damage mechanical hard drives — the head risked scratching the platter at the moment of an abrupt stop — but with the spread of **SSDs** this mortality has drastically reduced. Extreme heat, however, can melt plastic components and irreparably damage the **silicon** of electronic components, rendering them unusable.

> [!example] The thermodinamics of a rack at 40°C
>
> Some vendors certify that servers, with an average useful life of **5 years**, can operate for **40 days per year at 40°C**. This seems like a high limit, but one must consider the rack's **Delta T**: the temperature difference between the cold inlet air and the air expelled at the rear is typically about **15°C**. If the inlet air is at 40°C, the rear of the rack easily reaches 55–60°C. Mechanical disks for *cold storage* (inexpensive archiving) can reach operational surface temperatures of **100°C** — literally hot enough to cook on.

> [!example] The Pisan chiller failure
>
> During a July week with external temperatures exceeding **38°C**, the old chillers in the Pisan Data Center — designed in years when those peaks were not recorded — went into thermal shutdown because their dimensions did not allow dissipation of such a load. In just **two hours** the machine room reached **50°C**, forcing everything to be shut down and literally pouring cold water on the chiller housings to restart them. This episode demonstrates how vulnerable Internet services are to the physical conditions of the environment.

---

## Electrical Anomalies and "Bugs"

Not only heat, but also power interruptions cause serious disruptions. **UPS** (*Uninterruptible Power Supply*) units intervene during micro-interruptions to buy time for diesel generators to start. In one specific case, continuous **10–15-second** micro-interruptions were not tolerated by the control electronics of the chillers — which were not put on UPS for cost reasons. The chillers, interpreting that disturbance as a danger, shut themselves down autonomously, causing three refrigeration systems to fail. The solution was to purchase a UPS dedicated exclusively to the chillers to stabilise their supply.

> [!example] The origin of the term "bug"
>
> A snake seeking warmth in winter slithered into a high-voltage distribution cabinet, attaching itself to the coils and causing a fatal short circuit that shut down the entire Data Center — leaving only its own charred body behind. The computing term **"bug"** derives from exactly this type of incident: real insects and rodents that caused blackouts by chewing cables or short-circuiting components.

---

## Energy Efficiency and PUE

Wasting energy not only causes direct economic losses, but imposes indirect costs: thicker and more expensive electrical components and cables to handle the peaks. In a scenario where energy costs have gone from **€300,000 to €1,000,000 per year** — partly due to the conflict in Ukraine — measuring efficiency has become crucial.

> [!definition] PUE — Power Usage Effectiveness
>
> **PUE** is the standard dimensionless parameter for measuring the energy efficiency of a Data Center. It is calculated as:
>
> $$
> \text{PUE} = \frac{\text{Total Facility Power}}{\text{IT Equipment Power}}
> $$
>
> *Total Facility Power* is the sum of IT consumption, lighting and — above all — cooling expenditure. Since IT absorption appears both in the numerator and denominator, the ideal theoretical value in the absence of overhead is **1**. PUE is always greater than 1: the higher the value, the worse the efficiency. Some use the inverse of the formula to obtain an index between 0 and 1, but the concept does not change.

| PUE Value | Interpretation |
|---|---|
| ≤ 1.2 | Excellent — modern standard |
| < 1.3 | EU-recommended for Hyperscale DCs |
| 1.15 | Annual PUE of University of Pisa Green DC at full load |
| 1.8 – 2.0 | Italian average — very poor |
| > 2.0 | For every kW of IT, more than 1 kW is spent just cooling it |

### The Pitfalls of PUE

Although essential, PUE can hide misleading information. The first question to ask when looking at any declared value is: *"Over what time period and under what conditions was it measured?"*

> [!warning] Traps in comparing PUE values
>
> **Seasonal impact.** Cooling in winter costs much less than in summer. Displaying an instantaneous PUE measured in January (e.g. 1.05) is "cheating". A **yearly PUE** must be calculated, averaging the seasonal curve over the entire year. In Europe the peak consumption is in summer; in Australia — with seasons shifted by six months — the peak falls between December and January.
>
> **Data Center saturation.** A nearly empty DC (e.g. with a single server) may show a poor PUE because the chiller compressor runs "idle" relative to the IT load. Conversely, the same DC full will work at its designed efficiency.
>
> **Facility size.** For a small rack of 5 servers with an annual budget of **€1,000,000**, it might be economically more advantageous to install a normal domestic air conditioner — accepting a PUE of 2.0 — rather than face the multi-million euro investment of an optimised industrial system.

---

## Orbital Data Centers: The Perfect PUE

The obsession with PUE has driven companies to design **Orbital Data Centers in space**. In the absence of atmosphere, heat dissipation and solar energy absorption are enormously facilitated, allowing a theoretical PUE approaching **1** (realistic estimated values: 1.01–1.07). These systems would have a life cycle of **10–15 years**, at the end of which modules would be de-orbited to burn up in the atmosphere or — ideally — recovered and replaced.

> [!note] Latency in orbital communications
>
> For *point-to-point* communications, latency would not be an insurmountable problem: light travels distances in air almost instantaneously, while it travels at approximately **0.6c** in copper cables on the ground. The current real challenge remains the amount of debris in orbit (*orbital debris*), which makes any stable installation dangerous.

---

## The Air Architecture: CRAC Systems

Since water is a significantly superior thermal conductor to air — which in fact behaves as an excellent thermal insulator — air cooling is inexpensive but decidedly suboptimal. In the 1990s machine room temperatures were maintained at around **17°C**; today the operational standard is **26–27°C**. In 2000, in the Tiscali data center in Sardinia, a rack consumed just **3 kW** and could be cooled by keeping the windows open. With rising power densities (*High Density Computing*), more sophisticated architectures became necessary.

> [!definition] CRAC — Computer Room Air Conditioning
>
> The **CRAC** is the first industrial air cooling architecture, still in use in classic facilities — such as some Aruba buildings, with predictable web server loads of about 300 W per server. It features vertical machines placed in the room, a **raised floor** (*floating floor*) and **perforated tiles** positioned carefully in front of racks.

The underlying physical principle is **convection**: warm air naturally tends to rise. The operating cycle is as follows: the CRAC machine draws warm air from the upper part of the room, generates cold air and forces it below the floating floor. The cold air travels in the underlying *plenum* and rises only through the perforated tiles placed in front of the racks. It is then drawn in by the front fans of the servers, cools them, and exits as warm air at the rear, rising towards the ceiling to begin the cycle again.

> [!tip] The golden rule of CRAC
>
> **Never mix hot air with cold air.** Mixing the two flows would nullify the thermodynamic work spent generating the cooling, drastically lowering efficiency. In some specialised DCs such as those of Aruba, racks have a front glass blast-proof door with ventilation grilles positioned *inside* — between the glass and the servers — to channel the cold flow directly into the IT without dispersal.

> [!warning] The limit of the CRAC architecture
>
> With today's power densities — up to tens of kW per rack — the physical space becomes saturated. If the ceiling is not high enough and the *plenum* under the floor is not sufficiently deep, the volume of hot air becomes so imposing that convection is overcome: the hot air descends back onto the servers, sending the system into thermal shutdown. The CRAC architecture therefore encounters a structural limit in modern *high-density* installations.

---

> [!abstract] Summary
>
> Power and cooling management in a Data Center intertwines thermodynamics, economics and systems engineering. **PUE** is the standard tool for measuring efficiency, but must always be interpreted on an annual basis and in relation to actual load — an instantaneous value is almost always misleading. Cooling architectures, from traditional **CRAC** systems to more modern solutions, must address growing power densities while maintaining a clear separation between hot and cold air flows. *Vendor lock-in* and maintenance contracts represent the true hidden cost of an infrastructure that, by its nature, can never stop.
