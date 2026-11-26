# Australians-identified-mineral-resources
with combonation of colab google as well idk 

Code to create table 

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
        
        # Standardize expected columns if present
        required_cols = ["Entity ID", "State", "Longitude", "Latitude", "Current Status", "Commodity"]
        if all(col in df.columns for col in ["State", "Longitude", "Latitude"]):
            # Keep only standard columns to prevent shifting errors
            available_cols = [c for c in required_cols if c in df.columns]
            df = df[available_cols].copy()
            
            # Clean coordinates
            df["Latitude"] = pd.to_numeric(df["Latitude"], errors="coerce")
            df["Longitude"] = pd.to_numeric(df["Longitude"], errors="coerce")
            
            # Drop rows with missing core geospatial data
            df = df.dropna(subset=["State", "Latitude", "Longitude"])
            df["Commodity_Group"] = sheet
            all_deposits.append(df)

# Combine and save clean master CSV
master_df = pd.concat(all_deposits, ignore_index=True)
master_df.to_csv("gold_australian_mineral_deposits.csv", index=False)

print("Cleaned total deposits:", len(master_df))
master_df.head()

This is the img idk 

<img width="892" height="451" alt="image" src="https://github.com/user-attachments/assets/29de0ad5-6e00-40db-91d8-e0f54c9af7fa" />



source :
https://data.gov.au/data/dataset/australias-identified-mineral-resources-2024 
