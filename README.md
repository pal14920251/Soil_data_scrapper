# Soil_data_scrapper
**Contents:**

_1.README.md_ — this file

_2.gemini_batch.py_ — included script (see code for exact CLI/flags)


_3.output _— harvested GeoJSON / CSV data

**Summary**

This repo implements a robust, three-stage pipeline to discover valid geographic IDs, retrieve configuration (cycle years and bounding boxes), then harvest soil-sample features by sending tiled WMS requests and deduplicating results. The approach replaces older scraping tactics by using the portal’s GraphQL and WMS APIs.

**Key features**

1.Programmatic discovery of State and District IDs via GraphQL queries.

2.Configuration retrieval (cycle years, district BBOX) from the public layers API.

3.WMS tiling algorithm to request GeoJSON features per tile (GetFeatureInfo + info_format=application/json).

4.Overlap-based tiling and deduplication using unique feature IDs to avoid duplicates.

5.Outputs structured GeoJSON/CSV suitable for downstream analysis.


**How it works — three stages**
__Stage 1 — Data Discovery (GraphQL)__
POST GraphQL queries to the discovery endpoint to enumerate State _ids and District codes.
Purpose: produce the list of valid (state_id, district_id) pairs used in later stages.
__Stage 2 — Configuration Retrieval__
Request the obfuscated public/layers endpoint with state_code and district_code query params to obtain:
   i.shcLayers — available cycle years (e.g., ["2025-26","2024-25"])
   ii.bbox — district bounding box (minx, miny, maxx, maxy)
These values are required to construct the WMS layer ID and the tiling grid.

__Stage 3 — Data Harvesting (WMS Tiling)__
Use the WMS GetFeatureInfo interface with info_format=application/json to retrieve GeoJSON features.
Tile the district BBOX using a small tile size and an overlap to avoid missing points on tile boundaries.
Parse GeoJSON responses, deduplicate by unique feature ID, and store results as GeoJSON/CSV.

**Requirements**
Python 3.8+
Check requirements.txt

**Installation**
1.Clone the repository:
'''
git clone <repo-url>
cd soil_data_scrapper
'''




A set of scraper scripts that automatically extract soil nutrient information—such as Nitrogen, Phosphorus, Potassium, pH, and more. These codes let you quickly fetch and structure soil data for analysis, modeling, and agricultural decision-support applications.
