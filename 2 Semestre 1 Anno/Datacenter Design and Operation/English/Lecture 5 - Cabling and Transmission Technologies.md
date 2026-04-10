
# Cabling and Transmission Technologies

Designing and managing a modern data center requires a holistic vision embracing different disciplines: from thermodynamics to hydraulics, through to optical and electrical transmission technologies. This lecture recalls the cooling strategies already introduced, deepens infrastructure life cycle management, and addresses network cabling technologies that constitute the true "nervous system" of the data center.

## Cooling and Electrical Distribution

### From CRAC to In-Row Cooling

Heat dissipation is inextricably linked to electrical distribution. In traditional architectures, cooling was based on **CRAC** units (*Computer Room Air Conditioning*), which drew warm air from the ceiling to re-introduce it cold through a floating floor. This architecture has severe limitations related to **power density**: it works efficiently only up to 5–8 kilowatts per rack, a threshold beyond which cooling the entire room becomes impractical and energetically disadvantageous.

Today's standard favours **in-row cooling**, a system that delocalises cold production relative to the point of use, exploiting water as a thermal transport medium. Each unit is equipped with sensors and actuators manageable digitally via HTTP and Web APIs, allowing the fan speed to be dynamically increased only in areas where heat peaks are detected, without having to overcool the entire infrastructure.

### Liquid Cooling: Opportunities and Risks

When power density exceeds the capabilities of air, **liquid cooling** becomes a necessity. This introduces concrete risks, however: common mineral water conducts electricity, creating a real risk of short circuit and fire in racks drawing from 30 to 200 kilowatts. Alternatives present their own trade-offs: mineral oil is thermally efficient but expensive and difficult to manage, while glycol-based fluids offer high thermal capacity and do not conduct electricity, but at a significant cost.

> [!warning] Hydraulic management in data centers
>
> Water cooling circuits operate analogously to a car radiator, introducing new variables for IT teams: the build-up of mineral deposits that clog small-diameter tubes, pressure management to ensure correct flow without ruptures, and the need for specialist plumbers — a professional figure historically foreign to the IT world.

The electrical distribution also follows a tree structure, with a main line subdividing down to individual racks. The design includes **overbooking** (over-allocation) margins that allow electrical switchboards to be upgraded in the future without having to rewire the entire infrastructure. If fifteen years ago 15 kW per rack was sufficient for **HPC** (*High Performance Computing*), today the minimum threshold for high-performance installations starts at 30 kW and above.

---

## Data Center Life Cycle and Horizon Effect

A data center is not designed for a few years: its time horizon is typically a decade or more. When defining an architecture, physical boundaries are established that are difficult to overcome afterwards, making the estimation of future power density a strategic decision with long-term consequences.

> [!definition] Horizon Effect
>
> The inability to see beyond immediate needs during the design phase, which forces costly premature redesigns. An overly conservative estimate of future power density can make a data center obsolete within a few years of opening.

A virtuous example is the university data center at San Piero a Grado (Pisa). Designed with a power of 15 kW/rack, it served excellently for ten years. In the subsequent expansion, a hybrid **air-to-liquid** solution was adopted — using outside air to cool the liquid destined for the CPUs — allowing it to simultaneously support air-cooled and liquid-cooled servers for advanced research.

To govern infrastructures with thousands of objects over such long time horizons, inventory software is essential. Open-source tools such as **RackTables** allow the precise mapping of rack positions, tracking error logs and managing multi-brand databases, guaranteeing granular control over the entire infrastructure.

---

## Fundamentals of Optical Fibre Transmission

The network is the vascular system of the data center and responds to insurmountable physical constraints: there is a maximum capacity beyond which one cannot go. Unlike copper cables — in which electrons travel at about 60% of the speed of light ($0.6c$) — **optical fibre** exploits light itself, reaching significantly higher propagation speeds and eliminating thermal dissipation along the route.

### The Principle of Total Internal Reflection

The operation of optical fibre is based on the laws of geometric optics. When a ray of light strikes the separation surface between two media, it partially reflects and partially refracts. **Fresnel's law** establishes that the angle of reflection equals the angle of incidence. If, however, the angle of incidence exceeds a certain **critical angle**, refraction ceases entirely and *total internal reflection* is obtained: this is exactly the phenomenon that traps photons inside the cable.

The fibre core is coated with a reflective mineral material called **cladding**, which forces photons to bounce off the edges without escaping. For transmission, ordinary white light is not used — which a prism would separate into different wavelengths — but rather a **laser**, capable of generating a uniform and coherent beam of photons.

> [!note] Signal degradation and external factors
>
> At each bounce, a microscopically small fraction of photons is lost due to the inevitable imperfections of the material, imposing practical limits on transmission distances. External factors can worsen the degradation: in the metropolitan network of Pisa, a cable buried on the Ponte di Mezzo was progressively compressed by urban bus traffic, altering the internal geometry of the fibres and causing severe signal degradation — resolved only with costly road excavations.

---

## Types of Optical Fibre

### Single-mode and Multi-mode

Not all optical fibres are equal, and the incompatibility between laser sources and cable type makes correct category identification fundamental.

**Single-mode fibre** has an extremely thin *core* — about 9 micrometres — that allows a single straight path for light, minimising bounces and therefore signal dispersion. This permits covering immense distances, from 10 to 40 kilometres, typically operating at wavelengths of 1310 nm or 1550 nm. It is certified by the **OS** family of standards. Although more expensive and complex to produce, it guarantees unparalleled stability: it is the preferred choice not only for metropolitan and geographic links, but also — in excellence installations such as the University of Pisa — within the data center itself.

**Multi-mode fibre** has a wider *core*, up to 50 micrometres, that allows photons to follow multiple paths. This increases the number of bounces and consequently signal loss, making it suitable for short distances. It operates at wavelengths around 850 nm and is certified by the **OM** family of standards. It is the predominant choice within most data centers due to lower production costs.

Recently **silicon cables** (*Silicon Photonics*) have been introduced: not being pure glass, they are even more economical, although they cover lower bandwidths and distances. They are often found in **FTTH** (*Fibre To The Home*) home networks, where the maximum requirement is typically 1 Gigabit per second.

| **Standard** | **Type** | **Core** | **Max Distance (1 Gbps)** | **Notes** |
|---|---|---|---|---|
| **OM1** | Multi-mode | 62.5 µm | 275 m | 10G/40G not supported |
| **OM2** | Multi-mode | 50 µm | 550 m | Progressive improvement |
| **OM3** | Multi-mode | 50 µm | 1000 m | Optimised 850 nm laser |
| **OM4** | Multi-mode | 50 µm | 550 m at 10G | Modern standard in DCs |
| **OS1 / OS2** | Single-mode | 9 µm | 10–40 km | Maximum distance, metropolitan and excellence DC use |

---

## Optical Connectors

### SC, LC and Simplex Architecture

A cable, to be useful, must terminate in a connector. There are two macro-categories: **passive connectors**, simple mechanical terminals, and **active connectors**, which include an electronic **transceiver** that powers and converts the signal for the host device.

Among passive connectors, the **SC connector** is relatively bulky in size, but offers a large contact surface that minimises light dispersion at the junction point. It is the standard of choice for metropolitan cabling and geographic links managed by telephone *carriers*. The **LC connector** is instead extremely compact and optimised for high port density: it is the absolute standard within data center racks. The old FC and ST standards are today to be considered obsolete.

> [!tip] The directionality of optical fibre
>
> Unlike copper cables, a single optical fibre strand carries light in only one direction. To have full-duplex communication, a **pair of cables** is always required: one to transmit (TX) and one to receive (RX). LC connectors are often coupled via a plastic mechanical clip that can be removed to swap the two connectors when it is necessary to correctly match the TX and RX channels at the two ends of the connection.

---

## Copper Cables and Ethernet Standards

### The Dominance of Copper

Although there are dedicated HPC networks such as **InfiniBand**, Ethernet on copper cables still plays a dominant role, with over 70 billion metres of cable installed worldwide. Modern Ethernet cables use the **RJ45** connector, which about thirty years ago replaced the old RJ11 telephone connector. Inside the sheath are 8 wires twisted into 4 pairs: 10 and 100 Megabit networks used only two, historically making the transition to 1 Gigabit simple — it was sufficient to enable the two pairs left unused inside the same cable.

Manual assembly involves inserting the wires according to a precise colour scheme into the RJ45 plug and using a crimping press, which forces small "blade" metal contacts to cut through the individual wire sheaths, establishing the electrical connection.

### The 10 Gigabit Challenge

Surpassing the 1 Gigabit barrier over copper proved to be a titanic engineering challenge, linked to electromagnetic interference and the high frequencies required. The following table summarises the evolution of standards:

| **Category** | **Max Practical Capacity** | **Max Frequency** | **Notes** |
|---|---|---|---|
| **Cat 5 / 5e** | 1 Gbps | 100 MHz | Historic and widespread standard |
| **Cat 6** | 1 Gbps (10G at short range) | 250 MHz | Most installed; 10G over Cat 6 is discouraged |
| **Cat 6A** | 2.5 / 5 / 10 Gbps | 500 MHz | Maximum practical evolution; recommended for 10G over copper |
| **Cat 7 / Cat 8** | Over 10 Gbps | Over 600 MHz | Rarely adopted; cables are extremely thick and rigid |

> [!warning] The cost of redundancy and limits of Cat 7/8
>
> Cat 7 and Cat 8 are theoretically available but practically unusable: the rigidity of the cables makes them like "iron pipes", making physical cabling impractical in crowded environments. On the economic side, two high-performance copper cables — necessary to guarantee the vital redundancy of a server and eliminate the *single point of failure* — can weigh on the budget up to $300: a non-trivial cost multiplied across hundreds of servers.

---

## Cable Management

Rationally managing cabling (*cable management*) is not an aesthetic matter: it is a critical functional necessity with a direct impact on energy efficiency. The tendency to aggregate bandwidth has led to up to 4 cables per server. In a cabinet containing 30 servers, this translates to over 120 cables. If one adds the common practice of ordering cables in standard lengths — for example 2 metres — to standardise stock, the result is a significant accumulation of excess cable.

If cables are not routed in orderly paths, they form a physical barrier at the rear of the rack that obstructs the hot air extraction vents produced by the servers. The cooling system detects the abnormal temperature rise and forces the fans to accelerate drastically, resulting in energy waste and premature hardware wear. Disorderly cabling, in other words, degrades the thermal performance of the entire rack.

> [!abstract] Lecture summary
>
> The modern data center is a tightly integrated system in which cabling, cooling and electrical distribution choices condition one another. The transition from air cooling to liquid cooling follows the growth of power density per rack. Optical fibre is the dominant transmission medium for distances greater than a few metres, with the choice between single-mode and multi-mode depending on distance and budget. Copper remains relevant for final links, but Cat 6A is today the practical limit beyond which fibre becomes mandatory. Cable management, finally, is not a detail: disorderly cabling degrades the thermal performance of the entire rack.

> [!question] Possible exam questions
>
> - What are the practical limits of CRAC systems and why does in-row cooling overcome them?
> - What is the Horizon Effect in a data center and how is it mitigated during the design phase?
> - What is the physical difference between single-mode and multi-mode fibre? In which context is each preferred?
> - Why is a single optical fibre strand insufficient for full-duplex communication?
> - Which category of Ethernet cables is recommended for 10 Gigabit, and why are Cat 7/Cat 8 not practically adopted?
> - How does disorderly cabling affect the thermal efficiency of a rack?
