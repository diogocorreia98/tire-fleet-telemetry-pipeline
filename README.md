# Tire Fleet Telemetry Pipeline - Hypothetical Business Use Case

A comprehensive **data engineering case study** demonstrating end-to-end telemetry pipeline design: from raw sensor ingestion through quality validation, business analytics, and warehouse modeling.

**Dataset**: 16.8M+ tire mileage events across 23 parquet chunks (September 2025 focus: 3.1M records)  
**Result**: 49.2K clean tire-day aggregates | 444K quality violations detected | 1,133 mapped vehicles | 4,240 tires

---

## Project Overview

This repository contains a **complete case study notebook** addressing three critical data engineering domains:

### **Q1: Data Ingestion & Quality Validation**
- Loaded September 2025 telemetry by compiling the dataset from `sent_files/parquet_chunks/`
- Implemented quality checks for:
  - exact duplicate records
  - cumulative mileage monotonicity
  - physically impossible interval jumps using a 130 km/h speed cap
  - mapping coverage via the tire-to-vehicle reference parquet
- Results:
  - **Duplicates removed**: 577
  - **Negative mileage increments**: 235,643
  - **Impossible interval jumps**: 207,732
  - **Final clean dataset**: 49,200 validated tire-day records
  - **Unmapped tires**: 199

### **Q2: Business Analytics & Fleet Insights**
- Analyzed fleet utilization patterns for September using the cleaned daily dataset
- **Top 10 Tires by September Distance**:
  - #1: `tire_2baebf83` with **34.6K km**
  - Top-10 average: **27.4K km**
  - Average records per top tire: **16.6**

- **Top 10 Vehicles by September Distance**:
  - #1: `vehicle_3503c7a7` with **213.1K km**
  - Top-10 average: **131.2K km**
  - Average tires per top vehicle: **10.1**

- **Inactivity Detection**:
  - **243 vehicles** with 7+ consecutive inactive days
  - **21.4%** of mapped September vehicles
  - Maximum inactivity: **27 days**

### **Q3: Data Warehouse Architecture**
- Designed a **Star Schema (Dimensional Model)** supporting historical analysis, reprocessing, and auditability
- **Core Tables**:
  - `FACT_DAILY_MILEAGE`
  - `DIM_TIRE`
  - `DIM_VEHICLE`
  - `DIM_TIRE_VEHICLE_RELATIONSHIP`
  - `DIM_DATE`
  - `PIPELINE_RUNS`
  - `DATA_QUALITY_METRICS`
  - `LINEAGE_LOG`

- **Key Features**:
  - ✓ Partitioned by `date_key`
  - ✓ Idempotent via `pipeline_run_id`
  - ✓ SCD Type 2 for mounting history
  - ✓ Explicit tracking for duplicates, negative increments, speed-cap violations, and unmapped tires

---

## Key Findings

| Metric | Finding |
|--------|---------|
| **Initial Data** | 3,118,463 September 2025 records |
| **Quality Issues** | 443,952 violations (14.2%) |
| **Clean Data** | 49,200 validated tire-day records |
| **Coverage** | 4,240 tires across 1,133 mapped vehicles |
| **Mapping Gap** | 199 tires without a vehicle mapping |
| **Inactive Vehicles** | 243 vehicles (21.4%) with 7+ day gaps |
| **Top Tire** | `tire_2baebf83` (34.6K km in September) |
| **Top Vehicle** | `vehicle_3503c7a7` (213.1K km in September) |

---

## Technology Stack

- **Language**: Python 3.11+
- **Data Processing**: Pandas
- **Notebook**: Jupyter
- **Storage Format**: Parquet

---

## Project Structure

```text
tire-fleet-telemetry-pipeline/
├── Case Study Notebook.ipynb          # Main analysis (executable)
├── README.md                          # This file
├── sent_files/
│   ├── CaseStudy_DataEngineering.pdf  # Original problem statement
│   ├── mapping_vehicle_tire.parquet   # Tire-vehicle relationships
│   └── parquet_chunks/                # Chunked telemetry source used by the notebook
│       ├── chunk_0000.parquet
│       ├── chunk_0001.parquet
│       └── ...
└── pdf_content.txt                    # Extracted problem description
```

The notebook reads telemetry from `sent_files/parquet_chunks/` so the analysis does **not** depend on a monolithic `data_for_mileage_ingestion.parquet` file being present in the repository.

---

## How to Use

### Prerequisites

```bash
pip install pandas pyarrow jupyter
```

### Run the Notebook

```bash
jupyter notebook "Case Study Notebook.ipynb"
```

### Notebook Layout

- **Cells 1-6**: Q1 - ingestion, mapping cross-reference, and quality validation
- **Cells 8-9**: Q2 - business analytics and inactivity detection
- **Cells 11-13**: Q3 - warehouse design and final wrap-up

---

## Key Insights

1. **No more `N/A` vehicle counts**: the notebook cross-references the mapping parquet to recover vehicle coverage directly.
2. **Physical plausibility belongs in QA**: monotonicity alone misses impossible jumps that exceed what a tire could travel between timestamps.
3. **Windowed distance must stay windowed**: the first September reading should not be treated as if the tire started at zero.
4. **Aggregation still drives most row reduction**: validated raw readings collapse into a compact daily fact grain.
5. **Mapping gaps should be first-class metrics**: unmapped tires need to be visible in both analysis and warehouse monitoring.

---

## Production Recommendations

1. **Operationalize mapping coverage checks** so unmapped tires are surfaced daily instead of leaking into analytics as missing vehicle context.
2. **Deploy the speed-cap validation** alongside monotonicity checks to catch physically impossible intervals early.
3. **Store rejected raw rows in a quarantine table** for replay, root-cause analysis, and sensor debugging.
4. **Monitor inactive vehicles proactively** because more than 21% of mapped vehicles showed at least one 7-day gap in September.
5. **Publish the daily quality metrics** so downstream users can track duplicates, impossible jumps, and mapping coverage over time.

---

## Scalability Considerations

- **Data Volume**: 16.8M+ raw events across 4 months, compacted into 49.2K validated daily rows for September
- **Warehouse Retention**: 5-year rolling window with aggregate fact storage
- **Query Performance**: optimized for fast daily and fleet-level rollups
- **Concurrent Users**: suitable for BI dashboards, analytics consumers, and operational monitoring
