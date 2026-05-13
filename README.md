# Utility Network Asset QA/QC Pipeline
### Toronto Ward 13 Water Distribution Network — Data Quality Audit

![ArcGIS](https://img.shields.io/badge/ArcGIS_Pro-0079C1?style=flat&logo=esri&logoColor=white)
![FME](https://img.shields.io/badge/FME-FF6D00?style=flat&logoColor=white)
![Python](https://img.shields.io/badge/ArcPy-3776AB?style=flat&logo=python&logoColor=white)
![AGOL](https://img.shields.io/badge/ArcGIS_Online-0079C1?style=flat&logo=esri&logoColor=white)

## 🔴 Live Dashboard
**[View Toronto Ward 13 Utility QA Dashboard](https://www.arcgis.com/apps/dashboards/0619306fb6ec40c691bb2becd51f8ad5)**

---

## Project Overview
A fully automated GIS-based quality assurance pipeline for Toronto's Ward 13 water distribution network. This project simulates the daily data integrity work performed by Geospatial Technicians at utility companies — validating spatial and attribute data across thousands of infrastructure assets using industry-standard tools: FME, ArcGIS Pro, ArcPy, and ArcGIS Online.

Real open data from the City of Toronto was processed through a 5-check automated QA/QC workflow, identifying 77 data quality issues across 3,696 water infrastructure features.

---

## QA Results Summary

| Metric | Value |
|---|---|
| Total Assets Audited | 3,696 |
| Total Errors Found | 77 |
| Overall Network Health | 97.9 |
| Overall Error Rate | 2.1% |
| Valve Errors | 53 |
| Mains Errors | 24 |

### Error Breakdown

| Error Type | Count | Feature Class |
|---|---|---|
| INVALID_DIAMETER(ZERO)_TYPE | 39 | Water_Valves_W13 |
| INVALID_VALVE_TYPE(UNOKNOWN / ZERO) | 8 | Water_Valves_W13 |
| BOTH_ERRORS(DIAMETER / VALVE_TYPE) | 6 | Water_Valves_W13 |
| INVALID_MATERIAL | 7 | Water_Mains_W13 |
| INVALID_YEAR | 17 | Water_Mains_W13 |
| **Total** | **77** | |

---

## Data Sources

- **City of Toronto Open Data Portal**
  - Water Distribution Mains
  - Watermain Valves
  - City Wards Boundaries
- **Study Area:** Ward 13 — Toronto Centre
- **Coordinate System:** NAD83(CSRS) MTM Zone 10 (EPSG: 2952)
- **Features:** 1,457 water mains + 2,186 valves = 3,696 total

---

## Workflow
``` text
Toronto Open Data      FME Workspace           ArcPy Script            ArcGIS Dashboard 
(Water Mains +     →   (5 automated QA/QC  →   (PASS/FAIL stamping  →  (Interactive Dashboard
Water Valves)          checks per dataset)      + summary report)      with error map)

```
---

## Tools Used

| Tool | Purpose |
|---|---|
| ArcGIS Pro | Data preparation, clipping, geodatabase management |
| FME Form | Automated QA/QC validation workspace |
| ArcPy | PASS/FAIL stamping, summary report generation |
| ArcGIS Online | Web map publishing, Dashboard |
| Python | Scripting via ArcPy |

---

## Step 1 — Data Preparation (ArcGIS Pro)

- Downloaded Toronto Open Data: Water Distribution Mains, Watermain Valves, City Wards
- Set coordinate system to EPSG 2952 (Toronto standard)
- Clipped all layers to Ward 13 boundary
- Created File Geodatabase: `Utility_QA_Project.gdb`
- Output layers: `Water_Mains_W13` (1,457 features), `Water_Valves_W13` (2,239 features)

---

## Step 2 — FME QA/QC Workspace

**File:** `FME/Utility_QA_Checks.fmw`

Five automated checks run on each dataset:

### 💧 Water Mains Validation
| Validation Rule | FME Transformer | Target Field | Logic / Condition | Flagged Assets |
| :--- | :--- | :--- | :--- | :--- |
| **Invalid Material** | `AttributeValidator` | `Material` | Value IN approved domain (e.g., CI, DI, PVC). Rejects 'UNK'. | **7** |
| **Invalid Install Year** | `AttributeValidator` | `Install_Year` | Value > 1800 AND <= Current Year | **17** |
| **Null Geometry** | `GeometryValidator` | `Shape` | Asset must contain valid polyline geometry | **0** |
| **Duplicate IDs** | `DuplicateFilter` | `Asset_ID` | `Asset_ID` must be unique across the network | **0** |
| | | | **Total Failing Mains:** | **24** |

### 🚰 Water Valves Validation
| Validation Rule | FME Transformer | Target Field | Logic / Condition | Flagged Assets |
| :--- | :--- | :--- | :--- | :--- |
| **Invalid Diameter** | `AttributeValidator` | `Diameter` | Value > 0 (Catches human-entry `0` placeholders) | **39** |
| **Invalid Valve Type** | `AttributeValidator` | `Valve_Type` | Value NOT IN ('Unknown', 'None', 'Blank') | **8** |
| **Compound Errors** | `AttributeValidator` | `Diameter` & `Valve_Type` | Asset failed both physical and domain checks | **6** |
| **Null Attributes** | `AttributeValidator` | All Critical Fields | Value is not missing/null | **0** |
| | | | **Total Failing Valves:** | **53** |

### Output Layers
- `Mains_QA_Errors` — 24 flagged water mains
- `Valves_QA_Errors` — 53 flagged valves
- `Mains_QA_Passed` — 1,433 clean water mains
- `Valves_QA_Passed` — 2,186 clean valves

---

## Step 3 — ArcPy Script

**File:** `Scripts/QA_Report.py`

The ArcPy script performs three tasks:

1. Reads error layers and counts errors by type and feature class
2. Adds `QA_Status` field to original layers — stamps every feature PASS or FAIL
3. Generates a formatted summary report saved to `Outputs/QA_Summary_Report.txt`

### How To Run

Open ArcGIS Pro Python window and run:

```python
exec(open(r"path\to\Scripts\QA_Report.py").read())
```
### Sample Output
```
ERROR COUNT BY TYPE:
Invalid Diameter (Zero)             39
Invalid Install Year                17
Invalid Valve Type                  8
Invalid Material                    7
Compound Errors (Diameter & Type)   6

QA STATUS SUMMARY:
Water_Mains_W13    PASS: 1433   FAIL: 24
Water_Valves_W13   PASS: 2186   FAIL: 53

--------------------------------------------------
TOTAL FEATURES:  3696
TOTAL ERRORS:    77
ERROR RATE:      2.1%
==================================================
```
---

## Step 4 — ArcGIS Online Dashboard

**[Live Dashboard Link](https://www.arcgis.com/apps/dashboards/0619306fb6ec40c691bb2becd51f8ad5)**

Dashboard components:
- Interactive map — Ward 13 water network with error locations highlighted
- Overall Network Health KPI — 97.9%
- Error Rate KPI — 2.1%
- Valve Errors KPI — 53
- Mains Errors KPI — 24
- Clean Valves KPI — 2,186
- Clean Mains KPI — 1,433
- Bar charts — errors by type for mains and valves
- Legend panel

---

## Project Structure
```text
Utility_Network_Asset_QA_and_QC_Pipeline/
├── DATA/                          ← Raw Toronto open data shapefiles
├── FME/
│   └── Utility_QA_Checks.fmw     ← FME workspace (5 QA checks)
├── Outputs/
│   └── QA_Summary_Report.txt     ← Generated QA report
├── Scripts/
│   └── QA_Report.py              ← ArcPy script
└── Utility_QA_Project.gdb        ← File Geodatabase
├── Water_Mains_W13           ← Original + QA_Status field
├── Water_Valves_W13          ← Original + QA_Status field
├── Mains_QA_Errors           ← 24 flagged mains
├── Valves_QA_Errors          ← 53 flagged valves
├── Mains_QA_Passed           ← 1,433 clean mains
├── Valves_QA_Passed          ← 2,186 clean valves
└── Service_Area              ← Ward 13 boundary
```
---

## Key Skills Demonstrated

- FME Form — automated spatial and attribute validation
- ArcGIS Pro — geodatabase management, coordinate systems, geoprocessing
- ArcPy — `arcpy.da.SearchCursor`, `arcpy.da.UpdateCursor`, field management
- ArcGIS Online — feature layer publishing, web map styling, Dashboard design
- Utility sector data knowledge — water mains, valves, asset IDs, network types
- Toronto Open Data — real infrastructure data processing
- Data quality methodology — null checks, domain validation, duplicate detection

---

## Author

**Ibrahim Mirza**
- GitHub: [Mi9320](https://github.com/Mi9320)
- LinkedIn: [ibrahim-mirza3](https://linkedin.com/in/ibrahim-mirza3)
- Algonquin College — Environmental Management & Assessment | Project Management
