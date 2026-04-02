# CBE3300B-Project

Progress Log

**February 12, 2026**

We connected the IO Rodeostat potentiostat to a laptop and conducted an initial chronoamperometry test using a commercial glucose test strip and legacy connector leads. The goal of this first experiment was to verify that the Rodeostat could successfully apply a potential step to the strip and record a measurable current response over time. At this stage, the emphasis was not yet on obtaining accurate glucose concentration values, but rather on confirming that the hardware and software system could generate and capture an electrochemical signal in the expected format.

During this stage, we developed a Python script in a Jupyter Notebook to interface with the Rodeostat, collect data, and visualize the results. The script generated current versus time plots for each run, which allowed us to verify that the system was recording transient current behavior following the applied voltage step. This was an important first milestone because it established that the device could perform chronoamperometry in a way that was suitable for later calibration and signal analysis. It also created the software foundation for all subsequent experiments, including data collection, plotting, and eventual concentration prediction.

From an engineering perspective, this phase functioned as a feasibility test of the acquisition system. Before a sensor can be evaluated quantitatively, it must first be shown that the instrumentation can consistently produce a usable signal. Confirming this signal pathway was therefore a necessary prerequisite for later calibration, model development, and hardware integration.

**February 26, 2026**

We prepared a full set of standard glucose solutions for calibration and improved the data pipeline used to process experimental results. The updated Python workflow was designed not only to generate current versus time plots, but also to export raw data to CSV files for later analysis. This improvement made the system more reproducible and better suited for engineering analysis, since it allowed us to retain complete datasets rather than relying only on visual inspection of plots.

At this stage, we collected our first full datasets for high and very high glucose concentrations. These experiments were intended to establish how the electrochemical response changed as glucose concentration increased. By systematically varying concentration and recording the corresponding current traces, we began building the dataset required for a calibration model. This represented the transition from simple system verification to quantitative sensor characterization.

The electrode configuration used during testing was working electrode equals gray, reference electrode equals white, and counter electrode equals black. Maintaining a consistent electrode configuration was essential because any reversal or inconsistency in these connections would alter the measured response and compromise the validity of the calibration data.

From a chemical engineering standpoint, this stage corresponded to establishing the input-output behavior of the sensing system. In any measurement process, a calibration dataset must be built by relating a known input, in this case glucose concentration, to a measurable output, in this case current. The purpose of these experiments was therefore to determine whether the electrochemical signal could serve as a reliable state variable for concentration prediction.

**March 4, 2026**

We completed all initial concentration measurements across the prepared glucose standards. This marked an important transition from data collection to calibration analysis, because we now had a full set of experimental responses that could be compared across concentrations. By this point, the system had moved beyond simple proof of concept and into the stage of evaluating whether its output could be used quantitatively.

With the complete dataset in hand, we began comparing the shape and magnitude of the current responses for each concentration. The purpose of this analysis was to determine whether there was a consistent and physically meaningful relationship between electrochemical response and glucose concentration. If such a relationship existed, then a calibration equation could be constructed and later used for prediction of unknown samples.

This stage was especially important because it framed the project as an engineering measurement problem rather than only a coding or hardware problem. The quality of the final glucometer depends not just on whether the Rodeostat can record current, but on whether the recorded current contains sufficient and reproducible information to distinguish between concentrations.

**March 5, 2026 — Calibration Curve**

Using the initial concentration data, we constructed our first calibration model relating electrochemical response to glucose concentration. The resulting equation was:

Glucose concentration (mM) = 1.1978 × (max current in μA) − 0.4567
<img width="936" height="305" alt="Screenshot 2026-03-26 at 2 03 40 PM" src="https://github.com/user-attachments/assets/7304cc3f-2995-456e-9858-e5e4f0a6e959" />


This equation represented the first functional mapping from measured sensor output to predicted glucose concentration. In practical terms, it allowed the system to move from simply recording an electrochemical signal to operating as a quantitative sensing platform. Once the maximum current from a given run was measured, the calibration equation could be used to estimate the glucose concentration associated with that response.

The reasoning behind using current as the measured variable comes from the fundamentals of chronoamperometry. Under ideal diffusion-controlled conditions, the transient current should follow the Cottrell equation:

<img width="285" height="117" alt="image" src="https://github.com/user-attachments/assets/fdda0aaa-203e-4aba-a888-078e02fedd13" />


This relationship predicts that current is proportional to concentration, assuming the number of electrons transferred, electrode area, and diffusivity remain constant. Because of this, features derived from the current response, such as maximum current or current at a specified time, are reasonable candidates for calibration. Our initial choice of max current was based on the expectation that larger glucose concentrations should produce correspondingly larger electrochemical responses.

At this stage, the calibration equation appeared to provide a usable first approximation of concentration. However, further testing revealed that the relationship was not valid across the full concentration range.

_Key Finding: Breakdown at High Concentrations_

We observed that the calibration relationship fails at very high glucose concentrations, especially in extreme diabetic ranges. The response becomes nonlinear and unreliable, meaning that current no longer scales in a predictable way with concentration. As a result, the system cannot accurately extrapolate from the initial linear fit to very high concentrations.

This behavior is consistent with the physical limitations of commercial glucose test strips, which are generally designed for typical physiological glucose ranges rather than extreme concentrations. At high concentrations, several non-ideal effects may occur. Enzymatic reaction steps within the strip may begin to saturate, which means that increasing glucose no longer produces a proportional increase in reaction rate. In addition, mass transport can become limiting, so the supply of glucose to the electrode surface rather than the intrinsic reaction rate begins to control the response. Under these conditions, the assumptions underlying the simplest form of the Cottrell equation are no longer valid.

From a chemical engineering perspective, this is a classic example of a process leaving its ideal operating regime. In the same way that a reactor model may fail when its assumptions about mixing or kinetics no longer hold, an electrochemical calibration fails when the signal is no longer governed by a single consistent transport and reaction mechanism. The engineering implication is that the sensing system is only valid within a defined operating range, and any calibration model must respect those limits. The data therefore suggested that future work would require either restricting the operating range, selecting a different signal feature, or developing a more advanced calibration model.

Validation Testing, System Limitations, and Revised Hardware Plan

Following completion of our initial calibration model, we carried out extensive validation testing to determine whether the system could reliably predict glucose concentration from measured current responses. We rigorously tested the system using repeated trials and evaluated multiple feature extraction approaches, specifically both peak (instantaneous) current and steady-state current measured at a fixed time. The purpose of this stage was to determine whether the calibration framework could generalize beyond the initial dataset and support practical sensing. This expanded the project from simple calibration construction to true sensor validation.

Despite the fact that the Rodeostat is able to apply a voltage step and record current, our results showed that the system does not yet produce accurate or reliable glucose concentration predictions. Although calibration curves can be generated from the collected data for both methods, those relationships do not consistently hold across new measurements, and the predictions are not sufficiently reproducible. This was true for both the peak current approach and the steady-state current approach, indicating that the issue is not specific to a single signal-extraction method but rather reflects a broader limitation of the sensing system. This means that the current system demonstrates electrochemical signal acquisition, but not yet dependable glucose quantification.

**March 25 - Theoretical Basis and Deviation from Expected Behavior**


<img width="927" height="587" alt="PNG image" src="https://github.com/user-attachments/assets/6bb37bf7-06df-4d12-b4ae-94cb09e91898" />

To better understand this issue, we analyzed the system using the expected theory of chronoamperometry. Under ideal diffusion-limited conditions, the current should follow the Cottrell equation, in which current is directly proportional to analyte concentration and decreases with the inverse square root of time. Based on this model, both peak current and current measured at a fixed time should correlate with glucose concentration if the experiment is governed primarily by diffusion and if the electrochemical reaction pathway is well controlled.

Because both feature extraction methods failed, this indicates that the system is not behaving according to this idealized diffusion-controlled model. If the Cottrell relationship were valid, at least one of these approaches (early-time current or fixed-time current) would have produced a stable calibration. The fact that neither did strongly suggests that additional physical or chemical effects are dominating the system response.

_Engineering Interpretation of Failure_

Our experimental results deviate from ideal behavior, suggesting that the system is not operating under a purely diffusion-controlled regime. Several possible causes are being considered.

1. Strip–Instrumentation Mismatch
Commercial glucose test strips are designed to operate with proprietary voltage waveforms, timing windows, and internal electronics. The simple chronoamperometric step applied by the Rodeostat may not match these required operating conditions. As a result, the measured current may not directly reflect glucose concentration, even if the electrochemical measurement itself is functioning correctly.

2. Mass Transport Limitations
The Cottrell equation assumes a well-defined diffusion layer and predictable transport behavior. If glucose transport to the electrode surface is inconsistent or influenced by convection, geometry, or strip structure, then current will vary independently of concentration. This would break the expected linear relationship for both peak and steady-state current.

3. Nonlinear Reaction Kinetics / Enzyme Effects
The strip chemistry may introduce nonlinearities due to enzyme kinetics, reaction intermediates, or saturation effects. If the reaction is no longer first-order in glucose concentration, then current will not scale linearly, causing both calibration methods to fail.

4. Signal Instability and Measurement Variability
Even small variations in contact quality, strip condition, or timing can disproportionately affect measured current. Because both methods rely on consistent signal behavior (either maximum or fixed-time value), variability directly degrades predictive accuracy.

_Hardware Verification_

We also considered whether the issue might arise from the hardware itself. To investigate this, we confirmed that the Rodeostat was set up properly and physically inspected the device for any obvious faults. No clear hardware issue was identified. This supports the conclusion that the problem is not due to a fundamental device failure, but rather due to limitations in sensing compatibility and measurement conditions.

_Engineering Implication_

At this stage, the system cannot yet be considered a reliable glucometer because it does not satisfy the assumptions required for stable quantitative sensing. Although it successfully performs chronoamperometry and records current data, the measured signal cannot yet be translated into accurate concentration values with sufficient confidence.

Importantly, this conclusion is strengthened by the fact that multiple independent calibration approaches (instantaneous and steady-state) were tested and both failed validation, indicating a systemic limitation rather than a methodological error.

In engineering terms:

Acquisition layer → working
Processing layer → working
Sensing model → not validated
Revised Project Status

The system presently functions as a computer-dependent electrochemical sensing prototype. It demonstrates:

Electrochemical signal acquisition
Data collection and visualization
Calibration framework implementation

However, it does not yet achieve:

Reliable glucose concentration prediction
Arduino communication
OLED display output
Standalone device operation

Because accurate sensing is not yet achieved, hardware integration steps (Arduino + OLED) are being deferred until the sensing model is validated.

_System Architecture_

Even in its current state, the project still follows a modular sensing pipeline:

Sensor Layer
Glucose test strip produces current from electrochemical reaction

Acquisition Layer
IO Rodeostat applies voltage and records current

Processing Layer (Core System)
Python (Jupyter Notebook):

Data collection
Signal visualization
Feature extraction (peak and fixed-time current)
Calibration analysis

Output Layer (Current)
Laptop display (plots + predicted glucose values)

Output Layer (Planned)
Arduino + OLED display (future integration)

_Future Work_

Our immediate focus is to resolve the sensing limitation before proceeding with hardware integration. Planned next steps include:

Testing alternative voltage protocols (to better match strip operation)
Exploring additional feature extraction methods (e.g., integrated current, decay fitting)
Improving signal processing (smoothing, baseline correction)
Investigating strip-specific operating requirements
Evaluating whether commercial strips are fundamentally compatible with Rodeostat-based measurements

Once a reliable calibration model is established, we will proceed with:

Python → Arduino serial communication
OLED display integration
Transition to a standalone embedded system
Current Project Definition

The current prototype is best described as a:

Computer-dependent electrochemical sensing system with unresolved measurement accuracy limitations

It successfully demonstrates:

Signal acquisition
Data processing
Calibration construction

But does not yet achieve:

Reliable quantitative sensing
****Preliminary Design Report****

**Introduction**

A glucometer is a small, portable medical device used to measure the concentration of glucose in blood. It is most commonly used by individuals with diabetes to monitor blood sugar levels and inform daily management decisions. As rates of diabetes continue to rise worldwide, access to reliable and affordable glucose monitoring has become an increasingly important public health concern.
This trend is reflected locally in Philadelphia, where diabetes prevalence has increased by more than 50% over the past 15 years. Nearly one in three individuals living with diabetes in the city are unaware of their condition, highlighting a critical gap in early detection and routine monitoring. In 2017, diabetes was the sixth leading cause of death in Philadelphia, accounting for 374 deaths. These statistics highlight the need for low-cost, accessible glucometers that can expand glucose testing and support earlier intervention in at-risk populations.

**Project Goals**
Build a system that:
- Runs a chronoamperometry measurement on a commercial glucose test strip
- Extracts a robust electrochemical (current) feature (e.g., current at 10 s)
- Converts that feature to glucose concentration using an experimentally derived calibration curve
- Shows the result on a digital screen in real time

This project will be developed in two stages:
1. A computer-dependent prototype validating sensing and calibration
2. A standalone embedded prototype integrating measurement and display

**Physical and Chemical Principles of a Glucometer**
Electrochemical glucometers operate by coupling a selective biochemical reaction with an electrochemical transduction mechanism to convert glucose concentration into a measurable electrical signal. Commercial glucose test strips rely on the enzyme glucose oxidase (GOx), which selectively reacts with glucose present in the sample. In the presence of molecular oxygen, glucose oxidase catalyzes the oxidation of D-glucose to D-gluconolactone, producing hydrogen peroxide (H₂O₂) as a reaction byproduct. Because this reaction is highly specific to glucose, it provides the chemical selectivity necessary for accurate glucose sensing.
The generated hydrogen peroxide serves as the electroactive species in the sensing process. When a constant potential is applied across the electrode surface by a potentiostat, hydrogen peroxide is electrochemically oxidized at the working electrode. This oxidation reaction results in the transfer of electrons to the electrode, generating an electrical current. The magnitude of this current is directly related to the rate of hydrogen peroxide oxidation and, by extension, to the concentration of glucose originally present in the sample. In this way, chemical reaction kinetics at the electrode surface are converted into an electrical signal that can be measured with high sensitivity.

The electrochemical measurement is performed using chronoamperometry, in which the electrode potential is stepped to a fixed value and the resulting current is recorded as a function of time. Under diffusion-limited conditions, the time-dependent current response follows the Cottrell equation, which predicts that current decays proportionally to the inverse square root of time due to diffusion of electroactive species toward the electrode. Importantly, at a fixed time point after the potential step, the Cottrell equation shows that the measured current depends linearly on the concentration of the electroactive species. This relationship provides the physical justification for extracting a single current value at a specified time and using it as a robust electrochemical feature.
By measuring the steady or quasi-steady current at a fixed time for a series of known glucose standards, a calibration curve relating current to glucose concentration can be constructed. Once this calibration relationship is established, measured current values from unknown samples can be converted directly into glucose concentration using a linear fit. This combination of enzyme specificity, electrochemical oxidation, diffusion-controlled transport, and calibration-based signal interpretation forms the physical and chemical foundation of the glucometer system implemented in this project.

**Preliminary Calculations**
Blood Stimulant (NaCl Stock Solution)
To mimic the electrolyte balance of human blood, we will create a 100mL stock solution with a concentration of 140 mmol NaCl/L, which is the standard NaCl concentration in human blood.

(140 mmol/L)  (1 mol/1000 mmol)  (58.44 g/mol)   (1L/1000 mL)  (100 mL) = 0.818 g NaCl

Glucose Standards
We prepared four standards by adding specific masses of glucose to 25 mL of our stock solution. The molar mass of glucose is 180.156 g/mpl. The concentrations were calculated as follows:

**Glucose Standards**
Concentration (mM) = Mass (g) / 180.156 g/mol0.025 L   1000


<img width="492" height="244" alt="Screenshot 2026-02-05 at 7 49 14 PM" src="https://github.com/user-attachments/assets/97abdf26-e984-4bfc-b0cf-d5142b466d45" />


**Proposed Prototypes**

_**Prototype 1: Computer Based Interface**_
In this initial prototype, the laptop serves as the primary control and data-processing platform, while the Arduino is dedicated solely to user-interface output.
A Python-based workflow provided by IO Rodeo is used to execute the chronoamperometry measurement. The script controls the potentiostat, applies the required potential step, and records the resulting current–time response from the commercial glucose test strip. From this dataset, a robust electrochemical feature (such as the current at a specified time point) is extracted and converted to glucose concentration using an experimentally derived calibration curve. The calculated glucose value is then transmitted to an Arduino Uno over a USB serial connection. The Arduino receives this value and displays the glucose concentration on a 128 × 64 I2C OLED screen.

Purpose of Prototype 1

- Validate electrochemical sensing performance and calibration methodology
- Leverage a high-level computing environment to simplify data processing
- Minimize embedded system complexity during early development
- Enable rapid iteration, debugging, and performance evaluation


_**Prototype 2: Fully Standalone Embedded Measurement System**_

The second prototype advances the system toward a fully integrated, portable device by migrating all measurement, processing, and display functions to the embedded platform.
In this configuration, the Arduino (or an upgraded microcontroller, if required) directly executes the chronoamperometry routine. The microcontroller interfaces with the potentiostat’s analog front end (AFE) to acquire current data from the glucose test strip in real time. Onboard signal processing routines extract the selected electrochemical feature, which is then converted to glucose concentration using stored calibration parameters. The computed glucose value is displayed directly on the 128 × 64 I2C OLED, with no reliance on external computing resources. The system operates entirely autonomously once powered.

Purpose of Prototype 2

- Demonstrate full hardware–software system integration
- Validate embedded data acquisition and signal-processing capability
- Enable a portable, self-contained glucose measurement platform
- Establish a foundation for future miniaturization and field deployment

_**Target Market, Pricing, and Marketing Strategy**_

The primary target market for this glucometer is underserved and low-income populations in urban environments, with an initial focus on Philadelphia, where diabetes prevalence is disproportionately high in communities experiencing poverty. Diabetes rates in Philadelphia are nearly twice as high in low-income populations, making affordability and ease of use critical design priorities.

Key target groups include:

- Adults aged 30–65, when Type 2 diabetes risk is highest
- Low-income individuals and families, particularly those with limited access to routine healthcare
- Undiagnosed or pre-diabetic individuals, who may not yet be engaged in regular glucose monitoring
- Community health clinics, nonprofits, and outreach programs that provide screening and preventive care
- Secondary markets include students, uninsured individuals, and older adults who benefit from a simplified, low-cost monitoring option.

The marketing strategy focuses on accessibility, trust, and community-based distribution, rather than traditional consumer retail channels.

Primary distribution pathways:

- Community health clinics and federally qualified health centers (FQHCs)
- Nonprofit organizations focused on chronic disease prevention
- Local health departments and diabetes outreach programs
- University- or hospital-affiliated screening initiatives
  
Marketing emphasis:

- Simple, easy-to-use design requiring minimal training
- Clear digital display for immediate interpretation
- Messaging centered on early detection, prevention, and empowerment

By partnering with trusted local organizations and prioritizing usability and affordability, this glucometer is positioned not as a luxury medical device, but as a basic health tool that supports routine glucose awareness and early intervention.

**SWOT Analysis**

A strengths, weaknesses, opportunities, and threats (SWOT) analysis was conducted to evaluate the potential impact and feasibility of the proposed glucometer system. One of the primary strengths of this device is its significant medical utility. Glucose monitoring is a critical component of diabetes management, resulting in consistent and recurring demand for reliable testing technologies. Additionally, the system provides real-time feedback, allowing users to immediately assess glucose levels and make informed health decisions. Its portable design and user-friendly interface further enhance accessibility, making the device practical for daily use in a variety of settings, including home monitoring and community health screenings.

Despite these advantages, several limitations must be considered. Accuracy can vary depending on user conditions such as sample handling, environmental factors, and device calibration, which may affect measurement reliability. Furthermore, manufacturing costs can be relatively high at smaller production scales, particularly during early development phases when economies of scale have not yet been achieved. These factors may influence both device affordability and widespread adoption.

At the same time, significant opportunities exist for expansion and impact. Rising diabetes prevalence both globally and locally increases the need for accessible monitoring technologies, particularly in underserved communities. There is also potential to optimize the device for manufacturability and large-scale production, which could reduce costs and improve accessibility. Advances in embedded electronics, biosensor fabrication, and scalable production methods provide pathways for future refinement and commercialization.

However, the project also faces notable external threats. The glucometer market already includes well-established commercial devices with strong brand recognition and regulatory approval, creating substantial competition. Additionally, inaccuracies in glucose measurements carry potential health risks, introducing liability concerns that must be addressed through careful design, calibration, validation, and regulatory compliance. These factors highlight the importance of rigorous testing and quality assurance as the system moves toward potential real-world deployment.

[Glucometer Preliminary Design Presentation.pdf](https://github.com/user-attachments/files/25115353/Glucometer.Preliminary.Design.Presentation.pdf)

[SweetSpot GANTT Chart](https://github.com/user-attachments/files/26286270/SweetSpot.GANTT.Chart.-.Gantt.Chart.Template.1.pdf)



**Initial Design Report**

Updates (as of 19 February 2026):

Familiarized team with IO Rodeostat and created Jupyter notebook that directly captures readings from IO Rodeostat. Computed glucose and NaCl requirements for different blood ranges. Once the connectors arrive, we will begin taking measurements and create our calibration curve. Once this curve is created, we will edit the Jupyter notebook to directly calculate glucose concentrations from the Rodeostat output. 

[Initial Design Report.pptx](https://github.com/user-attachments/files/25423979/Initial.Design.Report.pptx)


Initial Prototype

Improvements

Final Deliverable
