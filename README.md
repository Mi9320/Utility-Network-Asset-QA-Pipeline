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

A fully automated GIS-based quality assurance pipeline for Toronto's Ward 13 
water distribution network. This project simulates the daily data integrity 
work performed by Geospatial Technicians at utility companies — validating 
spatial and attribute data across thousands of infrastructure assets using 
industry-standard tools: FME, ArcGIS Pro, ArcPy, and ArcGIS Online.

Real open data from the City of Toronto was processed through a 5-check 
automated QA/QC workflow, identifying 701 data quality issues across 3,696 
water infrastructure features.

---

## QA Results Summary

| Metric | Value |
|---|---|
| Total Assets Audited | 3,696 |
| Total Errors Found | 701 |
| Overall Network Health | 81% |
| Overall Error Rate | 19% |
| Valve Error Rate | 30.2% |
| Mains Error Rate | 1.6% |

### Error Breakdown

| Error Type | Count | Feature Class |
|---|---|---|
| INVALID_NETWORK_TYPE | 625 | Water_Valves_W13 |
| INVALID_VALVE_TYPE | 52 | Water_Valves_W13 |
| NULL_ATTRIBUTE | 21 | Water_Mains_W13 |
| INVALID_MATERIAL | 3 | Water_Mains_W13 |
| **Total** | **701** | |

---

## Data Sources

- **City of Toronto Open Data Portal**
  - Water Distribution Mains
  - Watermain Valves
  - City Wards Boundaries
- **Study Area:** Ward 13 — Toronto Centre
- **Coordinate System:** NAD83(CSRS) MTM Zone 10 (EPSG: 2952)
- **Features:** 1,457 water mains + 2,239 valves = 3,696 total

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

### Water Mains Checks
| Check | Transformer | Field | Result |
|---|---|---|---|
| Null/empty attributes | Tester | Waterma2, Waterma4, Waterma5, Waterma7 | 21 errors |
| Duplicate Asset ID | DuplicateFilter | Waterma2 | 0 errors |
| Invalid diameter | Tester | Waterma4 = 0 | 0 errors |
| Invalid material | Tester | Waterma5 = UNK | 3 errors |
| Invalid install year | Tester | Waterma7 < 1800 or > 2026 | 0 errors |

### Water Valves Checks
| Check | Transformer | Field | Result |
|---|---|---|---|
| Null/empty attributes | Tester | Asset_I2, Water_V3, Water_V5 | 0 errors |
| Duplicate Asset ID | DuplicateFilter | Asset_I2 | 0 errors |
| Invalid diameter | Tester | Water_V3 = 0 | 0 errors |
| Invalid valve type | Tester | Water_V5 not in approved list | 52 errors |
| Invalid network type | Tester | Water_V4 != DIST | 625 errors |

### Output Layers
- `Mains_QA_Errors` — 24 flagged water mains
- `Valves_QA_Errors` — 677 flagged valves
- `Mains_QA_Passed` — 1,433 clean water mains
- `Valves_QA_Passed` — 1,562 clean valves

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
UTILITY NETWORK ASSET QA/QC PIPELINE
QA Summary Report — Ward 13 Toronto Centre
Generated: 2026-04-12 18:45
ERROR COUNT BY TYPE:
INVALID_MATERIAL                    3
INVALID_NETWORK_TYPE                625
INVALID_VALVE_TYPE                  52
NULL_ATTRIBUTE                      21
QA STATUS SUMMARY:
Water_Mains_W13    PASS: 1433   FAIL: 24
Water_Valves_W13   PASS: 1562   FAIL: 677
TOTAL FEATURES:  3696
TOTAL ERRORS:    701
ERROR RATE:      19.0%
```
---

## Step 4 — ArcGIS Online Dashboard

**[Live Dashboard Link](https://www.arcgis.com/apps/dashboards/0619306fb6ec40c691bb2becd51f8ad5)**

Dashboard components:
- Interactive map — Ward 13 water network with error locations highlighted
- Overall Network Health KPI — 81%
- Error Rate KPI — 19%
- Valve Errors KPI — 677
- Mains Errors KPI — 24
- Clean Valves KPI — 1,562
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
├── Valves_QA_Errors          ← 677 flagged valves
├── Mains_QA_Passed           ← 1,433 clean mains
├── Valves_QA_Passed          ← 1,562 clean valves
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
