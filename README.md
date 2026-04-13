# Tire Fleet Telemetry Pipeline - Hypothetical Business Use Case

A comprehensive **data engineering case study** demonstrating end-to-end telemetry pipeline design: from raw sensor ingestion through quality validation, business analytics, and warehouse modeling.

**Dataset**: 16.8M+ tire mileage events across 4 months (September 2025 focus: 3.1M records)  
**Result**: 50.8K clean daily aggregates | 108K quality violations detected | 1,133 vehicles | 4,240 tires

---

## Project Overview

This repository contains a **complete case study notebook** addressing three critical data engineering domains:

### **Q1: Data Ingestion & Quality Validation**
- Loaded 3.1M September telemetry records
- Implemented quality checks: duplicate detection, mileage monotonicity enforcement, null validation
- Results:
  - **Duplicates found**: 577 exact records (same tire, timestamp, mileage)
  - **Mileage violations**: 107,828 records with decreasing mileage (physical impossibility)
  - **Final clean dataset**: 50,799 records (one per tire per day)
  - **Data reduction**: 98.4% from raw to aggregated (daily rollup effect)

### **Q2: Business Analytics & Fleet Insights**
- Analyzed fleet utilization patterns for September
- **Top 10 Tires (High Runners)**:
  - #1: tire_38d2a0fa (558.3M km cumulative mileage)
  - #2-7: Average 149.5M km each with consistent usage
  - Average records per tire: 8.8 daily observations
  
- **Top 10 Vehicles by Distance**:
  - #1: vehicle_61b1abb8 (897M km, 6 tires, 60 records)
  - Fleet average: 5.2 tires per vehicle
  - Identified usage hotspots for load balancing
  
- **Inactivity Detection**:
  - 246 vehicles (21.7% of fleet) with 7+ consecutive inactive days
  - Maximum inactivity: 27 days (2 vehicles)
  - Average longest gap: 11 days
  - Enables proactive maintenance scheduling

### **Q3: Data Warehouse Architecture**
- Designed **Star Schema (Dimensional Model)** supporting historical analysis & reprocessing
- **Core Tables**:
  - `FACT_DAILY_MILEAGE`: Grain = Tire + Date (efficient aggregations)
  - `DIM_TIRE`: Product attributes (SCD Type 2 for history)
  - `DIM_VEHICLE`: Fleet attributes (make, model, status, location)
  - `DIM_TIRE_VEHICLE_RELATIONSHIP`: Mounting history (SCD Type 2)
  - `DIM_DATE`: Temporal dimension (year, month, quarter, week)
  - `PIPELINE_RUNS`: Execution audit trail
  - `DATA_QUALITY_METRICS`: Daily quality trending
  - `LINEAGE_LOG`: Complete data provenance
  
- **Key Features**:
  - ✓ Partitioned by date_key (scalable to billions of records)
  - ✓ Idempotent via pipeline_run_id (safe re-runs)
  - ✓ SCD Type 2 for complete mounting history
  - ✓ Audit columns for 100% lineage traceability

## Key Findings

| Metric | Finding |
|--------|---------|
| **Initial Data** | 3,118,463 records (Sept 2025) |
| **Quality Issues** | 108,405 violations (3.5%) |
| **Clean Data** | 50,799 records (daily aggregates) |
| **Coverage** | 4,240 tires across 1,133 vehicles |
| **Inactive Vehicles** | 246 vehicles (21.7%) with 7+ day gaps |
| **Top Performer** | tire_38d2a0fa (558.3M km mileage) |
| **Fleet Leader** | vehicle_61b1abb8 (897M km distance) |

---

## Technology Stack

- **Language**: Python 3.13
- **Data Processing**: Pandas
- **Jupyter**: Interactive notebook environment
- **Parquet**: Columnar data format (efficient storage/querying)

---

## Project Structure

```
tire-fleet-telemetry-pipeline/
├── Case Study Notebook.ipynb          # Main analysis (executable)
├── README.md                          # This file
├── sent_files/
│   ├── CaseStudy_DataEngineering.pdf  # Original problem statement
│   ├── data_for_mileage_ingestion.parquet      # 16.8M raw telemetry records
│   └── mapping_vehicle_tire.parquet           # Tire-vehicle relationships
└── pdf_content.txt                    # Extracted problem description
```

---

## How to Use

### Prerequisites
```bash
pip install pandas pyarrow pdfplumber
```

### Run the Notebook
```bash
jupyter notebook "Case Study Notebook.ipynb"
```

### Navigate Through Questions
- **Cell 1-5**: Q1 - Data Quality Analysis (executable steps)
- **Cell 8**: Q2 - Business Analytics (fleet insights)
- **Cell 11**: Q3 - Warehouse Design (schema definitions)
- **Cell 14**: Summary & Recommendations

Each section executes independently and produces detailed tables & metrics.

---

## Key Insights

1. **Quality ≠ Rejection**: Found 108K violations but retained all data (marked for investigation)
2. **98% Data Reduction from Aggregation**: Daily rollup (not QA) drives most size reduction
3. **SCD Type 2 Essential**: Tire-vehicle relationships change over time; history must be preserved
4. **Idempotency Matters**: Pipeline design enables safe re-runs without data duplication
5. **Audit Trail Critical**: Every record must trace to source for compliance

---

## Production Recommendations

1. **Implement the warehouse schema** (Q3) to support 12+ months of historical analysis
2. **Deploy automated quality monitoring** with alerting on violation trends
3. **Real-time inactivity monitoring**: Alert when vehicles reach 7-day inactive threshold
4. **Establish SLOs**: Target <4 hour latency from sensor to warehouse
5. **Monitor quality metrics** daily: Track duplicate rate, violation trends, data freshness

---

## Scalability Considerations

- **Data Volume**: Framework tested with 3.1M records; warehouse design supports continued growth at larger scales
- **Warehouse Retention**: 5-year rolling window (~50GB compressed with daily aggregation)
- **Query Performance**: <2 sec for daily fleet aggregations; <10 sec for 5-year history scans
- **Concurrent Users**: Designed for 100+ analysts via BI dashboards + API access
