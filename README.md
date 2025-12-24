## Real-Time Calorimeter State Logging and Monitoring

Outline of the design and deployment of **real-time monitoring and logging infrastructure** for calorimeter detector state.

Each calorimeter sector contains six interface boards used to communicate with the detector for both configuration and monitoring.  
Each interface board is connected to:

- A slow-control communication board  
- A Multi-channel Power Output Device (MPOD)  
- A low-voltage power supply  
- An Analog-to-Digital Converter (ADC) board  

The slow-control communication board is responsible for setting and monitoring the detector state, including:

- SiPM temperatures  
- Low-voltage and bias-voltage set points  
- Tower-by-tower bias voltage offsets  
- Tower-by-tower SiPM leakage current  
- Pre-amplifier gain mode  

These boards are also used for commissioning tasks such as enabling LEDs and injecting test pulses for calibration runs.

---

## Calorimeter Control Architecture

![IHCal/OHCal Half-Sector Communication Design](figures/hcal_control_diagram.png)

* Calorimeter sector communication design showing interface board connections to slow control, MPOD, low-voltage power, and ADC boards. Diagrams reproduced from the sPHENIX Technical Design Report.*

---

## Detector State Logging Infrastructure

Detector state information is critical for both online detector operation and offline reconstruction and calibration.  
For the EMCal and HCal, detector state information is logged in real time by:

1. Communicating with slow-control boards via telnet
2. Parsing detector state responses  
3. Writing parsed values to a PostgreSQL database via psycopg2  

Detector state has been logged at intervals of up to **every 10 minutes** for more than **three years of data taking**.

### Database Schema

![HCal Database Schema](figures/hcal_database_schema.png)

*Database schema used for logging HCal detector state, including slow-control data, bias voltage information, and LED/pedestal analysis results.*

Detector information is keyed by **offline tower ID** and **timestamp**, enabling direct access to tower-by-tower state during reconstruction and calibration.  
This structure supports temperature-dependent gain corrections and long-term stability monitoring.

**Implementation in sPHENIX core software:**

- Full integration into sPHENIX core software for use during reconstruction and calibration using odbc++ package  
  implemented in [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

---

## Real-Time Monitoring Dashboards

Grafana dashboards provide both real-time and historical views of the detector state. 
Tower-level information is aggregated to present clear sector and half-sector summaries suitable for shift operations.

![Shifter Monitoring Dashboard](figures/shift_monitoring.png)

*Shifter-facing dashboard showing aggregated HCal gain, SiPM temperature, bias voltage offsets, LED, and pin-diode status.*

![Bias Voltage Monitoring Dashboard](figures/expert_bias_monitoring.png)

*Expert dashboard displaying sector-level bias voltage monitoring for the HCal.*

---

### Summary

This monitoring, logging, and reconstruction framework enables:

- Robust real-time detector health monitoring  
- Long-term use in data production, reconstruction and calibration pipelines
