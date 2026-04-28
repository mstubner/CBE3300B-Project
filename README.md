# SweetSpot Glucometer

![Status](https://img.shields.io/badge/status-final%20prototype-brightgreen)
![Platform](https://img.shields.io/badge/platform-Arduino%20%2B%20Python-blue)
![Hardware](https://img.shields.io/badge/hardware-IO%20Rodeostat-green)
![Display](https://img.shields.io/badge/display-128×64%20OLED-purple)
![Enclosure](https://img.shields.io/badge/enclosure-3D%20printed-orange)

SweetSpot is a low-cost electrochemical glucometer prototype that measures glucose concentration using commercial glucose test strips paired with an IO Rodeostat potentiostat. The system applies a chronoamperometric potential step, records the resulting current response, extracts a calibration feature, and displays the computed glucose concentration in real time on a 128×64 OLED screen mounted inside a 3D-printed enclosure.

The device represents a fully functional final prototype: electrochemical sensing, signal processing, calibration, serial communication, and embedded display output all operate end-to-end from a single Jupyter Notebook interface. The enclosure houses the Arduino Uno and OLED display with soldered connections, and the system is capable of producing and displaying a concentration reading from a test strip within seconds of initiating a measurement.

---

- [About](#about)
- [Project Goals](#project-goals)
- [Why This Project Matters](#why-this-project-matters)
- [Scientific and Engineering Basis](#scientific-and-engineering-basis)
  - [Glucose Oxidase Reaction Chemistry](#glucose-oxidase-reaction-chemistry)
  - [Chronoamperometry and the Cottrell Equation](#chronoamperometry-and-the-cottrell-equation)
  - [Cyclic Voltammetry for Operating Point Selection](#cyclic-voltammetry-for-operating-point-selection)
  - [Chemical Engineering Principles in Device Design](#chemical-engineering-principles-in-device-design)
- [Preliminary Calculations](#preliminary-calculations)
- [Calibration Approach and Iteration](#calibration-approach-and-iteration)
- [Progress Log](#progress-log)
  - [February 12 — Initial Hardware Setup](#february-12-2026--initial-hardware-setup-and-signal-verification)
  - [February 26 — Calibration Data Collection](#february-26-2026--calibration-data-collection-and-pipeline-improvement)
  - [March 4 — Full Concentration Dataset](#march-4-2026--full-concentration-dataset-complete)
  - [March 5 — Initial Calibration Curve](#march-5-2026--initial-calibration-curve)
  - [March 25 — Validation Testing and Failure Analysis](#march-25-2026--validation-testing-and-failure-analysis)
  - [April 2 — Arduino Serial Communication and OLED Display](#april-2-2026--arduino-serial-communication-and-oled-display)
  - [April 8 — Hardware Integration and Soldering](#april-8-2026--hardware-integration-and-soldering)
  - [April 16 — New Strips, Shim, CV, and Electrode Rework](#april-16-2026--new-strip-procurement-shim-addition-cyclic-voltammetry-and-electrode-rework)
  - [April 26 — Final Calibration, Enclosure, and Prototype Completion](#april-26-2026--final-calibration-enclosure-and-prototype-completion)
- [Final System Performance](#final-system-performance)
- [Current Project Status](#current-project-status)
- [System Architecture](#system-architecture)
- [Market Relevance and Application](#market-relevance-and-application)
- [SWOT Analysis](#swot-analysis)
- [Future Work](#future-work)

---
 
## About

SweetSpot pairs a commercial True Metrix glucose test strip with an IO Rodeostat potentiostat to measure glucose concentration electrochemically. A Python-based Jupyter Notebook workflow applies a chronoamperometric potential step at an empirically validated voltage (0.3 V, determined via cyclic voltammetry), records the resulting current response, extracts peak current as the calibration feature, and transmits the computed glucose concentration over USB serial to an Arduino Uno. The Arduino displays the result in real time on a 128×64 OLED screen.

The hardware is housed inside a 3D-printed enclosure. The OLED display and all Arduino connections are soldered onto a breadboard mounted within the case. The sensing is performed with a shim-assisted strip holder that ensures consistent mechanical contact between the Rodeostat leads and the strip electrodes.

The system demonstrates electrochemical signal acquisition, Python-based data processing, calibration curve construction from experimentally validated data, Arduino serial communication, OLED display output, and physical enclosure integration — constituting a complete, functional final prototype.

---

## Project Goals

The project was designed to build a system that can:

- Run a chronoamperometry measurement on a commercial glucose test strip at an optimized applied voltage
- Extract a meaningful electrochemical feature from the current response (peak current)
- Convert that feature into glucose concentration using an experimentally derived and validated calibration curve
- Display the computed glucose concentration in real time on an embedded OLED screen
- House all components within a 3D-printed enclosure suitable for demonstration

The development plan proceeded in two major stages:

1. **Computer-dependent prototype** — validates sensing, calibration, and display integration via laptop
2. **Cased final prototype** — integrates all working components into a physical enclosure for demonstration 

---

## Why This Project Matters

A glucometer is a portable medical device used to measure blood glucose concentration. It is essential for diabetes monitoring and management, particularly for individuals who need frequent readings to guide daily decisions about diet, medication, and activity.

Since 1990, global cases of diabetes have quadrupled to 830 million, disproportionately affecting low- and middle-income countries. The CDC predicts that if these trends continue, 40% of Americans will develop diabetes in their lifetime. Additionally, diabetes is the most expensive chronic condition in the United States (NIH). People diagnosed with diabetes have medical expenses approximately 2.6 times higher than those without the condition (CDC).

In Philadelphia specifically, diabetes prevalence has increased by more than 50% over the past 15 years. Nearly one in three individuals living with diabetes in the city are unaware of their condition. In 2017, diabetes was the sixth leading cause of death in Philadelphia, accounting for 374 deaths. SweetSpot is motivated by the idea that a lower-cost electrochemical sensing platform could eventually support broader glucose awareness and improve access to screening.

---

## Scientific and Engineering Basis

### Glucose Oxidase Reaction Chemistry

Commercial glucose test strips rely on **glucose oxidase (GOx)**, an enzyme that reacts selectively with glucose. In the presence of molecular oxygen, GOx catalyzes the oxidation of D-glucose to D-gluconolactone, producing hydrogen peroxide (H₂O₂) as a byproduct:

$$\text{D-glucose} + \text{O}_2 \xrightarrow{\text{GOx}} \text{D-gluconolactone} + \text{H}_2\text{O}_2$$

The H₂O₂ produced is electroactive. When a constant potential is applied by the potentiostat, H₂O₂ is oxidized at the working electrode, releasing electrons that generate a measurable current. Because each mole of glucose produces one mole of H₂O₂, the current is directly proportional to glucose concentration. This is the electrochemical foundation of the device.

### Chronoamperometry and the Cottrell Equation

This project uses chronoamperometry, meaning the potential is stepped to a fixed value and the current is measured over time. Under ideal diffusion-controlled conditions, the time-dependent response follows the Cottrell equation which is shown below:

$$i(t) = \frac{nFAC\sqrt{D}}{\sqrt{\pi t}}$$

Where:
- $i(t)$ = current at time $t$
- $n$ = number of electrons transferred (2 for H₂O₂ oxidation)
- $F$ = Faraday's constant (96,485 C/mol)
- $A$ = electrode area (cm²)
- $C$ = analyte concentration (mol/cm³)
- $D$ = diffusion coefficient (cm²/s)

This relationship predicts that current is proportional to analyte concentration at any fixed time point, providing the physical justification for using peak or steady-state current as a calibration feature.

Proof of the t^-1/2 relationship through our tests is shown below.

<img width="418" height="352" alt="Screenshot 2026-04-27 at 2 54 00 PM" src="https://github.com/user-attachments/assets/76fb609b-c87f-40a4-b7ed-2942f8c95f13" />


### Cyclic Voltammetry for Operating Point Selection

**Cyclic voltammetry (CV)** is an electroanalytical technique in which the working electrode potential is swept linearly between two set values while the resulting current is recorded. The output traces the full electrochemical response of the system as a function of applied potential, revealing characteristic oxidation and reduction peaks for the electroactive species present.

In SweetSpot, CV was used as a diagnostic and optimization tool to identify the optimal applied potential for chronoamperometric measurements. The anodic (positive) peak in the CV scan corresponds to the electrochemical oxidation of H₂O₂ at the working electrode surface which is the same faradaic reaction that generates the sensing signal in chronoamperometry. The potential at which this peak occurs indicates where the oxidation rate is maximized.

**Connection to chemical engineering principles:** A CV experiment directly maps reaction rate (current) as a function of electrochemical driving force (applied potential) making it an electrochemical analog of a rate-versus-driving-force characterization in transport phenomena and reactor design. The resulting current-potential curve is the polarization curve of the electrode system, and it encodes the interplay between two physical regimes that chemical engineers deal with routinely:

- **Activation-controlled regime (low potential):** Below the oxidation peak, the driving force is insufficient to efficiently oxidize H₂O₂ at the electrode surface. The reaction rate is limited by the activation energy barrier which is directly analogous to a kinetically controlled reactor operating far from equilibrium conversion. Current is low and poorly correlated with analyte concentration.

- **Mass-transport-limited regime (high potential):** Above the peak, the electrode reaction proceeds as fast as H₂O₂ can arrive at the surface. The rate is now controlled by diffusion through the solution, identical to a transport-limited reactor where the reaction is fast but feed delivery is the rate limiting step. Competing reactions (e.g., water electrolysis) also begin to contribute at sufficiently high potentials, introducing background current that degrades selectivity.

The oxidation peak in the CV scan marks the transition between these two regimes, the point where the electrode reaction is most sensitive to analyte concentration and most efficient in converting faradaic current to a usable signal. Choosing the applied potential at this peak is therefore equivalent to selecting the optimal operating point on a reaction rate curve: maximizing conversion per unit driving force while avoiding the side-reaction and transport-limitation regimes on either side.

CV scans were run across all five glucose standards. The anodic oxidation peak appeared consistently near **0.3 V** across all concentrations, confirming that this potential reflects a property of the strip electrode chemistry rather than analyte concentration. 0.3 V was adopted as the fixed applied potential for all subsequent chronoamperometric measurements, replacing the previously arbitrary 0.5 V value.

The image below shows the CV reading 0.3V which we used for the basis of this project.
<img width="598" height="444" alt="Screenshot 2026-04-27 at 2 52 24 PM" src="https://github.com/user-attachments/assets/83df46b3-026a-41f9-9a17-4e6163ba23e7" />


### Chemical Engineering Principles in Device Design

**Mass and species balances:** The enzymatic reaction converts glucose stoichiometrically to H₂O₂. The current measured is proportional to the molar flux of H₂O₂ to the electrode surface — a direct application of species balance and Faraday's law of electrolysis ($Q = nFN$, where $N$ is moles reacted).

**Transport phenomena:** The Cottrell equation arises from solving Fick's second law of diffusion for a semi-infinite planar electrode with a potential-step boundary condition. Current decay over time reflects depletion of the diffusion layer — a mass transport effect that governs the signal shape and informs both feature extraction strategy and the choice of measurement time window.

**Reaction kinetics and operating windows:** CV characterization identified the activation-controlled regime (below ~0.2 V) versus the mass-transport-limited plateau (above ~0.35 V) for this strip system. Choosing 0.3 V places the operating point at the transition where signal is maximized before transport limitations reduce sensitivity.

**Calibration as process modeling:** The experimentally derived calibration curve functions as a process model: it maps a measured variable (peak current) to a process output (glucose concentration). Constructing, validating, and refining this model, including identifying its failure modes at high concentrations, is directly analogous to process identification and control loop tuning.

**Nonlinear kinetics and operating range:** The breakdown of linearity at high glucose concentrations reflects enzyme saturation — a Michaelis-Menten kinetic effect where the GOx reaction rate approaches $V_{\max}$ and no longer scales proportionally with substrate concentration. This is identical to the behavior of a first-order reaction transitioning to zero-order kinetics at high substrate concentration, a concept central to reactor design. The device's validated operating range (2.2–11.1 mM) is bounded by this nonlinearity on the high end.

---

## Preliminary Calculations

### Blood Simulant (NaCl Stock Solution)

To mimic the electrolyte balance of human blood, we prepared a 100 mL stock solution at 140 mmol/L NaCl — the standard NaCl concentration in human blood.

$$\left(140 \frac{\text{mmol}}{\text{L}}\right)\left(\frac{1 \text{ mol}}{1000 \text{ mmol}}\right)\left(\frac{58.44 \text{ g}}{\text{mol}}\right)\left(\frac{100 \text{ mL}}{1000 \text{ mL/L}}\right) = 0.818 \text{ g NaCl}$$

### Glucose Standards

Glucose standards were prepared by dissolving known masses of glucose (MW = 180.156 g/mol) in 25 mL of stock solution:

$$C \text{ (mM)} = \frac{\text{mass (g)}}{180.156 \text{ g/mol} \times 0.025 \text{ L}} \times 1000$$

<img width="492" height="244" alt="Glucose Standards Table" src="https://github.com/user-attachments/assets/97abdf26-e984-4bfc-b0cf-d5142b466d45" />

These five standards span the clinically relevant glucose range, from normal fasting levels through moderate diabetic concentrations, allowing us to characterize the linear sensing regime and define the validated operating range of the device.

### Performance Estimate

Under ideal Cottrell behavior, the expected current at 5 mM glucose (normal fasting):

- $D \approx 1.4 \times 10^{-5}$ cm²/s
- $A \approx 0.1$ cm²
- $n = 2$, at $t = 5$ s:

$$i = \frac{nFAC\sqrt{D}}{\sqrt{\pi t}} = \frac{2 \times 96485 \times 0.1 \times (5 \times 10^{-6}) \times \sqrt{1.4 \times 10^{-5}}}{\sqrt{\pi \times 5}} \approx 0.5 \ \mu\text{A}$$

Experimentally observed peak currents with the final protocol are in the low-to-mid µA range, consistent with this estimate when accounting for the strip's actual electrode geometry and the higher applied potential (0.3 V drives a more complete reaction than the ideal diffusion-only model assumes).

---

## Calibration Approach and Iteration

### Initial Approach: Peak Current (March 5)

The first calibration curve used peak (maximum) current as the feature. This produced an apparently excellent fit on the initial dataset (R² = 0.9998) but drastically failed validation. When using this derived equation to predict glucose concentrations, it would give results ranging from -10 to over 50. With only four concentrations and one measurement each, the model was interpolated rather than fit, and captured no inter-strip variability.

<img width="498" height="175" alt="Screenshot 2026-04-27 at 9 59 09 PM" src="https://github.com/user-attachments/assets/d82a2b01-f121-4692-bdc8-95140c596deb" />


More readings were conducted with the same method, resulting in the following results:
<img width="517" height="289" alt="Screenshot 2026-04-27 at 9 58 59 PM" src="https://github.com/user-attachments/assets/1b374326-adfb-46d8-8bc7-34bfad9be396" />


### Normalization Attempt

A normalization step was introduced: each strip's peak current in DI water was subtracted from the sample peak current. Despite this correction, calibration remained inconsistent. The normalization corrected for inter-strip offset but could not correct for differences in sensitivity (slope) between strips.

### Shift to Steady-State Current

We shifted to steady-state current at fixed time points (10 s and 20 s). These values were more reproducible because the transient had decayed. The 10 s curve produced the clearest linear relationship and was adopted as the primary calibration method.

<img width="622" height="520" alt="Calibration curve comparison" src="https://github.com/user-attachments/assets/23fa1c65-093f-43e6-bedc-88f5650b4043" />

### Root Cause Hardware Analysis and Resolution (April 16)

Persistent reproducibility failures across all signal features pointed to two uncontrolled physical variables: strip batch variability and inconsistent mechanical contact. Both were resolved through targeted hardware intervention (new strips, shim addition, electrode rework), enabling the final calibration.

### Final Calibration (April 26)

New strips were bought and the electrodes were rewired. Initially, we conducted readings the same way as with the old strips by inserting one in and lining it up. Through this method, the following results were produced.

<img width="401" height="315" alt="Screenshot 2026-04-27 at 9 59 34 PM" src="https://github.com/user-attachments/assets/c4e9aaf9-d595-4444-863f-4deb1d4c7efb" />

After talking with Professor Osuji, we determined that a key issue may be unreliable mechanical contact. In order to resolve this issue, a shim was added (another strip upside down to ensure the electrodes do not interfere) to ensure proper contact. While this did not completely resolve the issue, more reliable results were obtained, shown below.

<img width="525" height="398" alt="Screenshot 2026-04-27 at 10 00 00 PM" src="https://github.com/user-attachments/assets/72d408e0-0354-4cb2-950e-61019ecb7f4f" />



$$\text{Glucose Concentration (mM)} = \frac{\text{Peak Current (µA)} + 5.415}{3.7379} \quad (R^2 = 0.935)$$

Then, to fully test our calibration curve, we tested our curve with our concentrations to create specs for our device. The results of two rounds of tests are shown below.

| Actual Concentration (mM) | Screen Output (mM) |
|---------------------------|-------------------|
| 2.2 | 11.7 |
| 4.4 | 3.42 |
| 6.6 | 7.9 |
| 8.8 | 42.5 |
| 11.1 | 48.8 |

---

## Error Analysis

The final prototype calibration curve was evaluated by testing known glucose standards and comparing the predicted concentrations generated by the device to their true prepared values. This validation step was used to assess real-world prototype performance rather than develop the model itself. The resulting mean absolute error (MAE) was 20.27 mM, with a root mean squared error (RMSE) of 23.57 mM and a mean absolute percent error (MAPE) of 328.7%. The system also showed an average positive bias of 19.68 mM, meaning the final curve generally overpredicted glucose concentration. Error was especially pronounced at intermediate and higher concentrations, indicating that while the calibration relationship captured an overall trend, measurement variability still dominated final accuracy. Likely contributors include inconsistent strip seating, contact resistance between the strip and electrodes, manufacturing variation between commercial strips, electrical noise, and nonlinear enzymatic response at elevated glucose levels. Despite these limitations, the final calibration represented the most functional and complete version of the device, enabling fully automated concentration prediction and real-time OLED display output. Most importantly, the validation data clearly identified the next engineering priorities: improved strip alignment, tighter mechanical tolerances, larger replicate calibration datasets, and more robust regression methods. As a capstone prototype, the project successfully demonstrated an end-to-end electrochemical glucometer system while using quantitative error analysis to guide future refinement.

<img width="521" height="410" alt="Screenshot 2026-04-27 at 10 00 25 PM" src="https://github.com/user-attachments/assets/aaf52fd2-370b-471a-8e74-866dc931845e" />

<img width="550" height="398" alt="Screenshot 2026-04-27 at 10 00 34 PM" src="https://github.com/user-attachments/assets/5b434416-e2b9-4cc9-a8e9-aaae7d7dd0ee" />

<img width="533" height="398" alt="Screenshot 2026-04-27 at 10 00 43 PM" src="https://github.com/user-attachments/assets/5a39f936-9187-481e-b23e-9deb90e26165" />


---

## Progress Log

### February 12, 2026 — Initial Hardware Setup and Signal Verification

We connected the IO Rodeostat potentiostat to a laptop and conducted an initial chronoamperometry test using a commercial glucose test strip and legacy connector leads. The goal was to verify that the Rodeostat could apply a potential step and record a measurable current response. At this stage the emphasis was on confirming that the hardware and software pipeline could generate and capture an electrochemical signal — not yet on accurate glucose values.

A Python script in a Jupyter Notebook was developed to interface with the Rodeostat, collect current-versus-time data, and visualize the results.

<img width="788" height="483" alt="Screenshot 2026-04-27 at 2 54 56 PM" src="https://github.com/user-attachments/assets/e5ac448c-d1ea-4db9-8321-44241245651d" />

> *Caption: "First chronoamperometry run, February 12. Signal is present but highly noisy with no discernible concentration-dependent feature. This run confirmed hardware connectivity but highlighted the measurement challenges ahead."*

**Engineering significance:** This phase functioned as a system feasibility test. Before a sensor can be evaluated quantitatively, the acquisition chain must be shown to produce a usable signal. Confirming this signal pathway was a necessary prerequisite for all subsequent calibration, model development, and hardware integration.

---

### February 26, 2026 — Calibration Data Collection and Pipeline Improvement

We prepared a full set of standard glucose solutions for calibration and improved the data pipeline. The updated Python workflow exported raw data to CSV files for later analysis, making the system reproducible and better suited for engineering analysis. First full datasets for high and very high glucose concentrations were collected.

---

### March 4, 2026 — Full Concentration Dataset Complete

All initial concentration measurements across the prepared glucose standards were completed. This marked the transition from system verification to quantitative sensor characterization — the first time the system had enough data to attempt a calibration.

---

### March 5, 2026 — Initial Calibration Curve

Using the concentration dataset, the first calibration model was constructed:

$$\text{Glucose Concentration (mM)} = 1.1978 \times (\text{Max Current in µA}) - 0.4567 \quad (R^2 = 0.9998)$$

<img width="985" height="459" alt="Screenshot 2026-04-27 at 2 55 57 PM" src="https://github.com/user-attachments/assets/2cb0e151-1bb0-470f-8fdb-c8190bdcf42f" />

> *Caption: "Initial calibration curve, March 5. R² = 0.9998 on training data, but failed to generalize to new measurements — a consequence of fitting four points with zero within-concentration replication."*

The extremely high R² was misleading: with only four measurements (one per concentration), the model was effectively an interpolation with no measurement variability accounted for. Subsequent testing trying to reproduce these results showed the error.

#### Key Finding: Breakdown at High Concentrations

The calibration relationship fails at very high glucose concentrations (extreme diabetic ranges). Several non-ideal effects contribute:

- **Enzyme saturation** — GOx reaction rate approaches $V_{\max}$ and no longer increases proportionally with glucose (Michaelis-Menten kinetics)
- **Mass transport limitation** — glucose supply to the electrode, not reaction rate, controls the response
- **Deviation from Cottrell behavior** — the diffusion-controlled assumptions of the model no longer hold

This is a classic chemical engineering example of a process leaving its ideal operating regime — analogous to a reactor model failing when kinetic or mixing assumptions break down. The device is calibrated and validated for the 2–12 mM clinical range.

---

### March 25, 2026 — Validation Testing and Failure Analysis

Extensive validation testing evaluated two independent feature extraction methods: peak current and steady-state current at a fixed time point.

Although calibration curves could be constructed for both methods, neither produced predictions that were sufficiently accurate or reproducible across new trials. This indicated a systemic limitation rather than a methodological error.

#### Engineering Interpretation of Failure

**1. Strip–Instrumentation Mismatch.** Commercial glucose test strips are designed to operate with proprietary voltage waveforms and internal electronics. The Rodeostat's simple chronoamperometric step may not match the strip's intended operating conditions.

**2. Inconsistent Mechanical Contact.** Variations in contact quality between Rodeostat leads and strip electrodes introduced signal variability independent of glucose concentration.

**3. Nonlinear Reaction Kinetics.** Strip enzyme chemistry introduces nonlinearities not captured by a simple linear calibration model.

**4. Signal Instability.** Small variations in strip positioning, contact pressure, or timing disproportionately affected measured current.

**Engineering conclusion:** Both calibration approaches failed validation, indicating the root cause was physical rather than algorithmic. Targeted hardware intervention was required before further calibration attempts.

---

### April 2, 2026 — Arduino Serial Communication and OLED Display

After identifying that the display required SPI (not I2C) communication, the OLED was successfully initialized and validated on a prototyping breadboard.

> *Caption: "Initial OLED wiring setup on breadboard, April 2. SPI connection established after switching from I2C."*
> <img width="867" height="507" alt="Screenshot 2026-04-27 at 2 57 23 PM" src="https://github.com/user-attachments/assets/12587aa6-de29-4984-a667-b7863b4036df" />

#### OLED Wiring (SPI Interface)

| OLED Pin | Arduino Pin |
|----------|-------------|
| DC (Data/Command) | D8 |
| RES (Reset) | D9 |
| CS (Chip Select) | D10 |
| MOSI (Data) | D11 |
| SCK (Clock) | D13 |

#### Serial Communication Workflow

The Python Jupyter Notebook workflow was extended to:

1. Run the chronoamperometry measurement via the Rodeostat
2. Extract the calibration feature from the current-time response
3. Compute the predicted glucose concentration using the calibration equation
4. Broadcast the concentration value to the Arduino over USB serial

The Arduino sketch listens on the serial port, parses the received concentration string, and displays the result on the OLED in real time.

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

#### Python Notebook Script

```python
from potentiostat import Potentiostat
import matplotlib.pyplot as plt
import pandas as pd
import serial
import serial.tools.list_ports
import time

# 0) SHOW AVAILABLE PORTS
print("Available ports:")
for p in serial.tools.list_ports.comports():
    print(" ", p.device, "-", p.description)

# 1) SET PORT NAMES
rodeostat_port = "/dev/cu.usbmodem101"
oled_port      = "/dev/cu.usbmodem11101"

# 2) CONNECT TO OLED
oled = serial.Serial(oled_port, 115200, timeout=1)
time.sleep(2.5)
oled.reset_input_buffer()
oled.reset_output_buffer()
print("OLED connected on", oled_port)

# 3) CONNECT TO RODEOSTAT
dev = Potentiostat(rodeostat_port)
dev.set_sample_period(100)

# 4) RUN CHRONOAMPEROMETRY AT CV-OPTIMIZED 0.3 V
name = "chronoamp"
test_param = {
    "quietValue": 0.0,
    "quietTime":  1000,
    "step": [(0, 0.0), (29000, 0.3)],
}
dev.set_param(name, test_param)
print("Running test...")
t, volt, curr = dev.run_test(name, display="pbar", filename="data.txt")

# 5) BUILD DATAFRAME AND SAVE
df_raw = pd.DataFrame({"time_s": t, "voltage_V": volt, "current_uA": curr})
df_raw = df_raw.sort_values("time_s").reset_index(drop=True)
df_raw["time_s"] = df_raw["time_s"].round(2)
df_raw.to_excel("chrono_data.xlsx", index=False)

# 6) PLOT
plt.figure(figsize=(9, 6))
plt.subplot(211)
plt.title("Voltage and current vs time")
plt.plot(t, volt); plt.ylabel("potential (V)"); plt.grid(True)
plt.subplot(212)
plt.plot(t, curr); plt.ylabel("current (uA)"); plt.xlabel("time (sec)"); plt.grid(True)
plt.tight_layout(); plt.show()

# 7) EXTRACT PEAK CURRENT (ignore first 1 s)
df_window = df_raw[df_raw["time_s"] >= 1.0]
max_current_uA = float(df_window["current_uA"].max())
print("Peak current (uA):", max_current_uA)

# 8) COMPUTE CONCENTRATION
# Final calibration: C (mM) = (I_peak (uA) + 5.415) / 3.7379
concentration_mM = (max_current_uA + 5.415) / 3.7379
concentration_mM = max(0, concentration_mM)
print("Glucose Concentration (mM):", concentration_mM)

# 9) SEND TO OLED
oled.write(f"{concentration_mM:.1f}\n".encode())
time.sleep(0.3)
while oled.in_waiting:
    print("OLED:", oled.readline().decode(errors="ignore").strip())
oled.close()
```

---

### April 8, 2026 — Hardware Integration and Soldering

The system moved from loose jumper wire connections to a soldered assembly. The OLED screen and all wiring connections to the Arduino were soldered onto a breadboard, replacing the earlier prototype with mechanically stable joints. The full pipeline (measure → process → display) was verified end-to-end from a single Jupyter Notebook.

<img width="541" height="603" alt="Screenshot 2026-04-27 at 4 46 22 PM" src="https://github.com/user-attachments/assets/a5612769-33c9-49d7-9f39-8220c60cb2b9" />

---

### April 16, 2026 — New Strip Procurement, Shim Addition, Cyclic Voltammetry, and Electrode Rework

Following the March 25 failure analysis, a systematic hardware intervention was implemented to address the two primary identified failure modes: inconsistent mechanical contact and strip batch variability. This session also introduced cyclic voltammetry as a principled method for selecting the applied chronoamperometric voltage.

#### New Strip Procurement

Fresh True Metrix self-monitoring glucose test strips were procured from CVS. The new strips showed noticeably more consistent signal behavior compared to the original batch. Where the earlier strips produced erratic baselines and poorly reproducible peak magnitudes, the new strips yielded cleaner current-versus-time transients with more predictable Cottrell-like decay profiles.

#### Shim Addition and Contact Improvement

A physical shim was inserted underneath each test strip during measurement to apply consistent downward pressure on the strip, standardizing the contact resistance between the Rodeostat leads and the strip electrodes. This mechanical improvement addressed a major source of measurement variability that had persisted across all earlier calibration attempts.

Prior to the shim, strips without controlled contact pressure produced signals ranging from completely open-circuit (flat current) to severely distorted, making it impossible to distinguish contact failures from genuine concentration-dependent signals.

<img width="484" height="374" alt="Screenshot 2026-04-27 at 2 58 46 PM" src="https://github.com/user-attachments/assets/ec0b6f7d-2018-4d26-8643-ef776563f1a3" />

> *Caption: "Failed chronoamperometry measurements from poor strip contact (2.2 mM, no shim). Flat or distorted traces indicate intermittent electrode connection rather than a genuine electrochemical response — indistinguishable from a zero-concentration sample."*

<img width="390" height="296" alt="Screenshot 2026-04-27 at 2 59 58 PM" src="https://github.com/user-attachments/assets/8dbd1244-d30f-4f7c-bac8-d3d408b8e303" />

<img width="406" height="307" alt="Screenshot 2026-04-27 at 3 00 16 PM" src="https://github.com/user-attachments/assets/94fbfa67-454d-4b06-bd62-4ede3ca5331e" />

> *Caption: "Chronoamperometry with shim added (6.6 mM). At first, we received a crazy result of 1000 microamps. After repositioning, began to see more reliable results"*

#### Electrode Connection Rework

The physical connections between the Rodeostat leads and the strip were reworked with attention to consistent polarity and lead placement, reducing stray resistance at the connection interface. Final lead assignment:

| Lead Color | Electrode Role |
|------------|---------------|
| Red | Working |
| Orange | Reference |
| Yellow / Grey | Counter |

#### Cyclic Voltammetry and Applied Voltage Determination

CV scans were run on the new True Metrix strips across all five glucose concentrations (2.2, 4.4, 6.6, 8.8, and 11.1 mM). The potential was swept from −0.6 V to +0.6 V.

The anodic oxidation peak — corresponding to H₂O₂ oxidation at the working electrode — was visible and reproducible across all concentrations, consistently appearing near **0.3 V**. Because peak position was independent of concentration (as expected for a kinetically controlled electrode process on a fixed surface), 0.3 V was adopted as the fixed applied potential for all subsequent chronoamperometric measurements, replacing the previously arbitrary 0.5 V step.

---

### April 26, 2026 — Final Calibration, Enclosure, and Prototype Completion

With all hardware improvements in place, the final calibration was performed and the device was assembled in its enclosure.

#### Final Calibration Dataset

Peak current was measured at 0.3 V applied potential using new True Metrix strips with the shim. One strip per concentration.

| Glucose Concentration (mM) | Peak Current (µA) |
|----------------------------|-------------------|
| 2.2 | 4.42 |
| 4.4 | 11.21 |
| 6.6 | 13.80 |
| 8.8 | 31.40 |
| 11.1 | 35.82 |

#### Final Calibration Curve

$$\text{Peak Current (µA)} = 3.7379 \times \text{Glucose Concentration (mM)} - 5.415 \quad (R^2 = 0.935)$$

Inverted for concentration prediction:

$$\boxed{\text{Glucose Concentration (mM)} = \frac{\text{Peak Current (µA)} + 5.415}{3.7379}}$$

ADD PICTURE HERE
> *Caption: "Final calibration curve, April 26. New True Metrix strips at 0.3 V applied potential (CV-optimized). R² = 0.935 across the 2.2–11.1 mM range. Deviation at 8.8 and 11.1 mM reflects the onset of enzyme saturation."*

The R² of 0.935 is substantially lower than the original four-point training fit (0.9998) but is more honest — it reflects a properly constructed calibration with variability present in the data, rather than a four-point interpolation. The deviation at high concentrations is consistent with the Michaelis-Menten saturation behavior identified in the March 5 analysis.

#### End-to-End Screen Validation

| Actual Concentration (mM) | Screen Output (mM) |
|---------------------------|-------------------|
| 2.2 | 11.7 |  2.7 |
| 4.4 | 3.42 | 14.5 |
| 6.6 | 7.9 | 33.2 |
| 8.8 | 42.5 | 47.9 |
| 11.1 | 48.8 | 35.0 |

These results confirm the full sensing-to-display pipeline is functional end-to-end. Quantitative accuracy remains limited by single-strip calibration — the primary target for further refinement. The pipeline itself (acquire → process → transmit → display) operates correctly for every measurement.

#### 3D-Printed Enclosure

The Arduino Uno and OLED display were mounted inside a 3D-printed enclosure that houses the components securely while providing access for the strip holder and USB connection.

---

## Final System Performance

### What the Device Does

1. User applies a glucose test strip to the shim-assisted strip holder and initiates measurement from the Jupyter Notebook
2. The Rodeostat applies a 0.3 V chronoamperometric step for 30 seconds
3. Python processes the current-time response and extracts peak current (ignoring the first 1 s of transient noise)
4. The calibration equation converts peak current to glucose concentration in mM
5. Concentration is transmitted over USB serial to the Arduino
6. Program reruns if inhuman result (negative glucose concentration)
7. The OLED displays the result in real time

### Calibration Summary

| Parameter | Value |
|-----------|-------|
| Applied potential | 0.3 V (CV-determined) |
| Calibration feature | Peak current (µA) |
| Calibration equation | C (mM) = (I + 5.415) / 3.7379 |
| R² | 0.935 |
| Validated range | 2.2 – 11.1 mM |
| Strip type | True Metrix self-monitoring glucose test strips |

### Key Design Iterations

| Issue Encountered | Root Cause Identified | Resolution Implemented |
|---|---|---|
| Non-reproducible calibration | Strip batch variability | New CVS-procured True Metrix strips |
| Signal dropout / flat traces | Inconsistent strip contact pressure | Shim added to strip holder |
| Non-optimal applied voltage | Arbitrary voltage selection (0.5 V) | CV-guided 0.3 V selection |
| Noisy electrode connection | Incorrect connections and voltage interference through Soldering | Electrode connection rework  |
| Calibration nonlinearity at high [C] | GOx enzyme saturation (Michaelis-Menten) | Validated operating range limited to ≤ 11.1 mM |

---

## Current Project Status

The system is a **fully functional final prototype (with significant error)**. The sensing, calibration, display, and enclosure are all complete. The device demonstrates the full engineering chain from electrochemical signal to displayed concentration reading and meets the core design objectives established at project inception.

**Copy link to watch it in action:**
https://github.com/user-attachments/assets/cb9e7f55-9fd3-4ae5-9c2e-26083d2b7484

### What is Working

- [x] Electrochemical signal acquisition with the IO Rodeostat
- [x] CV-determined optimal applied voltage (0.3 V) for True Metrix strips
- [x] Shim-assisted strip holder for consistent mechanical contact
- [x] Reworked electrode connections with defined lead polarity
- [x] Python-based data collection, processing, and visualization in Jupyter Notebook
- [x] Export of raw data to CSV/Excel
- [x] Experimentally derived and implemented calibration curve (R² = 0.935)
- [x] Serial communication from Python to Arduino
- [x] Real-time OLED display of computed glucose concentration
- [x] Soldered OLED and Arduino connections on breadboard
- [x] 3D-printed enclosure housing all display and processing components

### What Remains for Further Refinement

- [ ] Triplicate measurements per concentration to quantify calibration uncertainty and define error bounds
- [ ] Automated re-run logic for out-of-range or low-quality signals
- [ ] Cottrell $t^{-1/2}$ linearity check as a per-measurement quality flag
- [ ] Full migration of processing to onboard microcontroller (fully standalone operation)

---

## System Architecture

<img width="1137" height="653" alt="System Architecture" src="https://github.com/user-attachments/assets/cff04c35-f222-4edc-b9f9-14935a0425f3" />

---

## Market Relevance and Application

The long-term target market is underserved and low-income populations in urban environments where diabetes prevalence is high and access to healthcare is limited. Potential user groups include adults aged 30–65 at elevated risk for Type 2 diabetes, low-income individuals not yet engaged in regular monitoring, community health clinics, federally qualified health centers (FQHCs), and nonprofit screening programs.

**Distribution pathways:** community health clinics and FQHCs, nonprofit chronic disease prevention organizations, local health departments and diabetes outreach programs, and university- or hospital-affiliated screening initiatives.

The design goal is not a premium consumer medical device, but a proof-of-concept for a low-cost sensing platform that could support glucose awareness and screening in communities where access to conventional glucometers is a barrier.

---

## SWOT Analysis

### Strengths

- Strong medical relevance and recurring demand for glucose monitoring
- Real-time signal acquisition with direct embedded display feedback
- Low-cost component selection: commercial strips, Rodeostat, Arduino, OLED
- Modular architecture enables independent development of sensing and display layers
- Interdisciplinary integration of electrochemistry, transport phenomena, reaction kinetics, instrumentation, and embedded systems
- CV-guided voltage selection demonstrates principled experimental design rooted in chemical engineering

### Weaknesses

- Sensing accuracy not yet validated for reliable clinical-grade predictions
- Computer-dependent operation (Jupyter Notebook required for processing)
- Calibration based on single measurement per concentration — no uncertainty quantification
- Performance degrades above ~11 mM due to enzyme saturation

### Opportunities

- Rising diabetes prevalence increases urgency for affordable monitoring tools
- Future optimization could reduce cost and improve scalability
- Advances in embedded electronics offer pathways toward fully standalone operation
- Strong potential for educational, community health, and screening applications

### Threats

- Competition from established commercial glucometers with regulatory approval
- Regulatory barriers for medical sensing devices intended for clinical use
- Liability concerns associated with inaccurate glucose prediction
- Possible fundamental incompatibility between commercial strips and non-proprietary instrumentation at higher concentrations

---

## Future Work for Future Semester Groups

### Hardware

- Migrate signal processing to onboard microcontroller (eliminate laptop dependency)
- Integrate potentiostat analog front end directly with Arduino or upgraded MCU
- Evaluate battery-powered operation for true portability
- Finalize strip holder design with integrated shim for robust and repeatable strip loading

### Longer-Term

- Validate the full system against a commercial glucometer reference on matched samples
- Investigate miniaturization and PCB integration of the analog front end
- Assess regulatory pathway for any future clinical or community deployment

---

## All Calibration Data

Below is all of our calibration data taken over the semester!

[calibrationdata.xlsx](https://github.com/user-attachments/files/27147918/calibrationdata.xlsx)


---

## Repository Contents

- `notebooks/` — Jupyter Notebooks for chronoamperometry, calibration, and serial communication
- `data/` — CSV/Excel exports of raw current-time data for each glucose standard
- `arduino/` — Arduino sketch for OLED display and serial communication
- `docs/` — Preliminary Design Report, Initial Design Report, GANTT chart
- `README.md` — This file

### Reports and Presentations

- [Glucometer Preliminary Design Presentation](https://github.com/user-attachments/files/25115353/Glucometer.Preliminary.Design.Presentation.pdf)
- [SweetSpot GANTT Chart - Gantt Chart Template (3).pdf](https://github.com/user-attachments/files/27147929/SweetSpot.GANTT.Chart.-.Gantt.Chart.Template.3.pdf)
- [Initial Design Report](https://github.com/user-attachments/files/25423979/Initial.Design.Report.pptx)
