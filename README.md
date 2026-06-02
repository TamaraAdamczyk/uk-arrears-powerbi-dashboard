# uk-arrears-powerbi-dashboard
Power BI dashboard analysing UK social housing arrears using fully anonymised and synthetic data.

# UK Arrears Dashboard (Synthetic Dataset)

## 1. Project Overview
This project showcases a Power BI dashboard analysing social housing arrears across UK regions, originally developed during my data analyst internship. The goal of the dashboard was to help the organisation monitor arrears trends, identify high‑risk areas, and support operational decision‑making.

Because the original dataset contained sensitive tenant‑level financial information, it could not be shared publicly. To make this project suitable for my portfolio, I rebuilt the entire dataset using a combination of:

- ID anonymisation
- Value diversification
- Region‑based synthetic arrears generation
- Trend reshaping techniques

The result is a fully anonymised, non‑reversible synthetic dataset that preserves the analytical structure and business logic of the original work, while ensuring complete data privacy.

This repository contains:

- A Power BI dashboard demonstrating arrears patterns across regions
- A synthetic dataset engineered to mimic realistic housing finance behaviour
- Documentation explaining the anonymisation and transformation process
- Screenshots and notes illustrating the final analytical outputs

The purpose of this project is to demonstrate my ability to:

- Clean, transform, and anonymise sensitive data
- Build structured data models for BI
- Apply analytical reasoning to real‑world housing finance problems
- Design clear, insightful Power BI dashboards
- Communicate data ethics and transformation decisions transparently

This is a faithful representation of the analytical work I completed during my internship - rebuilt safely, ethically, and professionally for public sharing.

## 2. Dataset Anonymisation & Transformation
This project uses a fully anonymised and synthetic dataset designed to preserve the analytical structure of the original arrears data while ensuring zero exposure of personal or sensitive information.
All identifiers were replaced, all financial values were transformed, and credit balances were re‑engineered to create a new, realistic arrears trend across regions.

The anonymisation process consisted of three main stages:
### 2.1 ID Anonymisation (Tenancy and Property IDs)
To maintain the relational structure of the dataset while removing all identifiable information, both Tenancy IDs and Property IDs were replaced with sequential synthetic codes.

**Method**
1. Extract all original IDs
2. Remove duplicates
3. Sort alphabetically
4. Generate anonymised IDs using sequential prefixes
5. Map anonymised IDs back into the dataset using XLOOKUP

**Formulas used**
Tenancy IDs
="TEN-" & TEXT(ROW(A2),"000000")

Property IDs
="PROP-" & TEXT(ROW(A2),"000000")

These synthetic IDs preserve relationships between tables (e.g., tenancy → property → arrears) while ensuring the original identifiers cannot be reconstructed.

### 2.2 Value Diversification (Initial Scramble)
Before applying any region‑based logic, the dataset underwent a light distribution shift to ensure the values no longer resembled the original dataset.

Purpose
- Introduce randomness
- Break direct numerical similarity
- Preserve realistic financial behaviour
- Maintain compatibility with Power BI measures

**Formula Used**
=IF(C2=0,
      IF(RAND()<0.1, RANDBETWEEN(5,50), 0),
   IF(AND(C2>0, C2<200, RAND()<0.05),
      0,
   IF(AND(C2>0, C2<150, RAND()<0.05),
      -RANDBETWEEN(5,50),
   C2)))

**Effect**
- Some zero balances become small arrears
- Some small arrears become clear accounts
- A small proportion become small credits
- Most values remain unchanged
- The overall distribution becomes more varied and less identifiable

### 2.3 Region‑Based Credit Rehaul (Final Transformation)
After the initial diversification, credit balances were still too similar to the original dataset.
To fully anonymise the financial patterns and create a new, realistic arrears trend, all credit balances were replaced with synthetic arrears values, scaled by region.

Purpose
- Remove all credit balances
- Introduce region‑specific behaviour
- Create a new trend for Power BI visuals
- Ensure the dataset cannot be reverse‑engineered

**Formula Used**
=IF(C2<0,
      IF(G2="A",
            RANDBETWEEN(120,300),
      IF(OR(G2="B",G2="C"),
            RANDBETWEEN(60,200),
      RANDBETWEEN(60,200))),
   C2)

**Effect**
- Region A → highest arrears (120–300)
- Regions B & C → moderate arrears (60–200)
- All credits removed
- Positive balances preserved
- A new, realistic arrears trend emerges
