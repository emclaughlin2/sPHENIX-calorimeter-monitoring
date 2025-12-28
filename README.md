# Real-Time Calorimeter State Logging and Monitoring

## Related sPHENIX Core Software Pull Requests

- **Detector state ETL pipeline, database schema, and reconstruction integration**  
  [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

## Project Overview

This project demonstrates the design and deployment of a **high-volume, real-time data ingestion, logging, and monitoring system** for calorimeter detector state.  
The system is engineered as a **continuous ETL pipeline** that extracts slow-control telemetry from hardware, transforms it into analysis-ready formats, and loads it into a relational database for **downstream analytics, monitoring, and physics reconstruction workflows**.

---

## Problem Statement

The sPHENIX detector operates continuously (24/7) with an estimated operating cost of **~$1M per week**.  
The physics analyses targeted by sPHENIX rely on **rare event signatures**, requiring the accumulation of **very large statistics over extended data-taking periods**.

Any degradation in detector performance directly impacts:
- Statistical power of analyses  
- Scientific return on a high-cost experimental operation  

To mitigate these risks, the experiment requires a **robust, real-time monitoring and logging system** capable of:

- Continuously tracking detector state during live operations  
- Providing rapid feedback for time-critical interventions  
- Persisting high-granularity detector telemetry for long-term analysis  

In addition to real-time observability, detector state data must be stored with sufficient temporal resolution to enable **post hoc de-trending and correction of time-dependent effects** (e.g., temperature-driven gain drift).  
This allows data collected over long time spans to be **combined safely**, maximizing usable statistics without introducing systematic bias.

The monitoring and ETL infrastructure described here addresses these requirements by ensuring **data quality, operational visibility, and long-term analytical integrity** across multi-year data-taking.

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
For the EMCal and HCal, detector state is processed through a real-time fault-tolerant ETL pipeline:

1. **Extract**  
   - Poll slow-control boards via persistent telnet connections  
2. **Transform**  
   - Parse raw telemetry  
   - Normalize units and formats  
   - Map hardware channels to offline tower identifiers  
3. **Load**  
   - Batch process structured records to a PostgreSQL database using `psycopg2`
   - Maintain integrity of composite keys (offline tower ID + timestamp)
   - Maintain fault-tolerant ETL with local persistence and automatic backfill to prevent data loss during database outages 

The pipeline operates at a configurable cadence (**every 1-10 minutes**) and has continuously ingested data for **over three years**, enabling robust longitudinal analysis and offline high statistics temporal data de-trending.

---

## Database Schema and Data Modeling

![HCal Database Schema](figures/hcal_database_schema.png)

*Relational schema for HCal detector state logging, including slow-control telemetry, bias voltage configuration, and derived quantities from LED and pedestal analyses.*

Key design features:

- **Composite primary keys** using **offline tower ID** and **timestamp** for ease in integration with offline data production pipeline
- Time-series–friendly schema optimized for near-time monitoring
- Clear separation between raw telemetry and derived calibration products
- Static tables of tower-sector and nominal tower value information for easy aggregation and time-series analysis in monitoring dashboards 

This data model enables:

-  Easy aggregation and near real-time monitoring for immediate alerts of trend changes
- Efficient tower-level queries during reconstruction  
- Time-dependent corrections (e.g., temperature-driven gain drift) for cross-run and cross-time period trend analysis and combination

---

### Core Software Integration

The database-backed detector state is fully integrated into the sPHENIX reconstruction and calibration stack via the `odbc++` interface.

- PostgreSQL → reconstruction workflow  
  implemented in [sPHENIX coresoftware PR #2552](https://github.com/sPHENIX-Collaboration/coresoftware/pull/2552)

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

---

## Tools & Technologies

**Data Engineering & ETL**
- Design and implementation of **fault-tolerant ETL pipelines** for high-volume detector telemetry Python-based ingestion using **`psycopg2`**  
- **Python** for parsing, normalization, and validation of semi-structured hardware telemetry
- **Cron jobs** for scheduled ingestion, retries, and health checks
- Local persistence and replay via **SQL snapshot files** to enable recovery from database outages  

**Databases & Storage**
- **PostgreSQL** for durable, queryable detector state storage  
- Schema design using composite keys and static tables for quick aggregation and analysis

**Dashboard Monitoring & Software Integration**
- **Grafana** dashboards for real-time and historical monitoring  
  - Metric aggregation and trend analysis across detector subsystems  
  - Data quality checks and anomaly detection using derived metrics
- Integration with large-scale **C++ data reconstruction framework** via the **`odbc++`** library

---

## Impact & Takeaways

- Enabled **real-time operational decision-making** during continuous 24/7 detector operation 
- Preserved **data integrity and completeness** across multi-year data-taking through fault-tolerant ingestion  
- Supported **long-term analytical workflows**, through integration in full production software for calibration, reconstruction, and de-trending of time-dependent effects  
- Reduced risk of data loss during infrastructure outages via **local persistence and automated backfill**  
- Demonstrated how **modern data engineering principles** can be applied to high-stakes, high-cost scientific systems  
