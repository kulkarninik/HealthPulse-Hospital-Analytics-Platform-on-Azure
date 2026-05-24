# HealthPulse-Hospital-Analytics-Platform-on-Azure
End-to-end data engineering project built on Azure cloud. Ingests hospital operational data from source systems, transforms it through a Medallion Architecture pipeline using Azure Databricks and PySpark, and serves business-ready insights through an interactive Power BI dashboard.
# Architecture
Hospital Source Systems (APIs + Flat Files)
        ↓
Azure Data Lake Gen2 (Bronze Layer — Raw Delta)
        ↓
Azure Databricks + PySpark (Silver Layer — Cleaned)
        ↓
Azure Databricks + PySpark (Gold Layer — Aggregated)
        ↓
Power BI Dashboard (Clinical + Financial + Risk Analytics)

Orchestrated end-to-end by Azure Data Factory on a daily schedule.
# Tech Stack
Layer        Technology
Cloud     PlatformMicrosoft Azure
Data Lake StorageAzure     Data Lake Storage Gen2
Transformation & ProcessingAzure     Databricks, PySpark, Spark SQL
OrchestrationAzure     Data Factory (ADF)
VisualizationPower     BI Desktop + Power BI Service
Language    Python, DAX
# Project Structure
healthpulse-azure-hospital-analytics/
│
├── notebooks/
│   ├── 00_generate_data.py          # Data generation (dev/testing)
│   ├── 01_bronze_ingestion.py       # Raw ingestion to Bronze Delta
│   ├── 02_silver_transform.py       # Cleansing, validation, enrichment
│   ├── 03_gold_aggregations.py      # Business-level aggregations
│   ├── 04_register_tables.py        # Register Delta tables
│   └── 05_export_for_powerbi.py     # Export Gold to CSV for Power BI
│
├── adf_pipeline/
│   └── pl_hospital_etl_daily.json   # ADF pipeline definition
│
├── powerbi/
│   └── HealthPulse_Dashboard.pbix   # Power BI report file
│
├── data_model/
│   └── schema.md                    # Table schemas and column definitions
│
└── README.md

# Data Sources

SourceFormatIngestion MethodPatient AdmissionsREST APIDatabricks API call (incremental)ICU VitalsREST APIHourly pollingBilling RecordsCSV Flat FileADF SFTP Copy ActivityLab ResultsJSON APIDatabricks API call (incremental)

Medallion Architecture
Bronze Layer

Raw data landed from source systems without transformation
Converted to Delta format
Audit columns added: ingestion_timestamp, source_file, pipeline_run_id

Silver Layer

Type casting and date formatting
Deduplication on primary keys
Derived columns: length_of_stay, age_group, is_critical
Data quality flagging: VALID / NEGATIVE_LOS / MISSING_DOCTOR

Gold Layer

dept_kpis — Department-level clinical KPIs
monthly_revenue — Monthly financial aggregations by payment status
risk_profile — Patient-level risk tier classification


Data Model
dept_kpis
ColumnDescriptiondepartmentHospital department nametotal_patientsTotal admitted patientsavg_length_of_stayAverage days per admissionavg_severityAverage severity score (1-10)readmissionsCount of readmitted patientsreadmission_rateReadmission %avg_patient_ageAverage patient age
monthly_revenue
ColumnDescriptionyear_monthBilling monthpayment_statusPaid / Pending / Disputed / Waivedtotal_billedTotal billed amounttotal_out_of_pocketPatient out-of-pocket amountclaim_countNumber of claims
risk_profile
ColumnDescriptionpatient_idUnique patient identifierage_groupAge bucket (18-29 / 30-49 / 50-64 / 65+)departmentTreating departmentdiagnosisPrimary diagnosisrisk_tierHigh Risk / Medium Risk / Low Riskinsurance_typeGovernment / Private / Corporate / Self-paytotal_amountTotal billed amount

Power BI Dashboard Pages
Page 1 — Executive Overview

Total Patients, Avg Length of Stay, Total Revenue, Readmission Rate KPI cards
Patients by Department (bar chart)
Insurance Distribution (donut chart)
Monthly Revenue Trend (line chart)

Page 2 — Clinical Insights

Readmission Rate by Department
Severity Score by Department
Risk Tier Breakdown (stacked bar)
Department Performance Summary (table)
Department slicer for cross-filtering

Page 3 — Financial Dashboard

Monthly Revenue Trend
Revenue by Payment Status
Billed vs Out-of-Pocket comparison
Payment Status slicer


ADF Pipeline
Pipeline name: pl_hospital_etl_daily
[Bronze_Ingestion] ──► [Silver_Transform] ──► [Gold_Aggregations]

Schedule: Daily at 2:00 AM UTC
Retry: 3 attempts per activity
Linked service: Azure Databricks (Access Token auth)


Azure Resources
ResourceNameResource Grouprg-hospital-analytics-devStorage AccountsahospitalanalyticsdlDatabricks Workspaceadb-hospital-analytics-devData Factoryadf-hospital-analytics-dev

Key DAX Measures
daxTotal Patients = COUNT(risk_profile[patient_id])

Overall Readmission Rate % = 
DIVIDE(SUM(dept_kpis[readmissions]), SUM(dept_kpis[total_patients]), 0) * 100

Collection Rate % = 
DIVIDE(
    SUMX(FILTER(monthly_revenue, monthly_revenue[payment_status] = "Paid"), monthly_revenue[total_billed]),
    SUM(monthly_revenue[total_billed]), 0
) * 100

High Risk % = 
DIVIDE(
    CALCULATE(COUNTROWS(risk_profile), risk_profile[risk_tier] = "High Risk"),
    COUNTROWS(risk_profile), 0
) * 100

# How to Run

Clone this repository
Create Azure resources as listed above
Upload notebooks to Databricks workspace
Configure storage credentials in each notebook
Create ADF pipeline using the JSON definition in adf_pipeline/
Run pl_hospital_etl_daily pipeline manually for first load
Open HealthPulse_Dashboard.pbix in Power BI Desktop
Update data source connection to your Databricks cluster
Refresh data and publish to Power BI Service


# Domain
Healthcare / Hospital Operations Analytics
Applicable to hospital chains, health insurance providers, health-tech platforms and government health departments managing multi-department patient and financial data at scale.
