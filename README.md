# SweetSpot Glucometer

![Status](https://img.shields.io/badge/status-final%20prototype-brightgreen)
![Platform](https://img.shields.io/badge/platform-Arduino%20%2B%20Python-blue)
![Hardware](https://img.shields.io/badge/hardware-IO%20Rodeostat-green)
![Display](https://img.shields.io/badge/display-128×64%20OLED-purple)
![Enclosure](https://img.shields.io/badge/enclosure-3D%20printed-orange)
![Course](https://img.shields.io/badge/course-CBE%203300B-red)

**Team:** Madeleine Stubner, Lauren Van | **University of Pennsylvania, Department of Chemical & Biomolecular Engineering** | **Spring 2026**

---

SweetSpot is a low-cost electrochemical glucometer prototype that measures glucose concentration using commercial glucose test strips paired with an IO Rodeostat potentiostat. The system applies a chronoamperometric potential step at an empirically validated voltage (0.3 V, determined via cyclic voltammetry), records the resulting current response, extracts peak current as the calibration feature, and displays the computed glucose concentration in real time on a 128×64 OLED screen mounted inside a 3D-printed enclosure.

The device demonstrates an end-to-end engineering chain: electrochemical sensing → signal processing → calibration curve application → serial communication → embedded display output. All subsystems are integrated and functional. Quantitative accuracy is limited by single-strip calibration and commercial strip variability, which are the primary targets for future refinement.

---

## Table of Contents

- [Problem Definition & Motivation](#1-problem-definition--motivation)
  - [The Clinical Gap](#the-clinical-gap)
  - [Market Research & Competitive Analysis](#market-research--competitive-analysis)
  - [Stakeholder Analysis](#stakeholder-analysis)
  - [Project Goals](#project-goals)
- [Chemical Engineering Principles](#2-chemical-engineering-principles)
  - [Glucose Oxidase Reaction Chemistry](#glucose-oxidase-reaction-chemistry)
  - [Chronoamperometry and the Cottrell Equation](#chronoamperometry-and-the-cottrell-equation)
  - [Cyclic Voltammetry for Operating Point Selection](#cyclic-voltammetry-for-operating-point-selection)
  - [Connections to Core ChE Concepts](#connections-to-core-che-concepts)
- [Device Design](#3-device-design)
  - [System Architecture](#system-architecture)
  - [Component Selection & Rationale](#component-selection--rationale)
  - [Cost Analysis](#cost-analysis)
  - [Glucose Standards Preparation](#glucose-standards-preparation)
  - [Preliminary Performance Calculations](#preliminary-performance-calculations)
  - [Schematics & Wiring](#schematics--wiring)
  - [Code](#code)
  - [3D Printed Enclosure](#3D-printed-enclosure)
- [Calibration: Iteration History & Challenges](#4-calibration-iteration-history--challenges)
  - [Iteration 1 — OneTouch Strips, Peak Current](#iteration-1--onetouch-strips-peak-current)
  - [Iteration 2 — Steady-State Current](#iteration-2--steady-state-current)
  - [Iteration 3 — DI Water Normalization](#iteration-3--di-water-normalization)
  - [Root Cause Analysis & Hardware Intervention](#root-cause-analysis--hardware-intervention)
  - [Iteration 4 (Final) — True Metrix Strips](#iteration-4-final--true-metrix-strips)
- [Performance Data & Validation](#5-performance-data--validation)
  - [Final Calibration Results](#final-calibration-results)
  - [End-to-End Screen Validation](#end-to-end-screen-validation)
  - [Error Analysis](#error-analysis)
  - [Nonlinear Kinetics & Operating Limit](#nonlinear-kinetics--operating-limit)
  - [System Limitations](#system-limitations)
- [Progress Log](#6-progress-log)
- [SWOT Analysis](#7-swot-analysis)
- [Future Work](#8-future-work)
- [Repository Contents](#repository-contents)

---

## 1. Problem Definition & Motivation

### The Clinical Gap

Diabetes is one of the most rapidly growing chronic diseases globally and locally, and access to routine glucose monitoring remains a critical unmet need.

**Global landscape:**
- Diabetes cases have quadrupled since 1990 to **830 million**, disproportionately affecting low- and middle-income populations and youth
- The CDC projects that **40% of Americans** will develop diabetes in their lifetime based on current trends
- In 2018, **13% of U.S. adults** had diabetes and **88 million** had prediabetes

**Philadelphia specifically:**
- Diabetes prevalence has increased by **>50% over the past 15 years**
- **1 in 3** individuals living with diabetes in Philadelphia are unaware of their condition
- In 2017, diabetes was the **6th leading cause of death** in Philadelphia (374 deaths)

**The economic burden:**
- Diabetes is the most expensive chronic condition in the United States (NIH)
- Diagnosed patients incur medical expenses approximately **2.6× higher** than those without diabetes (CDC)

SweetSpot is motivated by the hypothesis that a lower-cost electrochemical sensing platform could support broader glucose awareness and improve screening access in communities where current glucometer costs are a genuine barrier.

---

### Market Research & Competitive Analysis

The blood glucose monitoring devices market is projected to grow from approximately **$12.4B (2023) to $30.2B (2033)**, driven by rising diabetes prevalence and demand for continuous monitoring. The largest growth is in consumables (test strips), not instruments — validating the use of commercial strips in our design.

**Glucometer Market Sizing (Bottom-Up)**

| Scope | Diabetics | Addressable Market |
|---|---|---|
| Global | 830 million | $24.9 B |
| United States | 38 million | $1.14 B |
| Philadelphia | ~150,000 | $4.5 M |
| Our target (0.001% share) | — | $450 |

*Average self-service glucometer cost: ~$30/device. Our device targets this tier.*

**Competitor Analysis**

| Device | Cost | Accuracy | Accessibility | Notes |
|---|---|---|---|---|
| Dexcom G6 | $$$$ | High | Low | Continuous monitoring; requires prescription |
| Walgreens A1C Test Kit | $$$ | High | Low | Lab-based; not real-time |
| Traditional Strip Glucometers | $$ | Medium | Medium | Requires proprietary strips + reader |
| **SweetSpot** | **$** | **Medium*** | **High** | Open-architecture; targets low-resource settings |

*\*Current prototype; accuracy is the primary future improvement target.*

**Competitive positioning:** Existing devices prioritize accuracy but sacrifice accessibility due to cost. SweetSpot intentionally targets the middle ground — acceptable accuracy at dramatically lower cost — to serve underserved populations not currently engaged in routine monitoring.

---

### Stakeholder Analysis

| Stakeholder | Benefit |
|---|---|
| **Patients** | Earlier detection → reduced complications; lower barrier to monitoring |
| **Healthcare Workers** | Fewer emergency interventions from unmanaged hyperglycemia |
| **Community Health Clinics & FQHCs** | Affordable tool for screening programs |
| **Medical Device Industry** | Competitive pressure toward lower-cost solutions |
| **Insurance Companies** | Lower long-term costs from prevention vs. acute intervention |

**Target distribution pathways:** community health clinics, federally qualified health centers (FQHCs), nonprofit chronic disease prevention organizations, local health departments, and university/hospital-affiliated screening initiatives.

---

### Project Goals

The system was designed to:

1. Run a chronoamperometric measurement on a commercial glucose test strip at an empirically optimized applied voltage
2. Extract a meaningful electrochemical feature from the current response (peak current)
3. Convert that feature to glucose concentration using an experimentally derived and validated calibration curve
4. Display the computed concentration in real time on an embedded OLED screen
5. House all components within a 3D-printed enclosure suitable for demonstration

Development proceeded in two stages:
- **Stage 1:** Computer-dependent prototype — validates sensing, calibration, and display integration via laptop
- **Stage 2:** Cased final prototype — integrates all working components into a physical enclosure

---

## 2. Chemical Engineering Principles

### Glucose Oxidase Reaction Chemistry

Commercial glucose test strips rely on **glucose oxidase (GOx)**, an enzyme that reacts selectively with glucose. In the presence of molecular oxygen, GOx catalyzes the oxidation of D-glucose to D-gluconolactone, producing hydrogen peroxide (H₂O₂) as a byproduct:

$$\text{D-glucose} + \text{O}_2 \xrightarrow{\text{GOx}} \text{D-gluconolactone} + \text{H}_2\text{O}_2$$

The H₂O₂ produced is electroactive. When a constant potential is applied by the potentiostat, H₂O₂ is oxidized at the working electrode, releasing electrons that generate a measurable current. Because each mole of glucose produces exactly one mole of H₂O₂ (1:1 stoichiometry), the measured current is directly proportional to glucose concentration. This stoichiometric coupling provides the **selectivity** of the sensor — only glucose contributes to the signal, not other blood components.

---

### Chronoamperometry and the Cottrell Equation

Chronoamperometry applies a fixed potential step and records current as a function of time. Under ideal diffusion-controlled conditions, the time-dependent current follows the **Cottrell equation**:

$$i(t) = \frac{nFAC\sqrt{D}}{\sqrt{\pi t}}$$

Where:
- $i(t)$ = current at time $t$ (A)
- $n$ = number of electrons transferred (2 for H₂O₂ oxidation: H₂O₂ → O₂ + 2H⁺ + 2e⁻)
- $F$ = Faraday's constant (96,485 C/mol)
- $A$ = electrode area (cm²)
- $C$ = analyte concentration (mol/cm³)
- $D$ = diffusion coefficient (cm²/s)

**Key implication:** At any fixed time point, $i \propto C$. This is the physical basis for using peak or steady-state current as a calibration feature — current linearly encodes concentration, provided the measurement window is within the diffusion-controlled regime.

The $t^{-1/2}$ decay behavior predicted by the Cottrell equation was verified experimentally:

<img width="418" height="352" alt="Proof of t^-1/2 relationship between current and time" src="https://github.com/user-attachments/assets/76fb609b-c87f-40a4-b7ed-2942f8c95f13" />

> **Figure 1.** Chronoamperometry: current vs. time at 0.3 V applied potential. The rapid decay from peak current follows the $t^{-1/2}$ profile predicted by the Cottrell equation, confirming diffusion-limited behavior in the early time window.

**Assumptions and their validity:**
- Semi-infinite planar diffusion — valid for short measurement times relative to strip geometry
- No convection — valid under quiescent solution conditions
- Instant potential step — approximated by the Rodeostat's rise time
- Breakdown at high concentrations — GOx enzyme saturation violates the linear Cottrell assumption (see Section 5)

---

### Cyclic Voltammetry for Operating Point Selection

**Cyclic voltammetry (CV)** sweeps the working electrode potential linearly between two set values while recording current — producing a full polarization curve for the electrochemical system. The anodic (positive) current peak marks H₂O₂ oxidation at the working electrode, and its potential indicates where oxidation rate is maximized.

CV was used as a **diagnostic and optimization tool** to replace the arbitrary 0.5 V previously assumed for chronoamperometric measurements. CV scans were run across all five glucose concentrations (2.2, 4.4, 6.6, 8.8, and 11.1 mM), sweeping from −0.6 V to +0.6 V:

<img width="598" height="444" alt="Cyclic voltammetry for 4.4 mM solution showing oxidation peak at 0.3V" src="https://github.com/user-attachments/assets/83df46b3-026a-41f9-9a17-4e6163ba23e7" />

> **Figure 2.** Cyclic voltammetry for 4.4 mM glucose solution. The anodic oxidation peak appears consistently near 0.3 V across all CV cycles. Peak position was independent of concentration, confirming it reflects strip electrode chemistry rather than analyte quantity. 0.3 V was adopted as the fixed applied potential for all subsequent chronoamperometric measurements.

**Electrochemical regimes identified:**

| Regime | Potential Range | Governing Physics | Analog in ChE |
|---|---|---|---|
| Activation-controlled | Below ~0.2 V | Insufficient driving force; reaction rate limited by activation energy barrier | Kinetically controlled reactor operating far from equilibrium |
| **Optimal operating point** | **~0.3 V** | **Maximum signal sensitivity; transition between kinetic and transport control** | **Peak of a rate-vs-driving-force curve** |
| Mass-transport-limited | Above ~0.35 V | H₂O₂ oxidized as fast as it arrives; rate controlled by diffusion; competing reactions begin | Transport-limited reactor where feed delivery is rate-limiting |

Selecting 0.3 V is equivalent to choosing the optimal operating point on a reaction rate curve — maximizing conversion per unit driving force while avoiding side reactions and transport limitations on either side.

---

### Connections to Core ChE Concepts

**Mass and species balances:** The enzymatic reaction converts glucose stoichiometrically to H₂O₂. The current measured is proportional to the molar flux of H₂O₂ to the electrode surface — a direct application of Faraday's law ($Q = nFN$, where $N$ is moles reacted) combined with a steady-state species balance on the diffusion layer.

**Transport phenomena:** The Cottrell equation arises from solving Fick's second law of diffusion for a semi-infinite planar electrode with a potential-step boundary condition. Current decay over time reflects depletion of the diffusion layer — a mass transport effect that governs signal shape and informs both feature extraction strategy and the measurement time window.

**Reaction kinetics and operating windows:** CV characterization directly mapped the activation-controlled vs. mass-transport-limited regimes for this strip system, identical to the two asymptotic limits in heterogeneous catalysis (Damköhler number analysis). Choosing 0.3 V places the operating point at the transition where sensitivity is maximized.

**Nonlinear kinetics and saturation:** Breakdown of linearity at high glucose concentrations reflects **enzyme saturation** — a Michaelis-Menten kinetic effect where the GOx reaction rate approaches $V_{\max}$ and no longer scales proportionally with substrate:

$$v = \frac{V_{\max}[S]}{K_M + [S]}$$

At low [S] (below ~11 mM in our system), $[S] \ll K_M$ and $v \approx \frac{V_{\max}}{K_M}[S]$ — linear, first-order behavior. At high [S], the rate saturates to $V_{\max}$ — zero-order behavior. This transition is identical to a first-order-to-zero-order reactor kinetic shift at high substrate concentration, a concept central to reactor design. The device's validated operating range (2.2–11.1 mM) is bounded by this saturation on the high end.

**Calibration as process modeling:** The experimentally derived calibration curve functions as a process model: it maps a measured variable (peak current) to a process output (glucose concentration). Constructing, validating, and refining this model — including identifying its failure modes at high concentrations — is directly analogous to process identification and control loop tuning in process engineering.

---

## 3. Device Design

### System Architecture

The SweetSpot system operates through a five-stage pipeline:

```
[Glucose Sample + Test Strip] 
        ↓ electrochemical signal
[IO Rodeostat Potentiostat] — applies 0.3 V chronoamperometric step, records i(t)
        ↓ current-time data (USB)
[Laptop / Jupyter Notebook] — extracts peak current, applies calibration equation
        ↓ concentration value (USB serial)
[Arduino Uno] — receives concentration string, formats display output
        ↓ SPI data
[128×64 OLED Display] — renders concentration in real time
```

<img width="1137" height="653" alt="System Architecture Diagram" src="https://github.com/user-attachments/assets/cff04c35-f222-4edc-b9f9-14935a0425f3" />

> **Figure 3.** Full system architecture showing signal flow from electrochemical measurement through Python processing, Arduino microcontroller, and OLED display output.

**Device operation summary:**
1. User applies a glucose test strip to the shim-assisted strip holder and initiates measurement from the Jupyter Notebook
2. The Rodeostat applies a 0.3 V chronoamperometric step for 29 seconds (1 s equilibration at 0 V, then 29 s at 0.3 V)
3. Python extracts peak current by absolute magnitude across the full time series
4. The calibration equation (C = (I − CAL_INTERCEPT) / CAL_SLOPE) converts peak current to glucose concentration in mM
5. If the result is outside 0–50 mM (physiologically implausible), the system automatically re-runs up to 5 times
6. On a valid result, the concentration is transmitted as a plain string over USB serial to the Arduino
7. The Arduino sketch — running independently on the board — receives the string and renders it on the OLED in real time; no Arduino IDE is required during operation

---

### Component Selection & Rationale

All components were selected to maximize cost-effectiveness while maintaining functional adequacy for electrochemical sensing and embedded display.

| Component | Role | Design Rationale |
|---|---|---|
| **IO Rodeostat** | Potentiostat — applies controlled voltage, measures current | Open-source, low-cost potentiostat ($240); enables chronoamperometry and CV; USB interface compatible with Python |
| **True Metrix Test Strips** | Electrochemical transducer — GOx enzyme layer converts glucose to H₂O₂ signal | Selected after comparative testing vs. OneTouch strips; more compatible electrode geometry with Rodeostat leads; procured fresh from CVS to minimize batch variability |
| **Arduino Uno** | Microcontroller — receives concentration over serial, drives OLED | Low-cost ($27.60); supports SPI for OLED; simple serial parsing; widely documented |
| **128×64 OLED Display** | User output — renders glucose concentration | SPI interface (faster than I²C); low power; compact ($19.50); readable in ambient lighting |
| **3D-Printed Enclosure** | Physical housing — contains Arduino + OLED | Enables demonstration-ready form factor; custom fit to component dimensions |
| **Shim (inverted strip)** | Mechanical — standardizes strip contact pressure | Addresses inconsistent electrode-to-strip contact resistance identified as primary reproducibility failure mode |

**Design trade-offs made:**
- Smaller OLED (no touchscreen) over larger display → ~$30 savings with no functional loss for this use case
- CVS True Metrix strips (lowest cost per strip at CVS) over premium brands → required voltage re-characterization via CV but maintained compatibility
- Laptop-dependent processing over onboard MCU → enables rapid iteration; migration to onboard processing is identified Future Work

---

### Cost Analysis

| Component | Unit Cost |
|---|---|
| Arduino Uno | $27.60 |
| IO Rodeostat Potentiostat | $240.00 |
| 128×64 OLED Graphic Display | $19.50 |
| Analyzer Cable/Probe | $9.54 |
| CVS True Metrix Glucose Strips (30 ct.) | $9.99 |
| **TOTAL** | **$306.63** |

The Rodeostat dominates cost and is the primary target for cost reduction in a production design. Integration of a custom analog front-end onto a single PCB could reduce instrumentation cost by an order of magnitude.

---

### Glucose Standards Preparation

To mimic the electrolyte balance of human blood, a **140 mmol/L NaCl stock solution** was prepared (standard physiological NaCl concentration):

$$\left(140 \frac{\text{mmol}}{\text{L}}\right)\left(\frac{58.44 \text{ g}}{\text{mol}}\right)\left(\frac{0.100 \text{ L}}{1}\right) = 0.818 \text{ g NaCl in 100 mL}$$

Glucose standards were prepared by dissolving known masses of glucose (MW = 180.156 g/mol) in 25 mL of stock solution:

$$C \text{ (mM)} = \frac{\text{mass (g)}}{180.156 \text{ g/mol} \times 0.025 \text{ L}} \times 1000$$

**Standard concentrations and clinical context:**

| Glucose Level | Mass Added (g) | Concentration (mM) | Clinical Reference |
|---|---|---|---|
| Baseline | 0.000 | 0.00 | — |
| Low | 0.010 | 2.22 | Below normal fasting |
| Normal | 0.020 | 4.44 | Normal fasting (~80 mg/dL) |
| High | 0.050 | 11.10 | Elevated (diabetic range, ~200 mg/dL) |
| Very High | 0.070 | 15.54 | Severely elevated |

The five standards span the clinically relevant glucose range (normal fasting through moderate diabetic), enabling characterization of the linear sensing regime and definition of the validated operating range.

<img width="492" height="244" alt="Glucose Standards Table" src="https://github.com/user-attachments/assets/97abdf26-e984-4bfc-b0cf-d5142b466d45" />

---

### Preliminary Performance Calculations

Under ideal Cottrell behavior, the expected peak current at 5 mM glucose (normal fasting level) can be estimated as follows:

**Given:**
- $D \approx 1.4 \times 10^{-5}$ cm²/s (H₂O₂ diffusion coefficient in aqueous solution)
- $A \approx 0.1$ cm² (estimated strip electrode area)
- $n = 2$ (electrons transferred per H₂O₂ molecule oxidized)
- $C = 5 \times 10^{-6}$ mol/cm³ (5 mM)
- $t = 5$ s (evaluation time)

$$i = \frac{nFAC\sqrt{D}}{\sqrt{\pi t}} = \frac{2 \times 96485 \times 0.1 \times (5 \times 10^{-6}) \times \sqrt{1.4 \times 10^{-5}}}{\sqrt{\pi \times 5}} \approx 0.5 \ \mu\text{A}$$

Experimentally observed peak currents with the final protocol are in the **low-to-mid µA range**, consistent with this estimate when accounting for the strip's actual electrode geometry and the higher applied potential (0.3 V drives a more complete reaction than the ideal diffusion-only Cottrell model assumes).

---

### Schematics & Wiring

#### OLED Wiring (SPI Interface)

The OLED display requires SPI communication (not I²C — an early debugging finding). Wiring is as follows:

| OLED Pin | Arduino Pin | Function |
|---|---|---|
| DC (Data/Command) | D8 | Selects data vs. command byte |
| RES (Reset) | D9 | Hardware reset line |
| CS (Chip Select) | D10 | SPI chip select |
| MOSI (Data) | D11 | SPI data line |
| SCK (Clock) | D13 | SPI clock |

#### Electrode Lead Assignment

| Lead Color | Electrode Role |
|---|---|
| Red | Working |
| Orange | Reference |
| Yellow / Grey | Counter |

Consistent lead polarity and placement were critical to eliminating stray resistance at the strip-to-Rodeostat interface, identified during root cause analysis as a major noise source.

<img width="541" height="603" alt="Soldered Arduino and OLED assembly" src="https://github.com/user-attachments/assets/a5612769-33c9-49d7-9f39-8220c60cb2b9" />

> **Figure 4.** Soldered OLED and Arduino assembly on breadboard. All connections were soldered from loose jumper wires to eliminate intermittent contact — a hardware intervention that stabilized display communication.

---

### Code

#### Arduino Sketch — OLED Display

```cpp
#include <SPI.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_DC  8
#define OLED_CS  10
#define OLED_RST 9

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &SPI, OLED_DC, OLED_RST, OLED_CS);

String incoming = "";

void showValue(String value) {
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);

  display.setTextSize(1);
  display.setCursor(18, 6);
  display.print("GLUCOSE");

  display.drawLine(0, 18, 127, 18, SSD1306_WHITE);

  display.setTextSize(3);
  display.setCursor(10, 30);
  display.print(value);

  display.setTextSize(1);
  display.setCursor(92, 52);
  display.print("mg/dL");

  display.display();
}

void setup() {
  Serial.begin(115200);
  delay(2000);
  if (!display.begin(SSD1306_SWITCHCAPVCC)) { while (1) {} }
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(1);
  display.setCursor(10, 25);
  display.print("Waiting for value...");
  display.display();
  Serial.println("OLED_READY");
}

void loop() {
  while (Serial.available() > 0) {
    char c = Serial.read();
    if (c == '\n') {
      incoming.trim();
      if (incoming.length() > 0) {
        showValue(incoming);
        Serial.print("DISPLAYED: ");
        Serial.println(incoming);
      }
      incoming = "";
    } else {
      incoming += c;
    }
  }
}
```

#### Python Measurement & Processing Script (Jupyter Notebook)

All measurement, processing, and display communication runs from this single Jupyter Notebook script. The script connects to both USB devices (Rodeostat and Arduino) simultaneously, runs the electrochemical measurement, computes the glucose concentration, and pushes the result to the OLED — without ever opening the Arduino IDE.

```python
from potentiostat import Potentiostat   # IO Rodeostat Python library
import matplotlib.pyplot as plt
import pandas as pd
import serial                           # pyserial — USB communication with Arduino
import serial.tools.list_ports
import time

# ── SECTION 1: PORT DISCOVERY ────────────────────────────────────────────────
# Two USB devices are connected simultaneously: the Rodeostat (potentiostat)
# and the Arduino Uno (OLED controller). They appear as separate serial ports.
# Print all available ports so the correct port names can be identified.
# On Mac, ports appear as /dev/cu.usbmodem****; on Windows as COM*.
print("Available ports:")
for p in serial.tools.list_ports.comports():
    print(" ", p.device, "-", p.description)

rodeostat_port = "/dev/cu.usbmodem101"    # IO Rodeostat potentiostat
oled_port      = "/dev/cu.usbmodem11101"  # Arduino Uno (running OLED sketch)

# ── SECTION 2: CALIBRATION CONSTANTS ────────────────────────────────────────
# Derived from final calibration dataset (April 26, True Metrix strips, 0.3 V):
#   I_peak (µA) = CAL_SLOPE × C (mM) + CAL_INTERCEPT    [forward model]
#   C (mM) = (I_peak - CAL_INTERCEPT) / CAL_SLOPE        [inverted for prediction]
# R² = 0.892 on the final calibration dataset
CAL_SLOPE     = 4.065
CAL_INTERCEPT = -3.897

# ── SECTION 3: MEASUREMENT SETTINGS ─────────────────────────────────────────
datafile   = "data.txt"        # Raw output file from Rodeostat
out_01s    = "chrono_data.xlsx"  # Processed DataFrame saved to Excel
max_retries = 5                # Maximum re-runs if concentration is out of range

# ── SECTION 4: CONNECT TO ARDUINO (OLED) ────────────────────────────────────
# Opens a serial connection to the Arduino at 115200 baud — the same baud rate
# set in the Arduino sketch. The Arduino sketch runs continuously on the board
# (uploaded once via Arduino IDE), listening on its serial port for a number
# string sent by this Python script. Python talks to it directly over USB;
# the Arduino IDE does not need to be open.
oled = serial.Serial(oled_port, 115200, timeout=1)
time.sleep(2.5)                # Wait for Arduino to finish booting/resetting
oled.reset_input_buffer()      # Clear any stale bytes in the receive buffer
oled.reset_output_buffer()     # Clear any stale bytes in the transmit buffer
print("OLED connected on", oled_port)

# ── SECTION 5: CONNECT TO RODEOSTAT ─────────────────────────────────────────
# The Potentiostat class wraps the Rodeostat's USB serial protocol.
# set_curr_range sets the analog measurement range — "1000uA" means the
# Rodeostat will measure currents up to 1000 µA with appropriate resolution.
# set_sample_period(100) sets a 100 ms interval between current measurements,
# producing ~290 data points over the 29-second chronoamperometry window.
dev = Potentiostat(rodeostat_port)
try:
    dev.set_curr_range("1000uA")   # Set current range before test
except:
    pass                           # Some firmware versions handle this automatically
dev.set_curr_range("1000uA")
dev.set_sample_period(100)         # 100 ms → ~10 samples/second

# ── SECTION 6: DEFINE CHRONOAMPEROMETRY PARAMETERS ──────────────────────────
# Chronoamperometry applies a potential step and records current vs. time.
# quietValue / quietTime: hold at 0 V for 1 s before the step to allow the
#   electrochemical system to equilibrate (no current flows at open circuit).
# step: at t=0 ms, hold 0 V; at t=0 ms step to 0.3 V and hold for 29 s.
# 0.3 V was determined by cyclic voltammetry as the H₂O₂ oxidation peak —
# the potential at which GOx-produced H₂O₂ is oxidized most efficiently.
name = "chronoamp"
test_param = {
    "quietValue": 0.0,
    "quietTime":  1000,                      # 1 s equilibration at 0 V
    "step": [(0, 0.0), (29000, 0.3)],        # Step to 0.3 V, hold 29 s
}
dev.set_param(name, test_param)

# ── SECTION 7: MEASUREMENT LOOP WITH AUTOMATIC RE-RUN ───────────────────────
# Physiologically valid blood glucose is 0–50 mM. If the computed concentration
# falls outside this range, the measurement is flagged as an artifact (e.g., from
# poor strip contact, electrical noise, or a missed step) and the test re-runs.
# The loop retries up to max_retries (5) times before giving up and sending
# an error code to the OLED.
concentration  = None
df_raw         = None
curr           = None
peak_current_uA = None
peak_time      = None

for attempt in range(1, max_retries + 1):
    print(f"\nRunning test... Attempt {attempt}/{max_retries}")

    # Run the chronoamperometry test via the Rodeostat.
    # Returns three lists: time (s), voltage (V), and current (µA).
    t, volt, curr = dev.run_test(name, display="pbar", filename=datafile)

    # Build a pandas DataFrame for structured processing and export.
    df_raw = pd.DataFrame({
        "time_s":     t,
        "voltage_V":  volt,
        "current_uA": curr
    })
    df_raw = df_raw.sort_values("time_s").reset_index(drop=True)
    df_raw["time_s"] = df_raw["time_s"].round(2)

    # Save raw data to Excel for archiving and later analysis.
    df_raw.to_excel(out_01s, index=False)
    print("Saved:", out_01s)

    # Extract peak current by absolute magnitude.
    # Using abs() captures the peak regardless of polarity — important because
    # the Cottrell current can appear with a positive or negative sign depending
    # on Rodeostat lead orientation and strip electrode convention.
    peak_idx        = df_raw["current_uA"].abs().idxmax()
    peak_time       = float(df_raw.loc[peak_idx, "time_s"])
    peak_current_uA = float(df_raw.loc[peak_idx, "current_uA"])
    print(f"Peak current at {peak_time:.2f} s: {peak_current_uA:.3f} µA")

    # Apply calibration equation (inverted linear model):
    #   C (mM) = (I_peak (µA) - CAL_INTERCEPT) / CAL_SLOPE
    concentration = (peak_current_uA - CAL_INTERCEPT) / CAL_SLOPE
    print(f"Calculated concentration: {concentration:.2f} mM")

    # Quality gate: accept only physiologically plausible results (0–50 mM).
    # Values outside this range indicate a failed measurement — caused by
    # poor strip contact, a transient noise spike dominating peak extraction,
    # or electrical interference. Trigger a re-run automatically.
    if 0 <= concentration <= 50:
        print("Concentration in valid range. Accepting result.")
        break
    else:
        print("Concentration out of range (<0 or >50 mM). Re-running...")
        concentration = None
        time.sleep(1)   # Brief pause before next attempt

# ── SECTION 8: OUTPUT ────────────────────────────────────────────────────────
if concentration is None:
    # All retries exhausted — send error code to OLED
    print("\nNo valid concentration obtained after max retries.")
    oled.write(b"ERR\n")
else:
    # Plot the final valid run: voltage and current vs. time,
    # with a vertical dashed line marking the peak current time point.
    plt.figure(figsize=(9, 6))
    plt.subplot(211)
    plt.title("Chronoamperometry: Voltage and Current vs. Time")
    plt.plot(df_raw["time_s"], df_raw["voltage_V"])
    plt.ylabel("Potential (V)"); plt.grid(True)
    plt.subplot(212)
    plt.plot(df_raw["time_s"], df_raw["current_uA"])
    plt.axvline(x=peak_time, linestyle="--", label=f"Peak @ {peak_time:.2f} s")
    plt.ylabel("Current (µA)"); plt.xlabel("Time (s)")
    plt.grid(True); plt.legend()
    plt.tight_layout(); plt.show()

    print(f"Final peak current: {peak_current_uA:.3f} µA")
    print(f"Final concentration: {concentration:.2f} mM")

    # Send concentration to Arduino over USB serial as a plain string + newline.
    # The Arduino sketch listens for '\n' as the message terminator, then
    # parses the string and renders it on the OLED screen.
    # This communication happens entirely through the USB cable — no wireless,
    # no Arduino IDE required. Python writes; Arduino reads and displays.
    oled.write(f"{concentration:.1f}\n".encode())
    time.sleep(0.3)
    while oled.in_waiting:
        print("OLED:", oled.readline().decode(errors="ignore").strip())

oled.close()
```

---

### 3D Printed Enclosure
Reworking files provided in 3300A, the introductory course for 3300B, the files were rescaled by 1.60 times to ensure proper fit for our device. While GitHub does not support the necessary file type for including the printed files, a screenshot of the back case is shown below.<img width="759" height="540" alt="Screenshot 2026-05-06 at 12 30 32 PM" src="https://github.com/user-attachments/assets/d3386f4a-341f-4abb-9649-313c5f849cb8" />






## 4. Calibration: Iteration History & Challenges

Calibration required four distinct iterations over three months. Each failure was diagnosed and addressed through targeted hardware or methodological intervention — a systematic engineering process that ultimately produced the functional final prototype.

---

### Iteration 1 — OneTouch Strips, Peak Current (March 5)

**Method:** Peak (maximum) current from chronoamperometry at 0.5 V; four concentrations, one measurement each.

**Result:**
$$\text{Glucose (mM)} = 1.1978 \times I_{\text{peak}} (\mu\text{A}) - 0.4567 \quad (R^2 = 0.9998)$$

<img width="985" height="459" alt="Initial calibration curve March 5 with misleading R² = 0.9998" src="https://github.com/user-attachments/assets/2cb0e151-1bb0-470f-8fdb-c8190bdcf42f" />

> **Figure 5.** Initial calibration curve (March 5). R² = 0.9998 on training data — a misleading result from fitting exactly 4 points with one measurement per concentration (effectively interpolation, not regression). Subsequent validation produced predictions ranging from −10 to >50 mM.

**Failure analysis:** The extremely high R² was an artifact of interpolation rather than genuine model fit. With only four calibration points and no within-concentration replication, the model captured zero measurement variability. Predictions on new strips completely failed, confirming that the model did not generalize.

---

### Iteration 2 — Steady-State Current (March 25)

**Method:** Shifted from peak current to steady-state current at fixed time points (10 s and 20 s), where the Cottrell transient has decayed and the signal is more stable.

**Result:**

<img width="622" height="520" alt="Calibration curve comparison for steady-state at 10s and 20s" src="https://github.com/user-attachments/assets/23fa1c65-093f-43e6-bedc-88f5650b4043" />

> **Figure 6.** Calibration curves using steady-state current at 10 s (left) and 20 s (right) for OneTouch strips. R² values near zero; no usable concentration-dependent trend. Measurements scattered without correlation across replicate strips.

**Failure analysis:** Reproducibility failed across both feature types (peak and steady-state), pointing to a **systemic physical cause** rather than an algorithmic one. Strip-to-strip variability and inconsistent mechanical contact were identified as root causes requiring hardware intervention — no signal processing approach could compensate for uncontrolled physical variables.

---

### Iteration 3 — DI Water Normalization (March 25)

**Method:** Baseline normalization — each strip's response in DI water was subtracted from the sample response to correct for inter-strip offset.

**Result:**

> Normalization corrected for inter-strip baseline offset but could not correct for **differences in sensitivity (slope)** between strips. Calibration remained inconsistent.

**Engineering interpretation:** Normalization is equivalent to subtracting a systematic additive bias. If the true variability is multiplicative (i.e., slope varies between strips, not just intercept), baseline subtraction is insufficient. A ratiometric approach or strip-by-strip calibration would be required — both demanding more measurements than the current protocol allows.

---

### Root Cause Analysis & Hardware Intervention (April 16)

Persistent failure across all signal features — peak current, steady-state current, normalized current — indicated that no calibration methodology could succeed until the underlying physical sources of variability were controlled. Two primary causes were identified and addressed:

**Cause 1: Strip batch variability**
- *Root cause:* Original OneTouch strips produced erratic baselines and poorly reproducible peak magnitudes across the batch. Strip age and storage conditions were uncontrolled.
- *Resolution:* Fresh True Metrix Self-Monitoring Glucose Test Strips were procured from CVS. True Metrix strips showed noticeably more consistent current-time profiles with cleaner Cottrell-like decay behavior. Electrode geometry also provided better mechanical compatibility with Rodeostat leads.

**Cause 2: Inconsistent mechanical contact**
- *Root cause:* Without controlled contact pressure, the resistance at the strip electrode-to-Rodeostat-lead interface varied from measurement to measurement. This introduced signal variability independent of glucose concentration — impossible to distinguish from real chemistry.

<img width="484" height="374" alt="Failed chronoamperometry from poor strip contact showing flat/distorted traces" src="https://github.com/user-attachments/assets/ec0b6f7d-2018-4d26-8643-ef776563f1a3" />

> **Figure 7.** Failed chronoamperometry measurements from poor strip contact (2.2 mM, no shim). Flat or severely distorted traces indicate intermittent electrode connection — indistinguishable from a zero-concentration sample in the absence of a quality check.

- *Resolution:* A **shim** (a second strip, inverted) was inserted beneath each test strip during measurement to apply consistent downward pressure, standardizing contact resistance.

**Additional hardware improvements:**
- Electrode connection rework with defined lead polarity (Red = Working, Orange = Reference, Yellow/Grey = Counter)
- CV-guided voltage selection (0.3 V, replacing arbitrary 0.5 V)

**Summary of root cause analysis and resolutions:**

| Issue Encountered | Root Cause Identified | Resolution Implemented |
|---|---|---|
| Non-reproducible calibration | Strip batch variability | Fresh CVS True Metrix strips |
| Signal dropout / flat traces | Inconsistent mechanical contact pressure | Shim added to strip holder |
| Non-optimal applied voltage | Arbitrary voltage selection (0.5 V) | CV-guided 0.3 V selection |
| Noisy electrode connection | Incorrect polarity and loose leads | Connection rework with defined lead assignment |
| Calibration nonlinearity at high [C] | GOx enzyme saturation (Michaelis-Menten) | Validated operating range limited to ≤ 11.1 mM |

---

### Iteration 4 (Final) — True Metrix Strips (April 26)

**Method:** Peak current at 0.3 V applied potential (CV-determined); new True Metrix strips with shim; one strip per concentration.

**Improvements over prior iterations:**
- New strips with more compatible electrode geometry
- One measurement per strip to prevent enzyme layer degradation from repeated exposure
- Shim to increase and standardize electrical contact
- Applied voltage at experimentally determined 0.3 V

<img width="525" height="398" alt="Final calibration curve April 26 with True Metrix strips" src="https://github.com/user-attachments/assets/72d408e0-0354-4cb2-950e-61019ecb7f4f" />

> **Figure 8.** Final calibration curve (April 26). True Metrix strips at 0.3 V applied potential. R² = 0.935 across the 2.2–11.1 mM range. Error bars reflect within-measurement noise. Deviation at 8.8 and 11.1 mM reflects onset of GOx enzyme saturation.

---

## 5. Performance Data & Validation

### Final Calibration Results

**Final calibration dataset:**

| Glucose Concentration (mM) | Peak Current (µA) |
|---|---|
| 2.2 | 4.42 |
| 4.4 | 11.21 |
| 6.6 | 13.80 |
| 8.8 | 31.40 |
| 11.1 | 35.82 |

**Calibration equation (forward — current from concentration):**

$$I_{\text{peak}} \ (\mu\text{A}) = 3.7379 \times C_{\text{glucose}} \ (\text{mM}) - 5.415 \quad (R^2 = 0.935)$$

**Inverted for real-time concentration prediction:**

$$\boxed{C_{\text{glucose}} \ (\text{mM}) = \frac{I_{\text{peak}} \ (\mu\text{A}) + 5.415}{3.7379}}$$

**Calibration summary:**

| Parameter | Value |
|---|---|
| Applied potential | 0.3 V (CV-determined) |
| Calibration feature | Peak current (µA), ignoring first 1 s |
| Calibration equation | C (mM) = (I + 5.415) / 3.7379 |
| R² | 0.935 |
| Validated range | 2.2–11.1 mM (40–200 mg/dL) |
| Strip type | True Metrix self-monitoring glucose test strips |

The R² of 0.935, though lower than the original four-point training fit (0.9998), is substantially more honest — it reflects a properly constructed calibration with real measurement variability present, rather than a four-point interpolation. The deviation at high concentrations is physically consistent with the Michaelis-Menten saturation behavior identified analytically.

---

### End-to-End Screen Validation

Known glucose standards were measured and predictions displayed on the OLED — testing the full pipeline (electrochemical acquisition → processing → serial transmission → display):

| Actual Concentration (mM) | Screen Output Trial 1 (mM) | Screen Output Trial 2 (mM) |
|---|---|---|
| 2.2 | 11.7 | 2.7 |
| 4.4 | 3.42 | 14.5 |
| 6.6 | 7.9 | 33.2 |
| 8.8 | 42.5 | 47.9 |
| 11.1 | 48.8 | 35.0 |

These results confirm the **full sensing-to-display pipeline is functional end-to-end**. Quantitative accuracy varies significantly between trials at the same concentration — driven by strip-to-strip variability that a single-strip calibration cannot capture. The pipeline itself (acquire → process → transmit → display) operates correctly for every measurement.

**See the device in action:** https://github.com/user-attachments/assets/cb9e7f55-9fd3-4ae5-9c2e-26083d2b7484

**Full function video!** https://drive.google.com/file/d/17529kVom23vIuxYyKV71NutiktK6czey/view?usp=sharing

---

### Error Analysis

Validation was performed by testing known glucose standards against the device's screen output. This assessment evaluates real-world prototype performance — not model construction.

| Error Metric | Value |
|---|---|
| Mean Absolute Error (MAE) | 20.27 mM |
| Root Mean Squared Error (RMSE) | 23.57 mM |
| Mean Absolute Percent Error (MAPE) | 328.7% |
| Average bias | +19.68 mM (systematic overprediction) |

<img width="521" height="410" alt="Error analysis figure 1" src="https://github.com/user-attachments/assets/aaf52fd2-370b-471a-8e74-866dc931845e" />

<img width="550" height="398" alt="Error analysis figure 2" src="https://github.com/user-attachments/assets/5b434416-e2b9-4cc9-a8e9-aaae7d7dd0ee" />

<img width="533" height="398" alt="Error analysis figure 3 percent error by concentration" src="https://github.com/user-attachments/assets/5a39f936-9187-481e-b23e-9deb90e26165" />

> **Figures 9–11.** Error analysis: predicted vs. actual glucose concentration (parity plot), absolute error by concentration, and percent error by concentration. Error increases sharply at higher concentrations (>6.6 mM), consistent with the onset of enzyme saturation and amplified peak current artifacts.

**Error contributors (ranked by likely impact):**
1. **Single-strip calibration** — one measurement per concentration provides no inter-strip variability estimate; the calibration slope and intercept are highly sensitive to individual strip behavior
2. **Inconsistent strip seating** — residual mechanical contact variability despite shim intervention
3. **Contact resistance** — variable resistance at the strip electrode-to-Rodeostat-lead interface introduces noise directly into the current signal
4. **Strip manufacturing variation** — batch-to-batch and strip-to-strip variation in GOx enzyme layer deposition and electrode geometry
5. **Nonlinear enzymatic response** — GOx saturation above ~8 mM produces accelerating nonlinearity not captured by the linear calibration model
6. **Electrical noise** — Rodeostat signal noise at the low current magnitudes measured (<50 µA)

**Mitigation implemented:** If a reading is <0 or >50 mM, the system automatically re-runs the measurement up to 5 times — a real-time quality gate that catches obvious outliers.

**Engineering interpretation:** The quantitative error, while large, is a known and bounded limitation of a single-strip, single-measurement calibration. The path to improvement is clear: triplicate calibration measurements per concentration would provide uncertainty bounds, enable outlier rejection, and yield a more robust regression. This is the highest-priority future work item.

---

### Nonlinear Kinetics & Operating Limit

<img width="401" height="315" alt="Actual glucose concentration vs screen output showing nonlinear overprediction" src="https://github.com/user-attachments/assets/c4e9aaf9-d595-4444-863f-4deb1d4c7efb" />

> **Figure 12.** Actual vs. screen output glucose concentration. Ideal performance would follow the dashed diagonal. Screen output diverges dramatically above ~6.6 mM, consistent with the onset of GOx enzyme saturation (Michaelis-Menten regime transition) combined with peak-current artifacts at higher concentration.

The theoretical expectation at saturation is a **plateau** in current (Michaelis-Menten zero-order regime). The observed behavior of **increasing overprediction** suggests that peak-current artifacts (e.g., transient noise spikes that are disproportionately large at higher concentrations) are amplified by the single-strip calibration rather than saturating. This is a compound failure: enzyme saturation breaks the linear Cottrell assumption, and the peak-current extraction method becomes less reliable as the true signal shape deviates from the ideal Cottrell decay.

---

### System Limitations

| Category | Limitation |
|---|---|
| **Measurement & Signal** | Peak current sensitive to transient noise spikes; signal affected by electrode contact variability |
| **Chemical** | GOx enzyme saturation limits linear range to ≤ ~11 mM; strip chemistry introduces non-ideal electrochemical behavior at higher concentrations |
| **Hardware** | Precision limited by Rodeostat's low-cost analog front end; manual strip insertion introduces inconsistent alignment and contact |
| **Calibration** | Linear model valid only in 2.2–11.1 mM range; single-strip calibration provides no uncertainty quantification |
| **Operation** | Computer-dependent (Jupyter Notebook required for measurement and processing) |

---

## 6. Progress Log

A chronological record of key development milestones, engineering decisions, and findings.

### February 12, 2026 — Initial Hardware Setup and Signal Verification

Connected the IO Rodeostat to a laptop and conducted an initial chronoamperometry test using a commercial glucose test strip and legacy connector leads. Emphasis was on confirming that the hardware and software pipeline could generate and capture an electrochemical signal.

<img width="788" height="483" alt="First chronoamperometry run February 12 showing noisy signal" src="https://github.com/user-attachments/assets/e5ac448c-d1ea-4db9-8321-44241245651d" />

> **Figure 13.** First chronoamperometry run (February 12). Signal present but highly noisy with no discernible concentration-dependent feature. Confirmed hardware connectivity and established the measurement pipeline as a foundation for all subsequent work.

**Engineering significance:** Feasibility verification — before a sensor can be evaluated quantitatively, the acquisition chain must be shown to produce a usable signal. This confirmed the end-to-end pathway: strip → Rodeostat → USB → Python → plot.

---

### February 26, 2026 — Calibration Data Collection and Pipeline Improvement

Prepared a full set of standard glucose solutions for calibration and improved the data pipeline. The updated Python workflow exported raw data to CSV/Excel files, making measurements reproducible and analysis-ready. First full datasets for high and very high glucose concentrations were collected.

---

### March 4, 2026 — Full Concentration Dataset Complete

All initial concentration measurements across the five glucose standards were completed. First time the system had enough data to attempt a calibration model.

---

### March 5, 2026 — Initial Calibration Curve

First calibration model constructed using peak current. R² = 0.9998 on training data (see Calibration Section for failure analysis). The high apparent linearity was a four-point interpolation artifact. Breakdown at high concentrations also observed and attributed to enzyme saturation.

---

### March 25, 2026 — Validation Testing and Failure Analysis

Extensive validation testing across peak current, steady-state current at 10 s and 20 s, and DI water normalization. All methods failed validation. Root cause analysis concluded the failures were physical — strip variability and mechanical contact — not algorithmic.

---

### April 2, 2026 — Arduino Serial Communication and OLED Display

After identifying that the display required SPI (not I²C) communication, the OLED was successfully initialized and validated on a breadboard. Python-to-Arduino serial communication pipeline was designed and tested end-to-end.

<img width="867" height="507" alt="Initial OLED wiring setup on breadboard April 2" src="https://github.com/user-attachments/assets/12587aa6-de29-4984-a667-b7863b4036df" />

> **Figure 14.** Initial OLED wiring on breadboard (April 2). SPI connection established after diagnosing and correcting an I²C initialization failure. Full serial communication from Python to Arduino and display verified.

---

### April 8, 2026 — Hardware Integration and Soldering

Moved from loose jumper wires to a soldered assembly. OLED and all Arduino connections were soldered to a breadboard, producing mechanically stable joints. Full pipeline (measure → process → display) verified end-to-end from a single Jupyter Notebook.

---

### April 16, 2026 — New Strips, Shim, CV, and Electrode Rework

Systematic hardware intervention to address the two root causes identified March 25:

- **New True Metrix strips** procured from CVS — cleaner signals, better electrode geometry compatibility
- **Shim addition** — standardized mechanical contact pressure; eliminated contact resistance variability
- **Cyclic voltammetry** run across all five concentrations, establishing 0.3 V as the optimal applied potential
- **Electrode rework** with defined lead polarity

<img width="390" height="296" alt="Chronoamperometry results after shim addition 6.6 mM" src="https://github.com/user-attachments/assets/8dbd1244-d30f-4f7c-bac8-d3d408b8e303" />

> **Figure 15.** Chronoamperometry at 6.6 mM post-shim (one of several results). After repositioning and consistent shim placement, more reliable Cottrell-like decay profiles began to emerge.

---

### April 26, 2026 — Final Calibration, Enclosure, and Prototype Completion

Final calibration performed with all hardware improvements in place. 3D-printed enclosure assembled with Arduino Uno and OLED display mounted and secured. End-to-end validation runs performed across all five concentrations.

---

## 7. SWOT Analysis

| | Strengths | Weaknesses |
|---|---|---|
| **Internal** | Strong medical relevance; real-time OLED output; low-cost component selection; modular architecture enabling independent subsystem development; CV-guided voltage selection demonstrates principled experimental design; complete end-to-end pipeline | Quantitative accuracy not yet at clinical-grade; computer-dependent operation; single-strip calibration with no uncertainty quantification; performance degrades above ~11 mM |

| | Opportunities | Threats |
|---|---|---|
| **External** | Rising diabetes prevalence increases urgency for affordable tools; future PCB integration could reduce cost by 10×; strong potential for community health and screening applications; embedded processing migration would enable full portability | Competition from established commercial glucometers with regulatory approval; regulatory barriers for clinical deployment; liability concerns with inaccurate predictions; possible fundamental incompatibility between commercial strip chemistry and non-proprietary instrumentation at high concentrations |

---

## 8. Future Work

### Immediate (Next Semester)

1. **Triplicate measurements per concentration** — provides calibration uncertainty bounds, enables outlier rejection, and produces a statistically valid regression. This is the single highest-impact improvement.
2. **Automated re-run logic** — automated quality flagging for out-of-range or low-quality signals (e.g., Cottrell $t^{-1/2}$ linearity check as a per-measurement flag)
3. **Reference glucometer validation** — compare device output against a commercial glucometer on matched samples to establish relative accuracy
4. **Larger calibration dataset** — more concentrations and more replicates to better characterize the nonlinear regime

### Hardware

- Migrate signal processing to onboard microcontroller (eliminate laptop dependency — the most significant usability limitation)
- Evaluate battery-powered operation for true portability
- Finalize strip holder design with integrated shim for robust, repeatable loading
- Integrate potentiostat analog front end directly with Arduino or upgraded MCU

### Longer-Term

- Miniaturization and PCB integration of the analog front end (targets ~10× cost reduction from the current Rodeostat)
- Regulatory pathway assessment for any future clinical or community deployment
- Explore alternative strip types with more consistent electrode-to-instrument compatibility

---

## Repository Contents

```
sweetspot/
├── notebooks/          # Jupyter Notebooks — chronoamperometry, calibration, serial communication
├── data/               # CSV/Excel exports of raw current-time data for each glucose standard
│   └── calibrationdata.xlsx
├── arduino/            # Arduino sketch — OLED display and serial communication
├── docs/               # Design reports, presentations, GANTT chart
└── README.md
```

### All Calibration Data

All calibration data collected over the semester:
[calibrationdata.xlsx](https://github.com/user-attachments/files/27147918/calibrationdata.xlsx)

### Reports and Presentations

- [Glucometer Preliminary Design Presentation](https://github.com/user-attachments/files/25115353/Glucometer.Preliminary.Design.Presentation.pdf)
- [Initial Design Report](https://github.com/user-attachments/files/25423979/Initial.Design.Report.pptx)
- [SweetSpot GANTT Chart](https://github.com/user-attachments/files/27147929/SweetSpot.GANTT.Chart.-.Gantt.Chart.Template.3.pdf)
- [SweetSpot Final Design Report.pptx](https://github.com/user-attachments/files/27449603/SweetSpot.Final.Design.Report.pptx)



---

## Current Project Status

**The system is a fully functional final prototype.** The sensing, calibration, display, and enclosure are all complete and integrated. The device demonstrates the full engineering chain from electrochemical signal to displayed concentration reading and meets the core design objectives established at project inception. Quantitative accuracy is the primary target for next-semester refinement.

### What Is Working

- [x] Electrochemical signal acquisition with the IO Rodeostat
- [x] CV-determined optimal applied voltage (0.3 V) for True Metrix strips
- [x] Shim-assisted strip holder for consistent mechanical contact
- [x] Reworked electrode connections with defined lead polarity
- [x] Python-based data collection, processing, and visualization in Jupyter Notebook
- [x] Export of raw data to CSV/Excel
- [x] Experimentally derived and implemented calibration curve (R² = 0.935)
- [x] Serial communication from Python to Arduino
- [x] Real-time OLED display of computed glucose concentration
- [x] Automated re-run for out-of-range results (<0 or >50 mM, up to 5×)
- [x] Soldered OLED and Arduino connections on breadboard
- [x] 3D-printed enclosure housing all display and processing components

### What Remains for Further Refinement

- [ ] Triplicate measurements per concentration for calibration uncertainty quantification
- [ ] Cottrell $t^{-1/2}$ linearity check as a per-measurement quality flag
- [ ] Reference glucometer comparison for absolute accuracy validation
- [ ] Full migration of processing to onboard microcontroller (fully standalone operation)
