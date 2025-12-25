# Real-Time Calorimeter State Logging and Monitoring

## Related sPHENIX Core Software Pull Requests

- **Detector state ETL pipeline, database schema, and reconstruction integration**  
  [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

---

## Overview

This project demonstrates the design and deployment of a **high-volume, real-time data ingestion, logging, and monitoring system** for calorimeter detector state.  
The system is engineered as a **continuous ETL pipeline** that extracts slow-control telemetry from hardware, transforms it into analysis-ready formats, and loads it into a relational database for **downstream analytics, monitoring, and physics reconstruction workflows**.

Key objectives:

- Enable **real-time operational monitoring**  
- Support **long-term historical analysis** across multi-year datasets  
- Provide **analysis-ready, structured data** for physics reconstruction  

This project showcases **data engineering, ETL, and analytics skills**, including scalable data ingestion, data modeling, time-series database design, and performance monitoring.

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

**Skills Highlighted:** Data ingestion, telemetry parsing, structured and semi-structured data handling, metadata management.

---

## Calorimeter Control Architecture

![IHCal/OHCal Half-Sector Communication Design](figures/hcal_control_diagram.png)

*Calorimeter sector communication architecture illustrating data flow from hardware interfaces through slow-control systems, power distribution, and digitization layers. Diagram reproduced from the sPHENIX Technical Design Report.*

**Skills Highlighted:** Systems integration, data pipeline design, dependency mapping.

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
   - Maintain integrity of composite keys (offline tower ID + timestamp)
   - Maintain fault-tolerant ETL with local persistence and automatic backfill to prevent data loss during database outages 

The pipeline operates at a configurable cadence (up to **every 10 minutes**) and has continuously ingested data for **over three years**, enabling robust longitudinal analysis.

**Skills Highlighted:** fault-tolerant ETL design, batch processing, data parsing and normalization, time-series ingestion, relational database loading, logging and error handling, production-scale pipeline management

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

**Skills Highlighted:** Data modeling, relational schema design, time-series optimization, key management, ETL-to-DB mapping.

### Core Software Integration

The database-backed detector state is fully integrated into the sPHENIX reconstruction and calibration stack via the `odbc++` interface.

- End-to-end ETL → PostgreSQL → reconstruction workflow  
  implemented in [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

**Skills Highlighted:** API integration, pipeline-to-application interfacing, cross-system data flow.

---

## Analytics and Monitoring Layer

Detector state data is surfaced through **Grafana dashboards** that support both live operational monitoring and historical exploratory analysis.  
Raw tower-level data is aggregated into sector- and half-sector–level metrics suitable for real-time decision-making.

![Shifter Monitoring Dashboard](figures/shift_monitoring.png)

*Operational dashboard showing aggregated HCal gain, SiPM temperature, bias voltage offsets, LED, and pin-diode health metrics.*

![Bias Voltage Monitoring Dashboard](figures/expert_bias_monitoring.png)

*Expert-facing dashboard providing detailed bias voltage time series and sector-level diagnostics.*

These dashboards enable:

- Real-time anomaly detection and alerts
- Rapid feedback during data taking  
- Historical performance trending and validation

**Skills Highlighted:** Data aggregation, dashboarding, real-time monitoring, anomaly detection, metrics design, KPI visualization.

---

## Key Achievements

- Designed a **production-ready fault-tolerant ETL pipeline** for high-volume detector telemetry  
- Implemented **relational schema** optimized for time-series analysis and physics reconstruction  
- Enabled **real-time monitoring and anomaly detection** for detector health  
- Produced **analysis-ready datasets** for calibration, reconstruction, and longitudinal studies  
- Maintained **data integrity, traceability, and reproducibility** across multi-year datasets  

**Technical Skills Demonstrated:**

- ETL pipeline design and implementation  
- Large-scale time-series data engineering  
- Relational database design and optimization  
- Data cleaning, normalization, and transformation  
- Dashboard creation and metrics visualization  
- Data quality assurance and validation  
- Cross-system integration with scientific software stacks  

---

## Impact

This system aligns **detector operations with modern data engineering and analytics best practices**, enabling:

- **Operational decision-making** through real-time dashboards  
- **Long-term calibration and reconstruction support** using high-quality, structured detector state data  
- **Traceable and auditable workflows**, ensuring reliability and reproducibility  
- Transferable experience in **production-scale ETL and analytics pipelines** for industry applications
