 # Thermally-Managed Edge Inference Module

A custom active cooling solution for an NVIDIA Jetson Orin Nano Super, designed in SolidWorks, simulated in Flow Simulation, CNC machined, and tested against the stock cooler under sustained AI inference.

<img src="photos/main.png" width="700" alt="Assembled thermal module">


---

## Results

Sustained ResNet-50 FP16 inference at the Orin's 25 W power mode, run for 18 minutes on the stock cooler and 30 minutes with each custom configuration to steady state:

| Metric | Stock cooler | Custom — auto fan | Custom — max fan |
| --- | ---: | ---: | ---: |
| Fan control | Governor (auto) | Governor (auto) | Manual state 3 |
| Ambient temperature (assumed) | 25 °C | 25 °C | 25 °C |
| Steady-state junction temperature | 76.48 °C | 63.02 °C | **59.03 °C** |
| Board power (VDD_IN) | 20.85 W | 21.27 W | 21.32 W |
| Effective thermal resistance, using VDD_IN | 2.47 °C/W | 1.79 °C/W | **1.60 °C/W** |
| GPU clock under load | 909 MHz | 1006 MHz | **1007 MHz** |

With automatic fan control, the custom cooler reduced steady-state junction temperature from **76.48°C to 63.02°C**, a **13.46°C improvement** over the stock cooler. Effective thermal resistance decreased by **27.6%**, despite slightly higher board power: 21.27 W versus 20.85 W.

At maximum fan speed, junction temperature fell to **59.03°C**, a **17.45°C reduction** from the stock baseline. Effective thermal resistance decreased by **35.4%**, from 2.47°C/W to 1.60°C/W. This additional cooling came with noticeably higher fan noise. The stock cooler was quieter, while the custom cooler was nearly inaudible under automatic fan control. Noise comparisons are based on listening rather than sound-level measurements. Automatic fan control was preferred for normal operation because it was substantially quieter, while maximum fan speed reduced junction temperature by a further 3.99°C.

Effective thermal resistance was calculated as:

`R_eff = (T_junction − T_ambient) / VDD_IN`

This calculation uses total board input power, providing a consistent comparison between the tested cooling configurations rather than an isolated chip-to-air thermal resistance.

![Heat sink configuration comparison](photos/heatsinkcomparison.png)

*Figure 1. Junction temperature over the first 18 minutes of each load test. The stock run lasted 18 minutes; each custom-cooler run continued to 30 minutes. The early dip in the custom automatic-fan trace coincided with an inference-workload restart. Ambient temperature was assumed to be 25°C.*

---

## What this is

A custom cooling assembly for an NVIDIA Jetson Orin Nano Super, designed, simulated, manufactured, and tested against the stock cooler. Sustained AI inference in the 25 W power mode provides a repeatable workload for evaluating thermal performance.

The assembly includes:

- A CNC-machined aluminium heatsink with a tapered pedestal at the bare-die interface
- An MJF PA12 enclosure that directs airflow from the intake, through the fin array, to the exhaust
- A spring-loaded mounting system that applies preload at the die interface

A SolidWorks Flow Simulation model was used to compare fin configurations before fabrication. Physical testing then measured operating temperatures, board power, and GPU clock frequency.

The project focuses on the complete mechanical and thermal development process: selecting a geometry, designing for manufacture, assembling the hardware, and comparing simulated performance with experimental results.

---

## Design

### Preliminary sizing calculations

Simplified hand calculations were used to establish a starting geometry before CFD. Fin efficiency was estimated using an assumed convection coefficient to assess fin thickness:

$$
\eta_f = \frac{\tanh(mH)}{mH},
\qquad
m = \sqrt{\frac{2h}{kt}}
$$

Here, $H$ is fin height, $t$ is fin thickness, $k$ is aluminium thermal conductivity, and $h$ is the convection coefficient. Fin height was set to **25 mm** by packaging constraints. A thermal conductivity of **167 W/(m·K)** was used for 6061 aluminium, with an initial estimated convection coefficient of **25 W/(m²·K)**.

For **1 mm fins**, these inputs give an estimated fin efficiency of **94%**, supporting their use as the starting thickness for the CFD sweep. The approximation assumes a straight, uniform fin with uniform convection and negligible heat loss from its tip.


### Heatsink and CFD study

The heatsink contacts the Orin Nano’s exposed die through a thermal interface material. A **5 mm aluminium base with a tapered pedestal** conducts heat from the die footprint into the wider fin array.

Fin geometry was evaluated across **12 configurations**: 16, 18, 20, and 22 fins, each at three centre-to-centre pitches. Base width remained fixed at 55 mm, fin height at 25 mm, fin length at 67.9 mm, and fin thickness at 1 mm. Each configuration was simulated with a **22 W heat load**.

The lowest simulated maximum die temperature was **52.26°C**, obtained with **18 fins at 3.0 mm pitch**, corresponding to a **2.0 mm clear gap**. This configuration was selected for fabrication.

![Simulated temperature versus fin pitch for 12 heatsink configurations](photos/fincomparison.png)

*Figure 2. CFD comparison of 12 fin configurations at a 22 W heat load. The bottom axis shows centre-to-centre fin pitch; the top axis shows clear air gap. The circled point identifies the lowest simulated temperature.*

The 22-fin configurations produced higher temperatures despite their greater surface area. This suggests a tradeoff between heat-transfer area and airflow restriction, although confirming the cause requires comparing flow rates and pressure drops. The selected design was the best configuration tested; the sweep did not establish a global optimum.

The CFD-reported average convection coefficient of approximately **25 W/(m²·K)** was consistent with the initial sizing assumption. Estimated fin efficiency was **94% for 1 mm fins**, compared with **96% for 1.5 mm fins** at the same convection coefficient.

This check supported retaining 1 mm fins. A separate thickness sweep would be needed to assess the combined effects on conduction, channel width, and airflow.

The heatsink was left as-machined to preserve the specified interface dimensions and avoid adding a coating at the die-contact surface.

### Mount and interface

A spring-loaded mounting assembly with M2 screws provides a **calculated average die-contact pressure of approximately 30 psi**, based on total spring preload divided by die contact area. Published spring force–deflection data were used to estimate preload at the installed compression.

The pressure resulted from the selected mounting hardware rather than an independently optimized target. Actual pressure distribution depends on alignment and assembly tolerances.

Arctic MX-6 thermal paste forms the interface between the die and the heatsink pedestal.

### Enclosure

The enclosure was manufactured in MJF PA12 by JLC3DP. It locates the fan and defines the airflow path from the intake, through the fin array, to the exhaust.

The intake grille uses obround slots to increase open area within the available footprint. Open-area fractions calculated from the CAD layouts were approximately **45–56% for the round-hole patterns** and **68–77% for the obround-slot patterns**. These values describe the layouts evaluated, rather than universal limits for either shape.

The obround layout was selected for its greater open area. Pressure loss was not established from open area alone.

---

## Simulation vs. measurement

| Experimental condition | Measured temperature | Difference from 52.26°C |
|---|---:|---:|
| Custom — automatic fan | 63.02°C | +10.76°C |
| Custom — maximum fan | 59.03°C | +6.77°C |

The selected configuration produced a simulated maximum die temperature of **52.26°C** at a 22 W heat load. Experimental temperatures were higher under both fan settings.

These results are not yet a matched validation of the model. Inlet-air temperature was assumed rather than measured, and the installed fan operating point and interface resistance require further verification. The simulated maximum die temperature must also be compared with the specific temperature sensor used in the experimental analysis.

---

## Measurement method

Testing was performed on the Jetson using a ResNet-50 FP16 TensorRT engine. The same engine and inference settings were used for all cooling configurations.

### Workload and logging

Build the engine once:

```bash
/usr/src/tensorrt/bin/trtexec \
  --onnx=resnet50_b8.onnx \
  --saveEngine=resnet50_b8_fp16.engine \
  --fp16
```

Confirm that the same 25 W power mode is active before each run:

```bash
sudo nvpmodel -q
```

Start logging at one-second intervals, using a separate filename for each configuration. Capture a few minutes of idle operation before starting inference:

```bash
sudo tegrastats --interval 1000 --logfile custom_auto.txt
```

In a second terminal, run the inference workload:

```bash
/usr/src/tensorrt/bin/trtexec \
  --loadEngine=resnet50_b8_fp16.engine \
  --duration=1800 \
  --infStreams=2
```

The stock-cooler test ran for 18 minutes (`--duration=1080`); the custom-cooler tests ran for 30 minutes (`--duration=1800`).

Two inference streams were used consistently across all runs. In this setup, that workload produced approximately 21 W of reported board power. The 25 W power-mode setting does not imply a constant 25 W heat load.

### Fan settings and steady state

The stock cooler was tested under automatic fan control. The custom cooler was tested under both automatic control and maximum fan speed.

Steady state was defined as a junction-temperature drift magnitude below **0.1°C/min**. Reported temperature and board-power values were averaged over the final **five minutes** of each run.

Ambient temperature was assumed to be **25°C**, rather than measured. Effective thermal resistance was calculated using this assumption and the reported VDD_IN board power.

## Limitations and next steps

- **Measure inlet-air temperature.** The assumed ambient temperature introduces uncertainty into the thermal-resistance estimates.
- **Reconcile simulation and measurement.** Compare equivalent heat loads, fan operating conditions, and temperature quantities before assessing model accuracy.
- **Repeat the tests.** Additional runs are needed to quantify repeatability and the effects of mounting and thermal-paste application.
- **Measure acoustics.** Current noise comparisons are subjective; sound-level measurements would quantify the cooling–noise tradeoff.

---

## Repository layout

```
cad/                 SolidWorks assembly, part files, drawings
drawings/            engineering drawings of components
photos/              product photos
simulation/          simulation results
```

---

## Hardware

| Item | Part |
|---|---|
| Compute | Jetson Orin Nano Super Developer Kit, JetPack 6.2.1, L4T 36.4 |
| Fan | Noctua NF-A6x25 5V PWM |
| Fan adapter | MODDIY PICO125-PWM4 |
| Heatsink | CNC aluminum, JLC CNC |
| Enclosure | MJF PA12, JLC3DP |
| TIM | Arctic MX-6 |


## References

- NVIDIA Jetson Orin NX and Orin Nano Series Thermal Design Guide, TDG-11127-001 v1.5
- Jetson Orin Nano Series Modules Datasheet, DS-11105-001 v1.7
- Jetson Orin NX and Orin Nano Series Design Guide, DG-10931-001 v1.5
