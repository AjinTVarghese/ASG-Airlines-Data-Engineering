# ASG Airlines – End-to-End Data Engineering Case Study

## Project Overview

This project implements an end-to-end data engineering pipeline for ASG Airlines to transform raw operational airline data into reliable datasets for business intelligence and reporting.

The raw operational data from multiple systems (booking platforms, scheduling systems, and airport logs) contained intentional quality issues: missing values, duplicate records, inconsistent date/time formats, flight identifier irregularities, and overnight (cross-day) flights.

Python and Pandas were used for data ingestion, profiling, cleaning, validation, and transformation. The resulting cleaned datasets were loaded into Power BI via Power Query, modelled into a star schema, and visualized through an interactive dashboard tracking operational performance, traffic, and data quality.

---

## Architecture & Data Flow

```text
Raw Excel Workbook (Scheduling, Bookings, Payments, Passengers)
       │
       ▼
Data Ingestion & Profiling (Python / Pandas)
       │
       ▼
Data Cleaning & Transformation
  ├── Identifier Validation & Standardization
  ├── Date/Time Parsing & Overnight Flight Handling (Cross-day check)
  ├── Flight Duration Recalculation & Discrepancy Checks
  ├── PII Protection (Email/Phone Masking & Aadhaar Hashing)
  └── Anomaly & Data-Quality Flagging
       │
       ▼
Clean CSV Export (`flights_cleaned.csv`, `bookings_cleaned.csv`, `payments_cleaned.csv`, `passengers_cleaned.csv`)
       │
       ▼
Power BI (Power Query & Star Schema Modelling)
       │
       ▼
Interactive Power BI Dashboard & Executive Reporting

```

---

## Tech Stack & Tools

* **Data Engineering & Processing:** Python, Pandas, NumPy, Jupyter Notebook
* **Storage & Staging:** Local CSV / Staged Files (Designed for easy migration to Azure Data Lake / ADF)
* **Data Modeling & Visualization:** Microsoft Power Query, Power BI Desktop, DAX
* **Version Control:** Git, GitHub

---

## Key Pipeline & Transformation Steps

1. **Ingestion & Inspection:** Loaded multi-sheet Excel workbooks (`flights`, `bookings`, `payments`, `passengers`), checking shape, schema, and missing values.
2. **Text & ID Standardization:** Trimmed whitespace, converted city codes (`source`, `destination`) to uppercase, and cleaned corrupted `flight_id` values.
3. **Date/Time Parsing & Duration Calculation:** Converted string/time objects into robust datetime formats. Recalculated true flight duration in minutes using:

$$\text{Duration} = \text{Arrival Time} - \text{Departure Time}$$

4. **Overnight Flight Handling:** Automatically detected cross-day flights where arrival time precedes departure time (adding 1 calendar day) to prevent negative or corrupted durations.
5. **Data Quality & Anomaly Checks:** Flagged duration discrepancies (`duration_difference.abs() > 0.01`), missing timestamps, and structural irregularities.
6. **PII Protection & Governance:** Masked emails (`****@domain.com`) and phone numbers, hashed sensitive identifiers (`aadhaar_hash`), and excluded unprotected raw personal data from analytical exports.

---

## Relational Data Model (Star Schema)

The tables are structured in a relational star schema inside Power BI:

* **`flights_clean`** (Dimension Table) $\rightarrow$ Connected to `bookings` via `flight_id` (1 : Many)
* **`passengers_clean`** (Dimension Table) $\rightarrow$ Connected to `bookings` via `passenger_id` (1 : Many)
* **`bookings_clean`** (Fact Table) $\rightarrow$ Connected to `payments` via `booking_id` (1 : Many)

---

## Power BI Dashboard & KPIs

The interactive Power BI report features dedicated sections:

* **Executive Overview:** Total Flights, Total Bookings, Total Passengers, Total Revenue, and Average Flight Duration.
* **Duration Analysis:** Actual vs. original duration comparisons, overnight flight trends, and duration distributions.
* **Route Performance:** High-traffic routes, source-to-destination volume, and route-wise delays/anomalies.
* **Airline Trends:** Fleet distribution, booking activity, and performance broken down by airline.
* **Data Quality & Anomaly Insights:** Visibility into flagged data discrepancies and operational anomalies.

---

## 💡 Key Findings

Based on the Power BI dashboard:

### 1. Airline Distribution

* IndiGo has the highest number of flights with 272 flights, followed by Air India with 255, SpiceJet with 247, and Vistara with 230.

### 2. Booking Distribution

* IndiGo also has the highest booking count with 167 bookings.
* Air India and SpiceJet each have 158 bookings, while Vistara has 153.

### 3. Flight Duration

* The overall average flight duration is 163.11 minutes.
* Air India has the highest average duration at approximately 165.39 minutes.
* SpiceJet has the lowest at approximately 157.96 minutes.

### 4. Overnight Operations

* There are 122 overnight flights.
* Air India has the highest number with 33, followed by IndiGo with 32.

### 5. Route Traffic

* HYD → MAA is the highest-volume route shown in the route analysis, followed by MAA → BLR and BOM → CCU.

### 6. Route Duration

* Average flight duration varies considerably by route, indicating that route-level analysis is important when comparing operational duration.

### 7. Interactive Filtering

* The dashboard allows operational users to drill into specific airlines, sources, destinations, and booking statuses.

---

## ⚠️ Delay / Anomaly Assumption

The original problem statement mentions delays and anomalies. However, the supplied dataset does not contain both scheduled and actual departure/arrival timestamps. Therefore, traditional flight delay cannot be calculated reliably.

This project uses data-quality anomalies and operational indicators instead, including:

* Abnormal durations
* Missing timestamps
* Invalid/suspicious identifiers
* Overnight flights

If scheduled and actual timestamps become available, a proper delay calculation can be added.

---

## 📌 Assumptions

The following assumptions were made:

1. `flight_id` is used as the flight identifier.
2. Flight duration is calculated from departure and arrival timestamps.
3. A flight is classified as overnight when its arrival date is later than its departure date.
4. Missing timestamps are not artificially generated.
5. Passenger PII is not required for operational reporting.
6. Airline information is filled only where a reliable flight-prefix mapping is available.
7. Traditional delay cannot be calculated without scheduled and actual timestamps.
8. The cleaned datasets are intended for analytical reporting.

---

## Repository Structure

```text
ASG-Airlines-Data-Engineering/
│
├── README.md
├── notebooks/
│   └── ASG_Airlines_Data_Pipeline.ipynb
├── data/
│   ├── flights_cleaned.csv
│   ├── bookings_cleaned.csv
│   ├── payments_cleaned.csv
│   └── passengers_cleaned.csv
├── powerbi/
│   └── ASG_Airlines_Dashboard.pbix
├── architecture/
│   ├── architecture_diagram.png
│   ├── data_flow_diagram.png
│   └── data_model.png
└── documentation/
    └── ASG_Airlines_Project_Documentation.docx

```
