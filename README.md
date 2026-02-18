# Termite FRN Dashboard

> **R Shiny dashboard and geospatial mapping tool for monitoring termite Farmer Research Network (FRN) trials across western Kenya** — tracking termite attack patterns, crop vigour, and plant population loss across four growth stages with interactive filtering and variety-level attack analysis.

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Features](#features)
- [Data Sources](#data-sources)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Dashboard Tabs](#dashboard-tabs)
- [Map Outputs](#map-outputs)

---

## Overview

This project supports field monitoring of a **termite research trial** across smallholder maize farms in western Kenya. Farmers plant side-by-side plots of hybrid and local maize varieties, and field officers collect data at four crop growth stages — First Weeding, Second Weeding, Tasseling, and Harvesting.

The dashboard visualizes termite attack patterns, crop vigour scores, population reduction, and termite species locations across those stages. A dedicated **Variety Attack tab** infers whether attacks affected hybrid or local maize plots and computes attack rates per variety.

A companion **static map script** produces a figure showing trial site locations across Kenya, with organization-specific county insets at sub-county resolution.

---

## Project Structure

```
Termite_FRN/
├── app.R                                    # Shiny dashboard (main application)
├── map.R                                    # Static ggplot2 + cowplot map script
│
├── Data/
│   └── Longrains_Termite_FRN_Data_2025.xlsx # Main dataset (multi-sheet)
│       ├── Farmer_Profile                   # Farmer demographics
│       ├── Trial_setup                      # Plot setup, varieties, geolocation
│       ├── First_Weeding                    # Stage 1 observations
│       ├── Second_Weeding                   # Stage 2 observations
│       ├── Tasseling_Stage                  # Stage 3 observations
│       └── Harvesting_Stage                 # Stage 4 observations
│
└── outputs/
    └── Kenya_Termite_FRN_Professional_Map.png  # Exported static map (400 dpi)
```

---

## Features

| Feature | Description |
|---|---|
| Multi-stage Dashboard | Dedicated tabs for First Weeding, Second Weeding, Tasseling, and Harvesting |
| Plot/Farmer Toggle | Switch all charts between grouping by Plot or by Farmer via a single radio button |
| Leaflet Map | Interactive map of farmer trial locations from GPS geolocation data |
| Variety Attack Analysis | Infers hybrid vs. local maize attack labels from `Plot_*_...Attack...` column naming patterns |
| Population Reduction | Stacked bar and boxplot views of plant population loss per plot/farmer |
| Vigour Tracking | Vigour score distribution across plots and stages |
| Termite Species & Location | Species name vs. attack location breakdowns per stage |
| Robust Data Cleaning | Handles list-type columns, empty strings, `NA`/`None`/`-` values, and duplicate metrics |
| DT Attack Table | Sortable variety-level attack rate summary table |
| Static Map + Insets | County-level inset maps per organization with sub-county labels (publication-ready) |

---

## Data Sources

### `Longrains_Termite_FRN_Data_2025.xlsx`

| Sheet | Contents |
|---|---|
| `Farmer_Profile` | Farmer demographics and contact details |
| `Trial_setup` | Plot configuration, hybrid/local varieties planted, GPS geolocation, organization |
| `First_Weeding` | Weeding date, termite attack signs, termite species, location, vigour, moisture stress |
| `Second_Weeding` | Same metrics as First Weeding at second stage |
| `Tasseling_Stage` | Vigour, attack, termite location, pests/diseases at tasseling |
| `Harvesting_Stage` | Plant loss, stage most plants lost, inputs used (fertilizer, biopesticide), vigour |

**Plot columns** follow the pattern `Plot_1_metric`, `Plot_2_metric`, etc. The app dynamically pivots these into long format and infers plot names and attack labels from column naming conventions.

---

## Organizations & Trial Counties

| Organization | Counties |
|---|---|
| **SOFDI** | Kakamega, Vihiga |
| **RFRN** | Migori |
| **TEMBEA** | Siaya |

---

## Prerequisites

- **R** ≥ 4.1.0
- Internet connection (first run only, for `geodata::gadm()` map downloads)

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/termite-frn-dashboard.git
cd termite-frn-dashboard
```

### 2. Install R packages

```r
# Dashboard packages
install.packages(c(
  "shiny", "dplyr", "ggplot2", "tidyr", "plotly",
  "readxl", "leaflet", "forcats", "stringr", "DT"
))

# Map packages
install.packages(c(
  "sf", "geodata", "ggspatial", "cowplot"
))
```

### 3. Configure the data path

Update the file path at the top of `app.R`:

```r
xlsx_path <- "Data/Longrains_Termite_FRN_Data_2025.xlsx"
```

---

## Usage

### Run the Shiny Dashboard

```r
shiny::runApp("app.R")
```

### Generate the Static Map

```r
source("map.R")
# Saves: outputs/Kenya_Termite_FRN_Professional_Map.png (14×16 in, 400 dpi)
```

> **Note:** `map.R` requires `TrialSetup_data` to already be loaded in your R session (run the data loading section of `app.R` first, or source it separately).

---

## Dashboard Tabs

### Overview
- **KPI panel**: total farmers, organizations, average plot size, average plant loss
- **Leaflet map**: GPS-located trial sites, popup shows farmer name and organization
- **Trial plot farmer choice**: bar chart of farmer-selected plot types

### First Weeding
- Weeding dates by plot or farmer (jitter plot)
- Attack and population reduction (stacked bar)
- Signs of termite attack (bar)
- Termite species vs. attack location (stacked bar)
- Vigour scores by plot (stacked bar)
- Pests/diseases and moisture stress summary

### Second Weeding
- Weeding dates by plot or farmer
- Attack and population reduction
- Vigour scores
- Termite species and locations

### Tasseling
- Vigour scores by plot or farmer
- Attack and population reduction
- Termite species and locations
- Pests and moisture stress

### Harvesting
- Plant loss by plot (boxplot if numeric, stacked bar otherwise)
- Inputs used: fertilizer, biopesticide, conservation agriculture
- Stage most plants were lost
- Vigour by plot or farmer

### Variety Attack
Combines data from all four stages to answer: *which maize varieties are being attacked most?*

- **Dot plot**: attacked plots per farmer × variety, sized by attack count
- **Bar chart**: attack rate (proportion) per variety, grouped by plot type
- **Summary table**: total plots, attacked plots, and proportion attacked per variety (sortable)

Attack labels are inferred from `Plot_*_...Attack...` column names — if the column value mentions `"local"` or `"hybrid"`, the attack is mapped to the corresponding variety from `Trial_setup`.

---

## Map Outputs

`map.R` produces a **two-panel publication figure**:

**Top panel — Main Kenya map**
- All county boundaries (level 1)
- County name labels
- Trial site points colored by organization
- Scale bar and north arrow
- Light blue background (`#E6F2FA`)

**Bottom panel — Organization insets (side by side)**
- Sub-county boundaries (level 2) with name labels
- Trial site points per organization
- Organization-colored border around each inset
- Counties: SOFDI (Kakamega & Vihiga), RFRN (Migori), TEMBEA (Siaya)

**Organization color palette** (color-blind friendly):

| Organization | Color |
|---|---|
| RFRN | `#1F77B4` (Blue) |
| SOFDI | `#FF7F0E` (Orange) |
| TEMBEA | `#9467BD` (Purple) |

---

*Source: Longrains Termite FRN Field Research Data, 2025*
