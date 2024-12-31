# Aggregated Dataset of Fatal Police Shootings (2015–2021)

**Author**: Cholyeon Cho  
**Date**: August 2024  

## Overview

This repository contains:

1. **Aggregated Dataset**: A CSV file merging observations from three major datasets—Fatal Encounters, Mapping Police Violence, and The Washington Post—on fatal police shootings (both on-duty and off-duty) from 2015 to 2021.
2. **Methodology Documentation**: A \LaTeX\ document (and compiled PDF if applicable) describing how the final aggregated dataset was created.

## Purpose

The goal of this project is to provide a unified dataset of fatal police shootings by merging multiple sources. Each source has a slightly different definition of whether and how a police officer was involved in an incident. This repository aims to:

- Normalize the **definitions** to “on-duty/off-duty fatal police shootings.”
- **De-duplicate** overlapping observations.
- **Identify** unique incidents contributed by each dataset.

## Data Sources

- **Fatal Encounters (FE)**
  - Collects observations where a police officer was present at the time of death.
- **Mapping Police Violence (MPV)**
  - Collects observations where a police officer directly inflicted fatal force (most often by gunshot).
- **Washington Post (WPOST)**
  - Collects observations where an on-duty police officer fatally shot a victim.

Each source has its own unique identifiers, naming conventions, and data completeness.

## Data Preparation

### 1. Filtering

- **Fatal Encounters** was filtered by incidents specifically involving gunshots and fatal intent.
- **Mapping Police Violence** was filtered by incidents where the cause of death was a gunshot.

### 2. Matching Algorithm

An algorithm was designed to detect mutual observations across the three datasets. For every pair of datasets, each record in one dataset was compared to every record in the other. A **threshold-based scoring** system was applied:

- **Name**: Fuzzy matching using Levenshtein distance (highest weight).
- **Age**: Numerical comparison.
- **Gender & Race**: Direct string matching (after normalizing).
- **Location**: Geospatial proximity (distance between latitude/longitude pairs).
  - (Note: Where MPV lacked lat/long, an API was used to retrieve them by address.)

If the cumulative score exceeded a certain threshold, the records were considered identical. When multiple matches exceeded the threshold, the one with the highest match score was chosen.

### 3. Manual Checks

Any record not present across all three datasets was **manually verified** for final inclusion or exclusion. Manual verification covered:

- **Missing Data**: Some incidents had incomplete information, requiring additional research.
- **Obscure Cases**: Incidents where circumstances of the shooting were unclear or contradictory.
- **Out-of-Scope Incidents**: Non-fatal shootings or incidents that did not meet the definition of on-/off-duty police shootings were excluded.

### 4. Results

Ultimately, the algorithm yields three “matched” versions of each source (matched FE, matched MPV, matched WPOST), indicating:
- Whether an observation was shared by other datasets.
- The unique identifiers that link records across the datasets.

### 5. Construction of the Final Aggregated Dataset

The final dataset draws from:
1. Matched records in **FE** (highest priority).
2. If an observation is missing in FE, use **MPV**.
3. If an observation is missing in both FE and MPV, use **WPOST**.

This hierarchical approach ensures that all available observations are included while avoiding duplicates.

## Data Dictionary

Below are the primary columns in the aggregated dataset:

1. **name** (`String`) — Name of the victim (may include fuzzy-matched or standardized version).
2. **date** (`Date`) — Date of the incident (in `YYYY-MM-DD` format).
3. **gender** (`String`) — Gender of the victim.
4. **age** (`Integer`) — Age of the victim at the time of the incident.
5. **longitude** (`Float`) — Longitude coordinate of the incident location.
6. **latitude** (`Float`) — Latitude coordinate of the incident location.
7. **state** (`String`) — U.S. state or territory where the incident occurred.
8. **on_off_duty** (`String`) — Indicates whether the officer was “on-duty” or “off-duty.”
9. **obscurity** (`String` or `Boolean`) — Flags if details about the shooting’s cause are unclear.
10. **fe_id** (`String` or `Integer`) — Unique ID from Fatal Encounters (if applicable).
11. **mpv_id** (`String` or `Integer`) — Unique ID from Mapping Police Violence (if applicable).
12. **wpost_id** (`String` or `Integer`) — Unique ID from Washington Post (if applicable).

## Repository Structure
