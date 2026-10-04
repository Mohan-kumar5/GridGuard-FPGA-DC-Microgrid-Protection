# GridGuard: FPGA-Based DC Microgrid Protection and Fault Isolation

## Overview

GridGuard is an FPGA-based intelligent protection and fault-isolation system designed for low-voltage DC microgrids.

DC systems are increasingly used in battery storage, solar photovoltaic systems, telecommunications, electric vehicle charging, and DC distribution. Unlike AC systems, DC fault currents do not naturally pass through a current zero-crossing, allowing fault current to rise rapidly and making fast interruption more challenging.

GridGuard explores a hardware-based approach in which high-speed measurements are processed directly in FPGA logic to detect abnormal transients, distinguish genuine faults from normal system events, and isolate the affected feeder while keeping healthy branches operational.

The prototype is being developed around the **Microchip PolarFire FPGA platform** and a **24 V DC microgrid test system**.

---

## Problem

Conventional protection approaches for low-voltage DC systems face several challenges:

- DC fault currents can rise rapidly.
- There is no natural AC current zero-crossing to assist interruption.
- Mechanical protection devices may not react quickly enough for fast transients.
- A simple overcurrent threshold can incorrectly trip during legitimate load switching or startup events.
- In a multi-feeder DC microgrid, disconnecting the entire bus for a single branch fault reduces system availability.

GridGuard aims to address these challenges through fast digital signal processing and selective branch isolation.

---

## Proposed Approach

The system acquires synchronized electrical measurements from the DC microgrid and processes them using parallel FPGA hardware.

The protection pipeline includes:

1. High-speed voltage and current measurement
2. ADC interfacing
3. Digital derivative calculation
4. Transient feature extraction
5. Haar wavelet-based high-frequency analysis
6. Branch-current comparison
7. Fault discrimination
8. Fault localization
9. Hardware protection state machine
10. Selective MOSFET-based feeder isolation

The protection decision is intended to be deterministic and implemented in hardware so that the critical protection path does not depend on a software operating system or general-purpose processor.

---

## System Architecture

```text
                 24 V DC MICROGRID
                        |
        +---------------+---------------+
        |               |               |
     Feeder 1        Feeder 2        Feeder 3
        |               |               |
     Current         Current         Current
      Sensor          Sensor          Sensor
        |               |               |
        +---------------+---------------+
                        |
                  Voltage Sensor
                        |
                        v
              +-------------------+
              |   Multi-channel   |
              |       ADC         |
              +-------------------+
                        |
                        v
              +-------------------+
              |   PolarFire FPGA  |
              |                   |
              |  dI/dt Processing  |
              |  dV/dt Processing  |
              |  Haar DWT          |
              |  Branch Comparison |
              |  Fault Logic       |
              +-------------------+
                        |
                        v
              +-------------------+
              | Protection FSM     |
              +-------------------+
                        |
                        v
              +-------------------+
              | Gate Driver       |
              +-------------------+
                        |
             +----------+----------+
             |          |          |
          MOSFET 1   MOSFET 2   MOSFET 3
             |          |          |
          Feeder 1   Feeder 2   Feeder 3

Fault Detection
GridGuard does not rely solely on an instantaneous current threshold.
Multiple features are considered to improve discrimination between normal transients and genuine faults.
Current derivative
The FPGA calculates:
dI/dt

A rapid increase in current can indicate a developing short circuit.
Voltage derivative
The system also evaluates:
dV/dt

Abnormal bus-voltage behavior provides additional information about the electrical event.
Haar Wavelet Analysis
A Haar-based high-frequency analysis is used to identify abrupt changes in sampled signals.
For adjacent samples:
d1[n] = (x[n] - x[n-1]) / sqrt(2)

The resulting detail coefficients provide a compact representation of fast transient behavior.
Branch Comparison
Currents from multiple feeders can be compared to determine whether the disturbance is localized to a particular branch.
Fault Discrimination
The extracted features are combined using deterministic protection logic.
Conceptually:
             Measured Signals
                    |
          +---------+---------+
          |         |         |
         dI/dt     dV/dt    Haar Energy
          |         |         |
          +---------+---------+
                    |
            Branch Comparison
                    |
                    v
          +--------------------+
          | Fault Discrimination|
          +--------------------+
                    |
          +---------+---------+
          |                   |
       Normal              Fault
          |                   |
       Continue        Identify Branch
                              |
                              v
                       Isolate Branch

The objective is to avoid unnecessary tripping during legitimate load transients while responding rapidly to abnormal fault events.
Selective Fault Isolation
When a fault is identified, the protection controller determines the affected feeder and commands the corresponding gate driver.
Only the affected branch is disconnected.
Fault on Feeder 2

Feeder 1  --->  REMAINS CONNECTED
Feeder 2  --->  DISCONNECTED
Feeder 3  --->  REMAINS CONNECTED

This selective approach aims to maintain power availability to healthy branches.
FPGA Platform
The project is being developed using a Microchip PolarFire FPGA platform.
The FPGA is used for:
- Parallel signal processing
- Deterministic protection logic
- Low-latency feature extraction
- Hardware state-machine control
- Real-time fault discrimination
- Selective feeder control
The project specifically investigates where FPGA hardware acceleration provides an advantage over a purely software-based protection implementation.
Development Status
The project is currently under development.
Current work includes:
- System architecture development
- Protection algorithm design
- FPGA signal-processing architecture
- ADC and sensor interface planning
- Fault-detection logic development
- Hardware protection architecture
- Simulation and verification planning
Hardware validation and quantitative performance measurements will be added as the prototype progresses.
Planned Validation
The prototype will be evaluated using controlled DC microgrid events such as:
- Normal load switching
- Load startup transients
- Sudden load changes
- Branch short-circuit events
- Different fault locations
- Multiple operating conditions
The planned measurements include:
- Fault detection latency
- Fault localization accuracy
- False-trip behavior
- FPGA resource utilization
- Maximum achievable processing rate
- Protection response time
Why FPGA?
The central motivation for using an FPGA is deterministic, parallel hardware processing.
Several protection calculations can operate simultaneously rather than sequentially:
             ADC Samples
                  |
       +----------+----------+
       |          |          |
     dI/dt      dV/dt      Haar
       |          |       Analysis
       +----------+----------+
                  |
          Parallel Decision
                  |
            Protection FSM
                  |
             Gate Driver

This architecture is intended to minimize processing latency and provide predictable protection behavior.
Project Goals
The main goals of GridGuard are:
1. Develop a high-speed FPGA-based DC fault detection architecture.
2. Distinguish genuine faults from normal electrical transients.
3. Identify the affected feeder.
4. Isolate only the faulty branch.
5. Keep healthy DC microgrid branches operational.
6. Evaluate the benefits and limitations of FPGA-based protection.
7. Develop a reproducible hardware/software prototype.
Repository Structure
The repository will contain the project documentation, FPGA RTL, simulation files, hardware interface information, and supporting material as development progresses.
GridGuard-FPGA-DC-Microgrid-Protection/
│
├── README.md
│
├── docs/
│   ├── project-overview/
│   ├── system-architecture/
│   └── diagrams/
│
├── rtl/
│   ├── adc_interface/
│   ├── signal_processing/
│   ├── haar_dwt/
│   ├── fault_detection/
│   └── protection_controller/
│
├── simulation/
│   ├── testbenches/
│   └── waveforms/
│
└── hardware/
    ├── sensing/
    ├── gate_driver/
    └── protection_switch/

Team
GridGuard is being developed as an undergraduate Electrical and Electronics Engineering project.
Project Lead / Contributor:
Mohan Kumar S
Additional team members and contributors will be documented as the repository develops.
Disclaimer
This repository documents an academic engineering prototype under development. The system is intended for research, experimentation, and educational purposes and is not currently intended to replace certified commercial protection equipment.
Future Work
Future development will focus on:
- Completing FPGA RTL implementation
- Hardware-in-the-loop testing
- ADC integration
- Real-time fault experiments
- FPGA timing and resource optimization
- Protection latency measurement
- Hardware validation on the PolarFire platform
- Improving fault classification robustness
- Publishing reproducible implementation details

### But there's one important issue

Before you paste that, I want to correct something from my previous advice: **don't add technical details to the repository merely because they sound good.** The README above deliberately labels things as *planned/development* where they aren't yet experimentally verified.

Your GitHub currently has **1 contributor and 0 stars/forks**, which is completely normal for a brand-new repository. Don't try to artificially inflate those numbers.

Once you've updated the README, **don't create fake RTL files just to make the repository look bigger**. Upload the actual GridGuard documents, diagrams, Verilog/SystemVerilog, simulations, etc. that you genuinely have.

After that, your Claude application can truthfully point to the repository as evidence of the project you're working on.
