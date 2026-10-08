# Speed-Limit-Aware Network Routing & Attribute Imputation (QGIS)

![QGIS](https://img.shields.io/badge/QGIS-3.34_LTR-588240?logo=qgis&logoColor=white)
![Data Imputation](https://img.shields.io/badge/Data_Quality-Maxspeed_Gap_Filling-blue)
![CRS](https://img.shields.io/badge/Projection-EPSG%3A3035%20(ETRS89)-003399)

## Project Overview
This repository details an advanced GIS network routing workflow in [Brussels](http://googleusercontent.com/map_location_reference/0) addressing data gaps in raw OpenStreetMap (OSM) road networks. 

By applying national traffic speed regulations to impute missing `maxspeed` attributes, the pipeline compares unweighted distance-based routing against real-world speed-weighted time travel models, illustrating spatial route diversion and urban transit dynamics.

## Objectives
1. **Attribute Gap Analysis:** Audit raw OSM `maxspeed` coverage and quantify missing attribute percentages across functional road tiers.
2. **Regulatory Imputation Matrix:** Implement a QGIS Field Calculator expression to populate missing speed values using Belgian legal default speed limits by `highway` category.
3. **Time Cost Impedance Modeling:** Derive dynamic segment travel times (`time_min`) from imputed speed vectors.
4. **Before/After Route Shift Evaluation:** Run Dijkstra network solvers comparing naive distance paths against speed-aware time-minimized paths.
5. **Cartographic Output:** Create a high-resolution A3 comparative map highlighting route deviations, diverted distance, and time savings.

---

## Speed Imputation Matrix (Belgium Default Framework)

In accordance with Belgian road traffic legislation (e.g., 30 km/h urban zones in Brussels-Capital Region):

| OSM `highway` Tier | Raw Tag Presence | Default Imputed Speed (`speed_final_kmh`) | Justification / Regulation |
| :--- | :---: | :---: | :--- |
| **`motorway` / `motorway_link`** | High (~85%) | **120 km/h** | Standard Belgian Highway Speed Limit |
| **`trunk` / `trunk_link`** | Medium (~60%) | **90 km/h** | Express Dual-Carriageway Default |
| **`primary` / `secondary`** | Medium (~50%) | **50 km/h** | Major Urban/Regional Arterial Road |
| **`tertiary`** | Low (~30%) | **30 km/h** | Regional Collector Street |
| **`residential` / `living_street`** | Very Low (<15%) | **30 km/h** | Brussels Regional City-wide 30 km/h Zone |
| **`service` / `unclassified`** | Sparse (<10%) | **20 km/h** | Low-Speed Service Access |

---

## Workflow Implementation

### Step 1: Expression-Based Speed Imputation
* **QGIS Manual Reference:** *Section 18.11 - Vector Calculator / Field Calculator*
* Created field `speed_final_kmh` using the following SQL expression logic:
  ```sql
  CASE
    /* 1. Respect existing valid numerical maxspeed tags */
    WHEN "maxspeed" IS NOT NULL AND to_int("maxspeed") > 0 THEN to_int("maxspeed")
    /* 2. Fallback Imputation by Road Classification */
    WHEN "highway" IN ('motorway', 'motorway_link') THEN 120
    WHEN "highway" IN ('trunk', 'trunk_link') THEN 90
    WHEN "highway" IN ('primary', 'primary_link', 'secondary', 'secondary_link') THEN 50
    WHEN "highway" IN ('tertiary', 'tertiary_link') THEN 30
    WHEN "highway" IN ('residential', 'living_street') THEN 30
    WHEN "highway" = 'service' THEN 20
    ELSE 30
  END
