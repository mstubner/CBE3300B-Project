# CBE3300B-Project

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


[SweetSpot GANTT Chart - Gantt Chart Template.pdf](https://github.com/user-attachments/files/25115357/SweetSpot.GANTT.Chart.-.Gantt.Chart.Template.pdf)

**Initial Design Report**

Updates (as of 19 February 2026):

Familiarized team with IO Rodeostat and created Jupyter notebook that directly captures readings from IO Rodeostat. Computed glucose and NaCl requirements for different blood ranges. Once the connectors arrive, we will begin taking measurements and create our calibration curve. Once this curve is created, we will edit the Jupyter notebook to directly calculate glucose concentrations from the Rodeostat output. 

[Initial Design Report.pptx](https://github.com/user-attachments/files/25423979/Initial.Design.Report.pptx)


Initial Prototype

Improvements

Final Deliverable
