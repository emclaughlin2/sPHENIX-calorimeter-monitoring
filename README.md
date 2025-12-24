# Real-Time Calorimeter State Logging and Monitoring

## Related sPHENIX Core Software Pull Requests

- **Detector state ETL pipeline, database schema, and reconstruction integration**  
  [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

---

## Overview

This document outlines the design and deployment of a **real-time data ingestion, logging, and monitoring system** for calorimeter detector state.  
The system is engineered as a **continuous ETL pipeline** that extracts slow-control telemetry from hardware, transforms it into analysis-ready formats, and loads it into a relational database for **downstream analytics, monitoring, and reconstruction workflows**.

The infrastructure supports both **low-latency operational monitoring** and **long-term historical analysis** across multi-year data-taking periods.

---

## Data Sources and Acquisition Layer

Each calorimeter sector contains six interface boards used to communicate with detector hardware for configuration and monitoring.  
Each interface board connects to:

- Slow-control communication board (primary telemetry source)  
- Multi-channel Power Output Device (MPOD)  
- Low-voltage power supply  
- Analog-to-Digital Converter (ADC) board  

The slow-control boards act as the **primary data source**, exposing structured and semi-structured telemetry including:

- SiPM temperature time series  
- Low-voltage and bias-voltage set points  
- Tower-by-tower bias voltage offsets  
- Tower-by-tower SiPM leakage current  
- Pre-amplifier gain mode  

These same interfaces are also used for **control-plane operations** (LED pulsing, test pulses) that generate additional metadata consumed by calibration and validation workflows.

---

## Calorimeter Control Architecture

![IHCal/OHCal Half-Sector Communication Design](figures/hcal_control_diagram.png)

*Calorimeter sector communication architecture illustrating data flow from hardware interfaces through slow-control systems, power distribution, and digitization layers. Diagram reproduced from the sPHENIX Technical Design Report.*

---

## Detector State ETL Pipeline

Detector state data is critical for both **online observability** and **offline analytics**.  
For the EMCal and HCal, detector state is processed through a real-time ETL pipeline:

1. **Extract**  
   - Poll slow-control boards via persistent telnet connections  
2. **Transform**  
   - Parse raw telemetry  
   - Normalize units and formats  
   - Map hardware channels to offline tower identifiers  
3. **Load**  
   - Persist structured records to a PostgreSQL database using `psycopg2`  

The pipeline operates at a configurable cadence (up to **every 10 minutes**) and has continuously ingested data for **over three years**, enabling robust longitudinal analysis.

---

## Database Schema and Data Modeling

![HCal Database Schema](figures/hcal_database_schema.png)

*Relational schema for HCal detector state logging, including slow-control telemetry, bias voltage configuration, and derived quantities from LED and pedestal analyses.*

Key design features:

- **Composite primary keys** using **offline tower ID** and **timestamp**  
- Time-series–friendly schema optimized for aggregation and joins  
- Clear separation between raw telemetry and derived calibration products  

This data model enables:

- Efficient tower-level queries during reconstruction  
- Time-dependent corrections (e.g., temperature-driven gain drift)  
- Cross-run and cross-period trend analysis  

### Core Software Integration

The database-backed detector state is fully integrated into the sPHENIX reconstruction and calibration stack via the `odbc++` interface.

- End-to-end ETL → PostgreSQL → reconstruction workflow  
  implemented in [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

---

## Analytics and Monitoring Layer

Detector state data is surfaced through **Grafana dashboards** that support both operational monitoring and exploratory analysis.  
Raw tower-level data is aggregated into sector- and half-sector–level metrics suitable for real-time decision-making.

![Shifter Monitoring Dashboard](figures/shift_monitoring.png)

*Operational dashboard showing aggregated HCal gain, SiPM temperature, bias voltage offsets, LED, and pin-diode health metrics.*

![Bias Voltage Monitoring Dashboard](figures/expert_bias_monitoring.png)

*Expert-facing dashboard providing detailed bias voltage time series and sector-level diagnostics.*

These dashboards enable:

- Real-time anomaly detection  
- Rapid feedback during data taking  
- Historical performance trending and validation  

---

## Summary

This system provides a production-grade **detector telemetry ETL and analytics pipeline** that enables:

- Reliable real-time ingestion of hardware telemetry  
- Scalable relational storage for multi-year time series  
- Direct integration with physics reconstruction and calibration workflows  
- Actionable monitoring and data-driven detector operations  

The design emphasizes **data integrity, traceability, and analytical accessibility**, aligning detector operations with modern data engineering and analytics best practices.
