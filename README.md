# Project #4: End-to-End Geospatial Pipeline & Mineral Resource Analysis (Australia)

## Executive Summary
This project delivers an automated data engineering pipeline and executive visualization dashboard analyzing **Australia's Identified Mineral Resources (AIMR)** (Geoscience Australia 2024). By programmatically consolidating 29 individual commodity sheets into a single clean master dataset, this project highlights spatial concentrations of producing mines versus undeveloped deposits across Australian jurisdictions.

---

## Workflow & AI-Assisted Engineering
To build this project efficiently, artificial intelligence tools were leveraged as an active development partner alongside Python (Google Colab) and Power BI. Rather than writing repetitive boilerplate code from scratch, AI was utilized to:
* **Accelerate Script Architecture:** Rapidly prototype the dynamic header-detection loop to handle shifting row alignments across 29 messy commodity worksheets.
* **Debug ETL Anomalies:** Instantly diagnose coordinate shifting and string mismatches during the multi-tab consolidation phase.
* **Refine Documentation:** Structure clean portfolio narratives and professional Markdown templates to showcase end-to-end data engineering competency.

Using AI as a force multiplier allowed for rapid iteration, robust error handling, and a polished final deliverable ready for public sector and consultancy portfolios.

---

## 1. Data Engineering & ETL Pipeline (Python / Google Colab)
The raw public sector data contained structural complexities, including multi-row headers and mixed metadata across 29 commodity-specific worksheets. 

* **Ingestion & Dynamic Parsing:** Developed a robust Python script using `pandas` to iterate through the workbook, dynamically locate header rows containing key identifiers (`Entity ID`, `State`, `Latitude`, `Longitude`), and filter out non-deposit summary tabs.
* **Data Cleaning & Standardization:** Cast geospatial coordinates to numeric types, dropped records with missing spatial data, and filtered out non-standard text strings or coordinate noise in the jurisdiction column.
* **Output:** Generated a clean, consolidated master CSV containing over 4,000 standardized records ready for business intelligence consumption.

```python
import pandas as pd

file_path = "149861_01_0.xlsx"
xls = pd.ExcelFile(file_path)

skip_tabs = ["Contents", "Ore Reserves at Operating Mines", "Distribution of EDR"]
commodity_tabs = [s for s in xls.sheet_names if s not in skip_tabs]

all_deposits = []

for sheet in commodity_tabs:
    df_raw = pd.read_excel(file_path, sheet_name=sheet, header=None)
    
    header_row_idx = None
    for idx, row in df_raw.iterrows():
        row_str_values = [str(val) for val in row.values]
        if "State" in row_str_values or "Entity ID" in row_str_values:
            header_row_idx = idx
            break
            
    if header_row_idx is not None:
        df = pd.read_excel(file_path, sheet_name=sheet, skiprows=header_row_idx)
        required_cols = ["Entity ID", "State", "Longitude", "Latitude", "Current Status", "Commodity"]
        if all(col in df.columns for col in ["State", "Longitude", "Latitude"]):
            available_cols = [c for c in required_cols if c in df.columns]
            df = df[available_cols].copy()
            df["Latitude"] = pd.to_numeric(df["Latitude"], errors="coerce")
            df["Longitude"] = pd.to_numeric(df["Longitude"], errors="coerce")
            df = df.dropna(subset=["State", "Latitude", "Longitude"])
            df["Commodity_Group"] = sheet
            all_deposits.append(df)

master_df = pd.concat(all_deposits, ignore_index=True)
master_df.to_csv("gold_australian_mineral_deposits.csv", index=False)
print("Cleaned total deposits:", len(master_df))
```
### Power BI
Following the extraction and transformation phase in Google Colab, the refined dataset was seamlessly integrated into Power BI Desktop to construct an interactive executive dashboard. This phase focused on translating raw categorical variables into actionable visual intelligence regarding Australia's resource distribution.

The resulting visualization illustrates the breakdown of mineral entities by jurisdiction and operational classification:
<img width="892" height="451" alt="image" src="https://github.com/user-attachments/assets/29de0ad5-6e00-40db-91d8-e0f54c9af7fa" />


### Dashboard Insights & Analysis
The Power BI dashboard groups mineral assets by state and operational status (Deposit vs. Producing Mine), revealing critical strategic takeaways:

- **Jurisdictional Concentration: Western Australia (WA) and Queensland (QLD) dominate both resource volume and active operations by a wide margin, driven heavily by bulk commodities and critical mineral exploration.

- **Pipeline Conversion Ratio: Across most states, the volume of classified Deposits significantly outweighs Producing Mines, highlighting substantial future project pipelines and exploration activity.

- **Data Integrity: Stringent Power Query and Python filtering successfully eliminated coordinate-string anomalies, ensuring clean categorical groupings by state.

References
Geoscience Australia 2024, Australia's identified mineral resources 2024, Australian Government, Canberra, viewed 26 November 2026, .
https://data.gov.au/data/dataset/australias-identified-mineral-resources-2024/resource/9725688b-0c86-4ffc-bc8f-cbc7b2d1fbc3
