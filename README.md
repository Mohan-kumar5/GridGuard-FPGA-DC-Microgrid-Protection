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
