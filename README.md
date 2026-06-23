# Novus: Enterprise Quishing Detection & Cyber Forensics Pipeline

Novus is an end-to-end DataSecOps engineering project designed to detect, correlate, and visualize **Quishing (QR Code Phishing)** incidents within an enterprise network. Built entirely on **Microsoft Fabric**, the platform ingests raw multi-source security logs, processes them through a Medallion Architecture using **PySpark (Apache Spark)**, and exposes real-time analytical metrics via a **Power BI SOC Dashboard** using Direct Lake mode.



## 📊 Executive SOC Dashboard

![Novus Dashboard](REEMPLAZA_CON_EL_NOMBRE_DE_TU_IMAGEN.png)
*The Power BI SOC Dashboard displays live incident metrics, human-factor vulnerability rates, malicious infrastructure distribution, and a granular forensic ledger.*

---

## 🏗️ Architecture & Data Flow

The project implements a modern **Medallion Architecture** to guarantee data integrity, schema enforcement, and high-performance analytical processing:

1. **Ingestion Layer (Bronze):** Raw multi-line corporate event streams are stored in OneLake as JSON files:
   * `stg_email_qr_logs.json`: Outbound/Inbound email metadata containing extracted QR URLs from security gateways.
   * `stg_proxy_web_logs.json`: Real-time proxy navigation logs tracking employee web traffic.
2. **Refinement Layer (Silver):** PySpark notebooks clean text noise, validate schemas, and cast raw strings into high-precision timestamps, saving the outputs as optimized **Delta Tables**.
3. **Intelligence Layer (Gold):** A behavioral correlation engine applies cyber forensics logic to match proxy events against email timestamps, isolating active exploitation phases.
4. **Presentation Layer (Power BI):** Direct Lake connectivity reads Delta Parquet files in-memory (RAM) straight from OneLake, bypassing traditional Import/DirectQuery latency.

---

## 🛡️ Cyber Forensics Correlation Logic

The core security heuristic of Novus is built into the **Gold Layer**. To distinguish day-to-day web browsing noise from an active breach, the PySpark engine calculates the exact reaction window between the delivery of the malicious email and the user scanning the QR code:

```python
# Strategic 10-Minute Threat Hunting Window
quishing_alerts = quishing_matches.withColumn(
    "time_difference_sec",
    unix_timestamp(col("click_timestamp")) - unix_timestamp(col("event_timestamp"))
).filter((col("time_difference_sec") >= 0) & (col("time_difference_sec") <= 600))