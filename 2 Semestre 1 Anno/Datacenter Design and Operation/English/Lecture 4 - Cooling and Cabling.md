# Density, Heat and Advanced Cooling in Modern Data Centers

The data center industry is undergoing a radical transformation driven by an incontrovertible physical factor: the power density per rack has increased by an order of magnitude over the course of a decade. Understanding this transformation requires starting from the physical fundamentals of silicon and then working up to the architectural and infrastructural choices that follow.

## Physical Limits of Silicon

### The Frequency Wall

The semiconductor industry is confronting physical constraints that cannot be circumvented through engineering alone. Lithographic processes have been pushed down to 2 nanometres, a scale at which quantum effects and thermal interference become critical. On the clock frequency side, CPUs have reached a practical plateau around 2–4 GHz: going beyond generates an unsustainable number of logic errors and electromagnetic emissions that begin to approach the spectrum of X-rays.

> [!warning] The frequency plateau
>
> Increasing clock frequencies beyond 4 GHz is not simply difficult — it is physically dangerous for transistor integrity and produces unacceptable electromagnetic interference. This explains why the industry shifted to multi-core parallelism instead of pushing for higher single-core speed.

The industry's response has been parallel computing: increasing the number of cores rather than the speed of each individual core. However, this solution does not solve all problems. There are classes of workloads — such as Monte Carlo simulations at CERN — that are not parallelisable by nature and still require a few CPUs at very high frequency. This creates a niche but crucial market for high-performance single-core processors.

---

## The Impact of Artificial Intelligence on Data Centers

### TDP and Extreme Density

The explosion of deep neural networks and **Large Language Models (LLMs)** has completely redefined the design parameters of data centers. **TDP** (*Thermal Design Power*) indicates the maximum heat that a cooling system must be able to dissipate to keep the processor within operating limits. This parameter has grown exponentially: modern top-of-the-range CPUs reach 500 Watts, while next-generation GPUs (such as NVIDIA Blackwell architectures) exceed 1 Kilowatt per single processor.

The density reached by modern nodes is extraordinary: a single AI server equipped with a CPU, multiple GPUs and high-speed storage can contain up to 18 trillion transistors in a volume of a few rack units.

> [!note] Co-evolution of the system architecture
>
> To power these processors without bottlenecks, the entire architecture has evolved in a coordinated way. The PCIe standard has rapidly updated its specifications, and the introduction of non-volatile memories (NVDIMM) and ultra-fast SSDs has practically dissolved the boundaries between the old levels of the traditional memory hierarchy, making the very concept of "hierarchy" less rigid than it was a decade ago.

---

## The Transition to Liquid Cooling

Due to the transistor density reached by modern processors, air — which is a mediocre heat conductor and behaves effectively as a thermal insulator — is no longer physically able to dissipate heat within racks. The industry has undergone a structural transition to **liquid cooling**, which is articulated in several technologies with distinct characteristics and trade-offs.

### Direct-to-Chip

The **Direct-to-Chip** system brings the cooling fluid into direct contact with the heat-generating components, almost entirely eliminating fans. Solutions such as *Lenovo Neptune* use networks of tubes and plates in pure copper placed directly on CPUs, GPUs and RAM banks. The result is significantly superior thermal efficiency compared to air systems, with a meaningful reduction in noise and fan energy consumption.

### Waterless Cooling

To avoid the catastrophic hardware damage caused by potential water leaks — a real possibility in environments operating 24/7 — systems based on **dielectric fluids** such as glycol have been developed. These liquids, unlike water, do not conduct electricity: an accidental spill does not cause short circuits or fires. Their physical properties also allow the use of much thinner tubing compared to traditional hydraulic circuits, simplifying routing inside the rack.

### Immersion Cooling

The most radical method involves physically submerging servers in tanks of dielectric **mineral oil**. The TACC computing centre in Texas is one of the best-known examples of this technology. Thermal efficiency is extreme: the oil uniformly surrounds every component, eliminating any localised hot spot. The price to pay is operational complexity: the cost of oil is high, maintenance requires dedicated procedures and every extracted component is inevitably coated in oil.

> [!tip] Choosing the cooling method
>
> The choice between Direct-to-Chip, Waterless and Immersion Cooling is not only technical but also economic and operational. Direct-to-Chip is the most balanced compromise for most HPC data centers; Immersion Cooling is justified only for extreme densities and uniform, predictable workloads.

### Hydraulic Maintenance

The introduction of water circuits transforms the skill profile required of management teams. Common mineral water tends to deposit calcium scale that risks clogging pipes with diameters in the order of a few centimetres. Circuits must undergo rigorous gas pressurisation tests before activation to rule out leaks, and require periodic intervention from specialist plumbers — a professional figure entirely foreign to the traditional IT world.

---

## The Energy Challenge and AI Factories

### Industrial-Scale Consumption

Infrastructures dedicated to training AI models can no longer be classified as simple data centers: they are true **AI Factories**, industrial plants requiring power on the order of 1–2 Gigawatts. To give a concrete measure, 1 Gigawatt corresponds to the electrical consumption of a city of approximately one million inhabitants.

> [!example] Water consumption from AI training
>
> To dissipate the heat produced during the training of a single large model, evaporative cooling towers consume quantities of water comparable to entire Olympic swimming pools. Although evaporation does not produce chemical pollutants, it removes water resources from natural and agricultural reservoirs — a problem particularly acute in areas already subject to water stress such as the Colorado River basin.

### Geopolitical Limits and Grid Saturation

In Europe, and particularly in Italy, the main obstacle to the expansion of AI Factories is not the technology but the availability of electricity. The electrical grid of northern Italy has reached saturation levels such that almost no capacity remains for new high-density installations. This constraint pushes industrial players to identify unconventional sites: decommissioned heavy industrial areas — such as some parts of Bari — are particularly attractive because they already have the high-voltage electrical grid infrastructure needed to support the required Gigawatts, without having to proceed with costly upgrades to the distribution network.

> [!question] Possible exam questions
>
> - Why has the semiconductor industry abandoned increases in clock frequencies in favour of multi-core parallelism? Which workloads remain penalised by this choice?
> - What are the operational differences between Direct-to-Chip, Waterless and Immersion Cooling? In which scenario is each preferred?
> - Why is the construction of new AI Factories in Italy constrained by the electrical grid rather than land availability or technology?
> - What is TDP and why has it become the critical parameter in designing modern AI data centers?
