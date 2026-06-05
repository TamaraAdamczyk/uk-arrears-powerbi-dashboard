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

## 3. Data Model
The dataset is structured using a star‑schema‑inspired model, designed to support clear, efficient analysis of arrears, payments, contacts, and demographic information.
All tables connect through the anonymised TenancyReference, ensuring clean filter propagation and consistent reporting.
<img width="972" height="631" alt="image" src="https://github.com/user-attachments/assets/7d7fe459-b493-4e71-8cbf-9b83bb2a92fc" />

Figure 1. Star schema.

The data model is built around a trusted tenancy pipeline designed to ensure that only tenancies with consistent, non‑conflicting data are included in the final analysis.
This protects the dashboard from issues in the original system export, such as duplicated tenancy references or mismatched demographic/financial records.

The model consists of:
- Raw source tables (imported from Excel)
- Integrity check tables (row‑count and uniqueness validation)
- Trusted tenancy dimensions (filtered, clean tenancy lists)
- Final reporting dimensions and facts

### 3.1 Source Tables (Raw Imports)
These tables come directly from the Excel files and represent the operational data:
- FactArrearsActions
- FactBalances
- FactContacts
- CustomerDemographics
- PaymentType
- PropertyHeader
- TenancyHeader

They are used as the foundation for all downstream transformations.

### 3.2 Integrity Checks (Data Quality Layer)
To prevent inaccurate reporting, each tenancy is assessed across three dimensions:
- Demographics → must have exactly one demographic record
- Payer Type → must have exactly one payer record
- Property Mapping → must link to exactly one property
- Three integrity tables are generated:
- Integrity_DemoCounts - counts demographic rows per tenancy
- Integrity_PayerCounts - counts payer rows per tenancy
- Integrity_PropertyCounts - counts distinct property references per tenancy

These are merged into a single table:

**TenancyIntegrity**
This table assigns the following flags:
- DemoOK
- PayerOK
- PropertyOK
- IsTrusted_Profile (DemoOK + PayerOK)
- IsTrusted_Property (DemoOK + PayerOK + PropertyOK)

Reason (diagnostic explanation for failures)
This layer ensures that only tenancies with clean, reliable, non‑contradictory data progress into the model.

### 3.3 Trusted Tenancy Dimensions
Two filtered tenancy lists are created:

**DimTenancy_Trusted_Profile**
Tenancies with valid demographic + payer data.

**DimTenancy_Trusted_Property**
Tenancies with valid demographic + payer + property data.

These dimensions act as gatekeepers for the rest of the model, ensuring that only trustworthy tenancy references are used in downstream joins.

### 3.4 Cleaned Reporting Dimensions
Using the trusted tenancy flags, two additional dimensions are created:

**DimDemographics**
Demographic attributes for tenancies where DemoOK = 1.

**DimPaymentType**
Payment type attributes for tenancies where PayerOK = 1.

Both dimensions remove duplicates and exclude any tenancy failing integrity checks.

### 3.5 Fact Tables
The fact tables (arrears, balances, contacts, actions) remain structurally close to the raw imports but are filtered through the trusted tenancy dimensions to ensure:

- No double‑counting
- No mismatched demographic/financial profiles
- No corrupted tenancy references

This produces a clean, analysis‑ready star schema.

**FactBalances2**

This table contains period‑level financial balances for each tenancy. It is the core fact table used for arrears trend analysis.

It includes the following fields:
- TenancyReference - anonymised tenancy ID
- SubaccountID - synthetic sub‑account identifier
- PeriodEnd - reporting period end date
- PeriodBalance - arrears/credit balance for that period (synthetic)
- Heartland - region grouping (A, B, C)
- FinancialPeriod - numeric period key used for date joins
- PeriodBalance (bins) - categorised balance ranges for visualisation

Purpose
- Tracks arrears over time
- Supports regional comparisons
- Enables trend analysis
- Acts as the foundation for the DimTenancy table
- Provides the main numeric measure used across the dashboard

Origin
This table was built from the original balances spreadsheet, which contained multiple rows per tenancy (one per reporting period). Because of this, it was also used to derive the DimTenancy table by extracting a unique list of TenancyReference values.

**FactArrearsActions1**
Arrears‑related actions taken on each tenancy. Supports analysis of intervention activity and arrears management behaviour.

**FactContacts3**
Records contact attempts or interactions associated with tenancies. Used for operational insight into tenant engagement.

### 3.6 Dimension Tables
The model includes several curated dimension tables built from the raw tenancy, demographic, payment, and property data.
These dimensions are filtered using the trusted‑tenancy logic to ensure that only valid, consistent, non‑conflicting tenancy records appear in the analytical model.

#### 3.6.1 DimTenancy (Base Tenancy Dimension)
Purpose:  
Provides a unique list of TenancyReference values derived from the balances dataset.

Key steps:
- Extract tenancy references from FactBalances
- Remove duplicates
- Drop all non‑tenancy columns

This table forms the starting point for all tenancy‑level joins.

#### 3.6.2 DimTenancy_Trusted_Profile
Purpose:  
A filtered tenancy list containing only tenancies with clean demographic and payer data.

Logic:

Includes only tenancies where:
- DemoOK = 1
- PayerOK = 1

Removes all diagnostic columns from the integrity process

This dimension is used when demographic and payment‑type consistency is required.

#### 3.6.3 DimTenancy_Trusted_Property
Purpose:  
A stricter tenancy list containing only tenancies with clean demographic, payer, and property mappings.

Logic:

Includes only tenancies where:
- DemoOK = 1
- PayerOK = 1
- DistinctPropertyCount = 1

Removes all diagnostic columns

This dimension is used when property‑level reporting requires guaranteed one‑to‑one tenancy‑property relationships.

#### 3.6.4 DimDemographics
Purpose:  
Provides demographic attributes for tenancies with valid demographic profiles.

Logic:
- Remove empty tenancy references
- Deduplicate
- Join to TenancyIntegrity
- Keep only tenancies where DemoOK = 1
- Remove diagnostic columns

This ensures demographic reporting is based only on reliable, non‑duplicated demographic records.

#### 3.6.5 DimPaymentType
Purpose:  
Provides payer type information for tenancies with valid payment profiles.

Logic:
- Remove empty tenancy references
- Join to TenancyIntegrity
- Keep only tenancies where PayerOK = 1
- Remove diagnostic columns
- Deduplicate

This ensures payment‑type reporting is based only on consistent payer data.

#### 3.6.6 DimPropertyHeader
Purpose:  
Contains property‑level attributes (e.g., property reference, address, region).
Used for linking tenancies to their associated properties.

Notes: This table is taken directly from the raw PropertyHeader file. It is typically joined using DimTenancy_Trusted_Property to ensure one‑to‑one tenancy‑property relationships

## 4. Dashboard Features
The Power BI dashboard is organised into six interactive pages, each designed to explore arrears performance from a different analytical angle.

### 4.1 Main Figures (Latest Regional Metrics)
Provides a high‑level snapshot of the most recent reporting period.

- Total Arrears
- Tenancies in Arrears
- Tenancies Clear
- Total Tenancies

Visuals include:
- Total arrears by region
- Median arrears by region
- Tenancies in arrears by region
- Latest reporting period table

This page gives an immediate sense of scale, performance, and data freshness across regions.

4.2 Distribution (Arrears Distribution by Region)
Shows how arrears are distributed within the selected region.

- Arrears histogram
- Average arrears
- Tenancies in arrears
- % with arrears > £500

This page highlights skew, concentration of low‑balance vs high‑balance cases, and overall arrears pressure.

4.3 Trends (Monthly Arrears Trends)
Shows arrears performance over time.

- Monthly total arrears
- Monthly median arrears
- Regional breakdowns

Users can compare long‑term patterns, seasonality, and region‑specific behaviour.

4.4 Age & Tenancy (Demographic Arrears Patterns)
Explores arrears by demographic characteristics.

- Median arrears by age band
- Tenancies in arrears by age band
- Median arrears by tenancy length
- Tenancies in arrears by tenancy length

This page helps identify which customer groups experience higher arrears.

4.5 Payment (Arrears by Payment Type)
Breaks down arrears by payment method.

- Total arrears by payment type
- Median arrears by payment type
- Tenancies in arrears by payment type
- Payment type distribution

Useful for understanding risk profiles across UC, HB, DD payers, etc.

4.6 Demographic Data Quality
Shows the completeness and reliability of demographic data.

- Total vs excluded tenancies
- % usable tenancies by region
- Reasons for exclusion

Availability of age and tenancy length

This page validates the integrity‑filtered model and highlights where demographic data is strong or weak.

## 5. Key Insights
The dashboard highlights several consistent patterns across regions, demographics, payment types, and reporting periods. These insights summarise the most important findings from the latest available data.

### 5.1 Regional Performance (Latest Available Dates)
Totals are based on each region’s most recent reporting period, not a single shared date.

Region A shows the highest arrears levels overall, with the largest number of tenancies in arrears.

Region B sits in the middle, with strong consistency across metrics and a higher median arrears than Region A.

Region C has the lowest total arrears and tenancy volumes but still shows meaningful arrears concentration.

This staggered reporting structure ensures each region’s figures reflect its freshest available data.

<img width="1152" height="648" alt="image" src="https://github.com/user-attachments/assets/070c8175-b328-4213-a455-2f9ba93e43e1" />

Figure 2. Latest regional metrics.

### 5.2 Arrears Distribution (Latest by Region)
Each region’s distribution page shows a similar pattern:

A heavy concentration of low‑balance arrears, tapering sharply as balances increase.

Region A has the highest average arrears and the largest volume of cases.

Region B shows the highest proportion of tenants with arrears > £500, indicating deeper arrears among fewer households.

Region C has the lowest average arrears and the smallest share of high‑balance cases.

Across all regions, arrears are skewed toward lower balances, but the severity profile differs.

<img width="1153" height="641" alt="image" src="https://github.com/user-attachments/assets/3dd5222f-73f6-4eb1-bb6c-086d865474c0" />

Figure 3. Region A distribution - histogram.

<img width="1152" height="640" alt="image" src="https://github.com/user-attachments/assets/672b34ac-d387-496f-94e4-2a4ec786e132" />

Figure 4. Region B distribution - histogram.

<img width="1157" height="637" alt="image" src="https://github.com/user-attachments/assets/c80fb839-d7f2-4081-a579-d04d44d0361d" />

Figure 5. Region C distribution - histogram.

### 5.3 Trends Over Time (All Regions)
The combined and regional trend lines show:

- Total arrears fluctuate seasonally, with noticeable peaks around mid‑year.
- Region A consistently drives the highest arrears totals, shaping the combined trend.
- Average arrears spiked sharply in mid‑2024, then stabilised across all regions.
- Regions B and C show more stable month‑to‑month behaviour, with fewer extreme movements.

Overall, arrears pressures have stabilised after earlier volatility.

<img width="1167" height="647" alt="image" src="https://github.com/user-attachments/assets/b15b5611-cbdd-4c07-9f25-39b99608abf8" />

Figure 6. Trends over time. Line charts.

### 5.4 Demographic Patterns (Age Band & Tenancy Length)
Across all regions:
- Younger tenants (16-24) show higher median arrears and lower tenancy volumes.
- Middle age bands (35-54) hold the largest number of tenancies in arrears.
- Longer tenancy lengths (10+ years) tend to have lower median arrears, suggesting stability over time.
- Shorter tenancies (1-2 years) show higher arrears, indicating early‑stage vulnerability.

Demographics reveal clear behavioural differences in arrears risk.

<img width="1146" height="647" alt="image" src="https://github.com/user-attachments/assets/786b1b73-05d9-4545-849e-b80fd942ffdd" />

Figure 7. Demographic data. Clustered column charts.

### 5.5 Payment Type Behaviour
Payment type is one of the strongest predictors of arrears:

- UC (Universal Credit) accounts for the majority of tenancies in arrears and the largest share of total arrears.
- FullPayerActiveDD shows the lowest arrears levels, both median and total.
- FullPayerNoDD and PartialHB groups show higher median arrears, indicating increased risk when payments are not automated.
- The distribution of payment types is heavily skewed toward UC, shaping overall arrears behaviour.

Payment method strongly correlates with arrears severity.

<img width="1176" height="646" alt="image" src="https://github.com/user-attachments/assets/67c3fcb4-6e57-42e7-b2b7-88038cf9f227" />

Figure 8. Payer type data. Clustered column charts (total and median arrears) + donut chart (payment type) + clustered bar chart (number of tenancies in arrears).

### 5.6 Data Quality
Demographic data quality is strong and consistent:

- ~87–88% of tenancies are usable across all regions.
- The only exclusion reason is “Demographics not unique,” affecting around 12% of records.
- Age band availability is ~99.5%, and tenancy length availability is 100%, ensuring reliable demographic analysis.

Data quality supports confident interpretation of demographic insights.

<img width="1177" height="647" alt="image" src="https://github.com/user-attachments/assets/e6adfe09-c319-4333-bfd7-d0fe326f17e5" />

Figure 9. Demographic data quality.













