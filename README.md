# SweetSpot Glucometer Project

![Status](https://img.shields.io/badge/status-intermediate%20prototype-orange)
![Platform](https://img.shields.io/badge/platform-Arduino%20%2B%20Python-blue)
![Hardware](https://img.shields.io/badge/hardware-IO%20Rodeostat-green)
![Display](https://img.shields.io/badge/display-128×64%20OLED-purple)

SweetSpot is a computer-dependent electrochemical glucose sensing prototype designed to evaluate whether a commercial glucose test strip, paired with an IO Rodeostat potentiostat, can be used to measure glucose concentration through chronoamperometry.

The long-term goal is to develop a low-cost and accessible glucometer that can measure glucose concentration, process the signal, and display the result in real time on an embedded screen. The project is being developed in stages, beginning with a laptop-based prototype for sensing validation and progressing toward a fully standalone embedded device.

---

- [About](#about)
- [Project Goals](#project-goals)
- [Why This Project Matters](#why-this-project-matters)
- [Scientific and Engineering Basis](#scientific-and-engineering-basis)
- [Preliminary Calculations](#preliminary-calculations)
- [Progress Log](#progress-log)
  - [February 12 — Initial Hardware Setup](#february-12-2026--initial-hardware-setup-and-signal-verification)
  - [February 26 — Calibration Data Collection](#february-26-2026--calibration-data-collection-and-pipeline-improvement)
  - [March 4 — Full Concentration Dataset](#march-4-2026--full-concentration-dataset-complete)
  - [March 5 — Initial Calibration Curve](#march-5-2026--initial-calibration-curve)
  - [March 25 — Validation Testing and Failure Analysis](#march-25-2026--validation-testing-and-failure-analysis)
  - [April 2 — Arduino Serial Communication and OLED Display](#april-2-2026--arduino-serial-communication-and-oled-display)
    - [Arduino IDE Setup](#arduino-ide-setup)
    - [Arduino Sketch](#arduino-sketch)
    - [Python Notebook Script](#python-notebook-script)
  - [April 8 — Hardware Integration and Soldering](#april-8-2026--hardware-integration-and-soldering)
- [Current Project Status](#current-project-status)
- [System Architecture](#system-architecture)
- [Prototype Plan](#prototype-plan)
- [Device Component Sketch](#device-component-sketch)
- [Path to Minimum Viable Product](#path-to-minimum-viable-product)
- [Market Relevance and Application](#market-relevance-and-application)
- [SWOT Analysis](#swot-analysis)
- [Future Work](#future-work)

---

## About

SweetSpot pairs a commercial glucose test strip with an IO Rodeostat potentiostat to measure glucose concentration electrochemically. A Python-based Jupyter Notebook workflow applies a chronoamperometric potential step, records the resulting current response, extracts a calibration feature, and transmits the computed glucose concentration over serial to an Arduino Uno, which displays the result on a 128×64 OLED screen.

At its current stage, the system demonstrates electrochemical signal acquisition, Python-based data processing, calibration curve construction, Arduino serial communication, and OLED display output. The OLED and Arduino connections have been soldered onto a breadboard as part of the transition toward a cased, portable device. The sensing model has not yet been fully validated for reliable glucose prediction across all repeated measurements.

---

## Project Goals

The project was designed to build a system that can:

- Run a chronoamperometry measurement on a commercial glucose test strip
- Extract a meaningful electrochemical feature from the current response (e.g., current at a fixed time point)
- Convert that feature into glucose concentration using an experimentally derived calibration curve
- Display the computed glucose concentration digitally in real time

The development plan includes two major stages:

1. **Computer-dependent prototype** — validates sensing, calibration, and display integration via laptop
2. **Standalone embedded prototype** — migrates all processing and display functions to embedded hardware

---

## Why This Project Matters

A glucometer is a portable medical device used to measure blood glucose concentration. It is essential for diabetes monitoring and management, especially for individuals who need frequent readings to guide daily decisions.

As diabetes prevalence continues to rise, the need for affordable and accessible glucose monitoring tools becomes more urgent. This challenge is particularly acute in underserved urban communities, where access to routine testing and early diagnosis may be limited.

In Philadelphia specifically, diabetes prevalence has increased by more than 50% over the past 15 years. Nearly one in three individuals living with diabetes in the city are unaware of their condition. In 2017, diabetes was the sixth leading cause of death in Philadelphia, accounting for 374 deaths. Our project is motivated by the idea that a lower-cost electrochemical sensing platform could eventually support broader glucose awareness and improve access to screening.

---

## Scientific and Engineering Basis

Electrochemical glucometers work by coupling a selective biochemical reaction to an electrical measurement.

Commercial glucose test strips typically rely on **glucose oxidase (GOx)**, which reacts selectively with glucose in the sample. In the presence of molecular oxygen, GOx catalyzes the oxidation of D-glucose to D-gluconolactone, producing hydrogen peroxide (H₂O₂) as a byproduct. This reaction provides the chemical selectivity necessary for accurate glucose sensing.

The generated H₂O₂ serves as the electroactive species. When a constant potential is applied by the potentiostat, H₂O₂ is oxidized at the working electrode, generating an electrical current proportional to the concentration of glucose originally present in the sample.

This project uses **chronoamperometry**, where the potential is stepped to a fixed value and the current is measured over time. Under ideal diffusion-controlled conditions, the time-dependent response follows the **Cottrell equation**:

$$i(t) = \frac{nFAC\sqrt{D}}{\sqrt{\pi t}}$$

Where:
- $i(t)$ = current at time $t$
- $n$ = number of electrons transferred
- $F$ = Faraday's constant
- $A$ = electrode area
- $C$ = analyte concentration
- $D$ = diffusion coefficient

This relationship predicts that current is proportional to analyte concentration at any fixed time point, providing the physical justification for using a single current value as a calibration feature. Features such as maximum current, current at a fixed time, or current decay behavior can in principle be used to estimate glucose concentration.

This theoretical framework guided our calibration strategy and feature selection throughout the project.

---

## Preliminary Calculations

### Blood Simulant (NaCl Stock Solution)

To mimic the electrolyte balance of human blood, we prepared a 100 mL stock solution at 140 mmol/L NaCl — the standard NaCl concentration in human blood.

$$\left(140 \frac{\text{mmol}}{\text{L}}\right)\left(\frac{1 \text{ mol}}{1000 \text{ mmol}}\right)\left(\frac{58.44 \text{ g}}{\text{mol}}\right)\left(\frac{100 \text{ mL}}{1000 \text{ mL/L}}\right) = 0.818 \text{ g NaCl}$$

### Glucose Standards

Glucose standards were prepared by dissolving known masses of glucose (MW = 180.156 g/mol) in 25 mL of stock solution:

$$C \text{ (mM)} = \frac{\text{mass (g)}}{180.156 \text{ g/mol} \times 0.025 \text{ L}} \times 1000$$

<img width="492" height="244" alt="Glucose Standards Table" src="https://github.com/user-attachments/assets/97abdf26-e984-4bfc-b0cf-d5142b466d45" />

These standards span the clinically relevant glucose range, from normal fasting levels through extreme diabetic concentrations, allowing us to characterize both the linear sensing regime and its breakdown at high concentrations.

### Back-of-the-Envelope Performance Estimate

Under ideal Cottrell behavior, we can estimate the expected current magnitude for a typical glucose concentration. For a glucose concentration of 5 mM (normal fasting):

- Diffusivity of H₂O₂ in aqueous solution: $D \approx 1.4 \times 10^{-5}$ cm²/s
- Electrode area: $A \approx 0.1$ cm² (estimated for commercial strip)
- Electrons transferred: $n = 2$
- At $t = 5$ s:

$$i = \frac{nFAC\sqrt{D}}{\sqrt{\pi t}} = \frac{2 \times 96485 \times 0.1 \times (5 \times 10^{-6}) \times \sqrt{1.4 \times 10^{-5}}}{\sqrt{\pi \times 5}} \approx 0.5 \ \mu\text{A}$$

This estimate is consistent with the current magnitudes observed experimentally (sub-μA to low-μA range), confirming that the Rodeostat is operating in the expected range and that the strip electrochemistry is at least partially compatible with the measurement protocol.

Deviations from this ideal — particularly at high concentrations or across repeated runs — point to the non-ideal effects (enzyme saturation, mass transport inconsistencies) discussed in the failure analysis below.

---

## Progress Log

### February 12, 2026 — Initial Hardware Setup and Signal Verification

We connected the IO Rodeostat potentiostat to a laptop and conducted an initial chronoamperometry test using a commercial glucose test strip and legacy connector leads. The goal was to verify that the Rodeostat could apply a potential step and record a measurable current response over time. At this stage, the emphasis was not yet on accurate glucose concentration values, but on confirming that the hardware and software could generate and capture an electrochemical signal in the expected format.

During this phase, we developed a Python script in a Jupyter Notebook to interface with the Rodeostat, collect data, and visualize the results. The script generated current-versus-time plots for each run, verifying transient current behavior following the applied voltage step.

**Engineering significance:** This phase functioned as a feasibility test of the acquisition system. Before a sensor can be evaluated quantitatively, it must first be shown that the instrumentation can consistently produce a usable signal. Confirming this signal pathway was a necessary prerequisite for all subsequent calibration, model development, and hardware integration.

---

### February 26, 2026 — Calibration Data Collection and Pipeline Improvement

We prepared a full set of standard glucose solutions for calibration and improved the data pipeline used to process experimental results. The updated Python workflow was designed not only to generate current-versus-time plots, but also to export raw data to CSV files for later analysis. This made the system more reproducible and better suited for engineering analysis.

At this stage, we collected our first full datasets for high and very high glucose concentrations, beginning the transition from simple system verification to quantitative sensor characterization.

---

### March 4, 2026 — Full Concentration Dataset Complete

We completed all initial concentration measurements across the prepared glucose standards. This marked an important transition from data collection to calibration analysis — the system had moved beyond simple proof of concept to evaluating whether its output could be used quantitatively.

With the complete dataset in hand, we began comparing the shape and magnitude of current responses across concentrations to determine whether a consistent and physically meaningful calibration relationship existed.

---

### March 5, 2026 — Initial Calibration Curve

Using the initial concentration data, we constructed our first calibration model relating electrochemical response to glucose concentration:

$$\text{Glucose concentration (mM)} = 1.1978 \times (\text{max current in } \mu\text{A}) - 0.4567$$

<img width="936" height="305" alt="Calibration Curve" src="https://github.com/user-attachments/assets/7304cc3f-2995-456e-9858-e5e4f0a6e959" />

This equation represented the first functional mapping from measured sensor output to predicted glucose concentration. The use of maximum current as the calibration feature was grounded in the Cottrell equation: under ideal diffusion-controlled conditions, current is proportional to concentration, so larger glucose concentrations should produce correspondingly larger electrochemical responses.

At this stage, the calibration equation appeared to provide a usable first approximation of concentration. However, further testing revealed that the relationship was not valid across the full concentration range.

#### Key Finding: Breakdown at High Concentrations

We observed that the calibration relationship fails at very high glucose concentrations, especially in extreme diabetic ranges. The response becomes nonlinear and unreliable, meaning that current no longer scales in a predictable way with concentration.

This behavior is consistent with the physical limitations of commercial glucose test strips, which are generally designed for typical physiological glucose ranges. At high concentrations, several non-ideal effects may occur:

- **Enzyme saturation** — GOx reaction rate can no longer increase proportionally with glucose concentration
- **Mass transport limitation** — the supply of glucose to the electrode surface, rather than reaction rate, begins to control the response
- **Deviation from Cottrell behavior** — the assumptions underlying the ideal chronoamperometric model no longer hold

From a chemical engineering perspective, this is a classic example of a process leaving its ideal operating regime — analogous to a reactor model failing when kinetic or mixing assumptions break down. The engineering implication is that the sensing system is only valid within a defined operating range, and any calibration model must respect those limits.

---

### March 25, 2026 — Validation Testing and Failure Analysis

Following the initial calibration, we carried out extensive validation testing to determine whether the system could reliably predict glucose concentration from new measurements. We evaluated two independent feature extraction methods:

- **Peak (maximum) current**
- **Steady-state current at a fixed time point**

<img width="927" height="587" alt="Validation Data" src="https://github.com/user-attachments/assets/6bb37bf7-06df-4d12-b4ae-94cb09e91898" />

Although calibration curves could be generated for both methods, neither produced predictions that were sufficiently accurate or reproducible across new trials. This indicates that the issue is not specific to a single feature extraction method, but reflects a broader limitation of the sensing system.

#### Engineering Interpretation of Failure

The fact that both methods failed suggests the system is not behaving according to the idealized diffusion-controlled Cottrell model. Several possible explanations are being considered:

**1. Strip–Instrumentation Mismatch**
Commercial glucose test strips are designed to operate with proprietary voltage waveforms, timing windows, and internal electronics. The simple chronoamperometric step applied by the Rodeostat may not match the strip's intended operating conditions. We assumed that this failure was contributing most to the unreliable readings. To rectify this, we retook all of our data samples. To better understand the strips, for each strip we first took a distilled water reading on them. Then, we wiped off the strip and took 2 readings of whatever sample we were recording data on.

**2. Mass Transport Limitations**
The Cottrell equation assumes a well-defined diffusion layer and predictable transport behavior. If glucose transport to the electrode surface is inconsistent or influenced by convection, geometry, or strip structure, current will vary independently of concentration.

**3. Nonlinear Reaction Kinetics / Enzyme Effects**
Strip chemistry may introduce nonlinearities due to enzyme kinetics, reaction intermediates, or saturation effects. If the reaction is not first-order in glucose concentration, current will not scale linearly, causing both calibration methods to fail.

**4. Signal Instability and Measurement Variability**
Even small variations in contact quality, strip condition, or timing can disproportionately affect measured current. Since both methods depend on signal consistency, this variability directly degrades predictive accuracy.

#### Hardware Verification

We physically inspected the Rodeostat, confirmed the device setup, and checked all wiring and connections. No clear hardware fault was identified. This supports the conclusion that the limitation is not a fundamental device failure, but rather incompatibility between the sensing method and strip measurement conditions.

**Engineering conclusion:** At this stage, the system cannot yet be considered a reliable glucometer. Although it successfully performs chronoamperometry and records current data, the measured signal cannot yet be translated into accurate concentration values with sufficient confidence. Both calibration approaches failed validation, indicating a systemic limitation rather than a methodological error.

---

### April 2, 2026 — Arduino Serial Communication and OLED Display

We focused on verifying that the OLED screen would work with our Arduino and began with a prototyping breadboard. After quite a few bumps, we realized we had to switch the OLED screen from I2C to SPI. Once we did this, the screen was quick to work and we validated the connection between our Arduino and the screen.

Below is the initial wiring set up.

<img width="698" height="402" alt="Screenshot 2026-04-08 at 5 02 33 PM" src="https://github.com/user-attachments/assets/7b3861d4-89a2-44fa-8873-8d93ec5980c7" />

#### OLED Wiring (SPI Interface)

The OLED display was wired to the Arduino Uno using the SPI protocol, following the pin mapping shown below:

| OLED Pin | Arduino Pin |
|----------|-------------|
| DC (Data/Command) | D8 |
| RES (Reset) | D9 |
| CS (Chip Select) | D10 |
| MOSI (Data) | D11 |
| SCK (Clock) | D13 |

**Reference:** [The Beginner's Guide to Display Text, Image & Animation on OLED Display by Arduino](https://www.instructables.com/The-Beginners-Guide-to-Display-Text-Image-Animatio/) was used to guide the initial OLED setup and text rendering.

Working Screen:

<img width="565" height="649" alt="Screenshot 2026-04-08 at 5 22 39 PM" src="https://github.com/user-attachments/assets/55357784-2a30-4411-ad7d-7bf7101c4b1a" />


#### Serial Communication Workflow

The Python Jupyter Notebook workflow was extended to:

1. Run the chronoamperometry measurement via the Rodeostat
2. Extract the calibration feature from the current-time response
3. Compute the predicted glucose concentration using the calibration equation
4. **Broadcast the concentration value to the Arduino over USB serial**

The Arduino sketch was written to:

1. Listen on the serial port for incoming concentration values from Python
2. Parse the received string
3. Display the glucose concentration on the OLED in real time

This completed the full sensing-to-display pipeline within the computer-dependent prototype architecture.

#### Arduino IDE Setup

The Arduino sketch depends on two libraries that must be installed via the Arduino IDE Library Manager before uploading:

- **Adafruit SSD1306** — OLED display driver
- **Adafruit GFX** — graphics primitives (dependency of SSD1306)

To install: open Arduino IDE → Sketch → Include Library → Manage Libraries → search for each library by name → Install.

The sketch was uploaded to the Arduino Uno over USB before running the Python notebook. Once uploaded, the Arduino listens on Serial at 115200 baud and waits for a newline-terminated concentration string from Python.

#### Arduino Sketch

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

  // Header label
  display.setTextSize(1);
  display.setCursor(18, 6);
  display.print("GLUCOSE");

  // Divider line
  display.drawLine(0, 18, 127, 18, SSD1306_WHITE);

  // Large concentration value
  display.setTextSize(3);
  display.setCursor(10, 30);
  display.print(value);

  // Units
  display.setTextSize(1);
  display.setCursor(92, 52);
  display.print("mg/dL");

  display.display();
}

void setup() {
  Serial.begin(115200);
  delay(2000);

  if (!display.begin(SSD1306_SWITCHCAPVCC)) {
    while (1) {}  // halt if display not found
  }

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

#### Python Notebook Script

The following script runs the full pipeline end-to-end: connecting to both the Rodeostat and the OLED, running the chronoamperometry measurement, extracting the peak current, computing concentration via the calibration equation, and sending the result to the OLED over serial.


```python
from potentiostat import Potentiostat
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import serial
import serial.tools.list_ports
import time

# -----------------------------
# 0) SHOW AVAILABLE PORTS
# -----------------------------
print("Available ports:")
for p in serial.tools.list_ports.comports():
    print(" ", p.device, "-", p.description)

# -----------------------------
# 1) SET PORT NAMES
# -----------------------------
# Update these to match your system (run block 0 first to find them)
rodeostat_port = "/dev/cu.usbmodem101"
oled_port      = "/dev/cu.usbmodem11101"

# -----------------------------
# 2) CONNECT TO OLED (Arduino)
# -----------------------------
oled = serial.Serial(oled_port, 115200, timeout=1)
time.sleep(2.5)
oled.reset_input_buffer()
oled.reset_output_buffer()
print("OLED connected on", oled_port)

# -----------------------------
# 3) CONNECT TO RODEOSTAT
# -----------------------------
dev = Potentiostat(rodeostat_port)
# Note: set_curr_range is skipped because the installed library version
# raises KeyError: '10V_microAmpV0.2' for that call.
# dev.set_curr_range("100uA")
dev.set_sample_period(100)  # ms

datafile = "data.txt"

# -----------------------------
# 4) RUN CHRONOAMPEROMETRY
# -----------------------------
name = "chronoamp"
test_param = {
    "quietValue": 0.0,
    "quietTime":  1000,
    "step": [
        (0,     0.0),
        (29000, 0.5),
    ],
}

dev.set_param(name, test_param)
print("Running test...")
t, volt, curr = dev.run_test(name, display="pbar", filename=datafile)

# -----------------------------
# 5) BUILD DATAFRAME
# -----------------------------
df_raw = pd.DataFrame({
    "time_s":     t,
    "voltage_V":  volt,
    "current_uA": curr
})
df_raw = df_raw.sort_values("time_s").reset_index(drop=True)
df_raw["time_s"] = df_raw["time_s"].round(2)

# -----------------------------
# 6) SAVE DATA
# -----------------------------
out_file = "chrono_data.xlsx"
df_raw.to_excel(out_file, index=False)
print("Saved:", out_file)

# -----------------------------
# 7) PLOT
# -----------------------------
plt.figure(figsize=(9, 6))

plt.subplot(211)
plt.title("Voltage and current vs time")
plt.plot(t, volt)
plt.ylabel("potential (V)")
plt.grid(True)

plt.subplot(212)
plt.plot(t, curr)
plt.ylabel("current (uA)")
plt.xlabel("time (sec)")
plt.grid(True)

plt.tight_layout()
plt.show()

# -----------------------------
# 8) EXTRACT MAX CURRENT
# -----------------------------
# Ignore first 1.0 s to reduce startup noise
df_window = df_raw[df_raw["time_s"] >= 1.0]

# Use max for an upward peak response
max_current_uA = float(df_window["current_uA"].max())
# If signal is a downward dip, use .min() instead:
# max_current_uA = float(df_window["current_uA"].min())

print("Max current (uA):", max_current_uA)

# -----------------------------
# 9) COMPUTE CONCENTRATION
# -----------------------------
# Calibration: current = m * concentration + b
# → concentration = (current - b) / m
# Replace CAL_SLOPE and CAL_INTERCEPT with your experimentally derived values.
CAL_SLOPE     = 1.0   # replace with your m
CAL_INTERCEPT = 0.0   # replace with your b

concentration = (max_current_uA - CAL_INTERCEPT) / CAL_SLOPE
concentration = max(0, concentration)  # clamp to non-negative
print("Concentration:", concentration)

# -----------------------------
# 10) SEND TO OLED
# -----------------------------
oled.write(f"{concentration:.1f}\n".encode())
time.sleep(0.3)

while oled.in_waiting:
    print("OLED:", oled.readline().decode(errors="ignore").strip())

# -----------------------------
# 11) CLEANUP
# -----------------------------
oled.close()

# -----------------------------
# 12) DEBUG PRINTS
# -----------------------------
print("t[:10]    =", t[:10])
print("volt[:10] =", volt[:10])
print("curr[:10] =", curr[:10])
print(df_raw.head(10))
```

---

### April 8, 2026 — Hardware Integration and Soldering

This week marked a significant step toward a more robust and portable device. The system moved from loose jumper wire connections to a soldered assembly.

**Key updates:**

- Successfully connected the Rodeostat, Arduino Uno, and OLED display together through Python running in Jupyter Notebook — the full pipeline (measure → process → display) now operates end-to-end from a single notebook
- The OLED screen was soldered onto a breadboard, along with the wiring connections to the Arduino, replacing previous loose jumper wire connections with more reliable soldered joints
- This soldered configuration is more mechanically stable and represents the first step toward mounting all components within a physical enclosure

**Current hardware state:** The system is operational as a benchtop prototype. The next phase focuses on enclosure design and component mounting.

---





## Current Project Status

The system presently functions as a **computer-dependent electrochemical sensing prototype with active hardware integration in progress**.

**Copy link to watch it in action!**
https://github.com/user-attachments/assets/cb9e7f55-9fd3-4ae5-9c2e-26083d2b7484

### What is working

- [x] Electrochemical signal acquisition with the IO Rodeostat
- [x] Python-based data collection in Jupyter Notebook
- [x] Current-versus-time visualization
- [x] Export of raw data to CSV
- [x] Construction of calibration relationships from experimental data
- [x] Serial communication from Python to Arduino
- [x] OLED display of computed glucose concentration
- [x] Soldered OLED and Arduino connections on breadboard

### What is not yet working

- [ ] Reliable glucose concentration prediction across repeated measurements
- [ ] Fully validated sensing model
- [ ] Standalone device operation (no laptop required)
- [ ] Physical enclosure / casing

Although the system can successfully run chronoamperometry, process the signal, and display a result on the OLED, the measured signal does not yet translate into sufficiently accurate and reproducible glucose predictions to constitute a validated glucometer.

---

## System Architecture

<img width="1137" height="653" alt="Screenshot 2026-04-08 at 5 31 58 PM" src="https://github.com/user-attachments/assets/cff04c35-f222-4edc-b9f9-14935a0425f3" />


---

## Prototype Plan

### Prototype 1: Computer-Dependent System *(current)*

In the first prototype, the laptop serves as the primary control and data-processing platform. The Rodeostat performs the chronoamperometric measurement, Python handles signal processing and calibration, and the computed glucose value is transmitted to the Arduino over USB serial for display on the OLED.

**Purpose:**
- Validate electrochemical sensing performance and calibration methodology
- Leverage a high-level computing environment to simplify data processing
- Minimize embedded system complexity during early development
- Enable rapid iteration, debugging, and performance evaluation

**Current hardware state:** OLED and Arduino connections are soldered onto a breadboard. The system runs end-to-end from a single Jupyter Notebook.

---

## Path to Minimum Viable Product

The minimum viable product (MVP) is defined as a device that can:

1. Accept a commercial glucose test strip
2. Run a chronoamperometric measurement automatically
3. Compute glucose concentration from the measured signal
4. Display the result on the OLED screen within a few seconds
5. Be fully enclosed within a 3D printed box
6. Calculate error margins to write out specs of our device

The current gap between the prototype and the MVP centers on two parallel tracks:

**Track 1 — Sensing Validation**
- Resolve measurement reproducibility
- Calculate error margins
- Normalize strip behavior with distilled water readings
- Decide on what curve works best for the most amount of concentrations
- Characterize the validated operating range of the sensor

**Track 2 — Hardware Maturation (in progress)**
- ~~Solder OLED and Arduino connections~~ ✅ Complete
- ~~Establish Python → Arduino serial communication~~ ✅ Complete
- Design and fabricate physical enclosure / casing
- Mount all components securely within the case

---

## Market Relevance and Application

The long-term target market for this device is underserved and low-income populations, particularly in urban environments where diabetes prevalence is high and access to healthcare may be limited.

Potential user groups include:

- Adults aged 30–65 at elevated risk for Type 2 diabetes
- Low-income individuals and families
- Undiagnosed or pre-diabetic individuals not yet engaged in regular monitoring
- Community health clinics, federally qualified health centers (FQHCs), and nonprofit screening programs
- Students, uninsured individuals, and older adults who need simpler, lower-cost monitoring options

**Distribution pathways:**
- Community health clinics and FQHCs
- Nonprofit organizations focused on chronic disease prevention
- Local health departments and diabetes outreach programs
- University- or hospital-affiliated screening initiatives

The larger design goal is not to create a premium consumer medical device, but to explore the feasibility of a low-cost sensing system that could support broader glucose awareness and improve access to screening in communities where it is needed most.

---

## SWOT Analysis

### Strengths

- Strong medical relevance and recurring demand for glucose monitoring
- Real-time signal acquisition and direct display feedback
- Portable sensing concept with low-cost component choices
- Modular architecture enables independent development of sensing and display layers
- Interdisciplinary combination of electrochemistry, instrumentation, and embedded systems

### Weaknesses

- Sensing accuracy not yet validated across repeated measurements
- Current calibration relationships are not sufficiently reproducible
- Dependence on laptop-based processing in current prototype
- Hardware integration (enclosure, standalone operation) remains incomplete
- Small-scale development limits practical deployment at this stage

### Opportunities

- Rising diabetes prevalence increases urgency for affordable monitoring tools
- Future optimization could reduce manufacturing cost and improve scalability
- Advances in embedded electronics and biosensor design offer pathways for refinement
- Strong potential for educational, screening, and community health applications
- Academic setting provides access to fabrication and testing resources

### Threats

- Competition from established commercial glucometers with regulatory approval
- Regulatory barriers for medical sensing devices intended for clinical use
- Liability concerns associated with inaccurate glucose prediction
- Possible fundamental incompatibility between commercial strips and non-proprietary instrumentation

---

## Future Work

### Immediate Focus — Resolve Sensing Limitation

- Test alternative voltage protocols that better match the strip's intended operating conditions
- Explore additional signal features (integrated current, decay curve fitting, multi-point regression)
- Improve signal processing through smoothing, noise filtering, and baseline correction
- Investigate strip-specific operating requirements and manufacturer documentation
- Evaluate whether commercial strips are fundamentally compatible with Rodeostat-based sensing across the relevant concentration range
- Develop a more robust calibration model with defined operating bounds and uncertainty estimates

### Hardware Progression

- Design and fabricate a physical enclosure to house all components
- Evaluate form factor and ergonomics for practical use
- Test battery-powered operation for portability
- Migrate processing from laptop to onboard microcontroller (Prototype 2)
- Integrate potentiostat analog front end directly with Arduino or upgraded MCU

### Longer-Term

- Validate the full system against a commercial glucometer reference
- Evaluate performance across a wider subject population
- Investigate miniaturization and PCB integration of the analog front end
- Assess regulatory pathway for any future clinical or community deployment

---

## Final Project Definition

The current system is best described as a:

> **Computer-dependent electrochemical sensing prototype with active hardware integration in progress**

It successfully demonstrates:

- Electrochemical signal acquisition
- Python-based data processing and calibration analysis
- Serial communication from Python to Arduino
- Real-time OLED display of computed glucose concentration
- Soldered hardware assembly as a step toward a cased portable device

It does not yet demonstrate:

- Reliable quantitative glucose sensing across repeated measurements
- Validated concentration prediction
- Finished physical enclosure

That makes the current project a successful **sensing-platform and display-integration prototype**, advancing toward but not yet achieving a validated standalone glucometer.

---

## Repository Contents

- `notebooks/` — Jupyter Notebooks for chronoamperometry, calibration, and serial communication
- `data/` — CSV exports of raw current-time data for each glucose standard
- `arduino/` — Arduino sketch for OLED display and serial communication
- `docs/` — Preliminary Design Report, Initial Design Report, GANTT chart
- `README.md` — This file

### Reports and Presentations

- [Glucometer Preliminary Design Presentation](https://github.com/user-attachments/files/25115353/Glucometer.Preliminary.Design.Presentation.pdf)
- [SweetSpot GANTT Chart][SweetSpot GANTT Chart - Gantt Chart Template (2).pdf](https://github.com/user-attachments/files/26580797/SweetSpot.GANTT.Chart.-.Gantt.Chart.Template.2.pdf)
- [Initial Design Report](https://github.com/user-attachments/files/25423979/Initial.Design.Report.pptx)
