# Soil_Scrapping
**Contents:**

1.README.md — this file <br>
2.main_code.py — included script (see code for exact CLI/flags) <br>
4.Requirements.txt <br>
3.output — harvested GeoJSON / CSV data <br>

**Summary**

This repo implements a robust, three-stage pipeline to discover valid geographic IDs, retrieve configuration (cycle years and bounding boxes), then harvest soil-sample features by sending tiled WMS requests and deduplicating results. The approach replaces older scraping tactics by using the portal’s GraphQL and WMS APIs.

**Key features**

1.Programmatic discovery of State and District IDs via GraphQL queries.

2.Configuration retrieval (cycle years, district BBOX) from the public layers API.

3.WMS tiling algorithm to request GeoJSON features per tile (GetFeatureInfo + info_format=application/json).

4.Overlap-based tiling and deduplication using unique feature IDs to avoid duplicates.

5.Outputs structured GeoJSON/CSV suitable for downstream analysis.


**How it works — three stages**
<br>
__Stage 1 — Data Discovery (GraphQL)__
POST GraphQL queries to the discovery endpoint to enumerate State _ids and District codes.
Purpose: produce the list of valid (state_id, district_id) pairs used in later stages.
<br>
__Stage 2 — Configuration Retrieval__
Request the obfuscated public/layers endpoint with state_code and district_code query params to obtain:
   i.shcLayers — available cycle years (e.g., ["2025-26","2024-25"])
   ii.bbox — district bounding box (minx, miny, maxx, maxy)
These values are required to construct the WMS layer ID and the tiling grid.

__Stage 3 — Data Harvesting (WMS Tiling)__
Use the WMS GetFeatureInfo interface with info_format=application/json to retrieve GeoJSON features.
Tile the district BBOX using a small tile size and an overlap to avoid missing points on tile boundaries.
Parse GeoJSON responses, deduplicate by unique feature ID, and store results as GeoJSON/CSV.

**Requirements** <br>
Python 3.8+ <br>
Check requirements.txt

**Installation** <br>
1.Clone the repository:
```
git clone <repo-url> 
cd soil_data_scrapper
```
2.Create and activate a virtual environment:
```
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
```
3.Install dependencies:
```
pip install -r requirements.txt
```

**Typical workflow**

Discovery — generate a list of State and District IDs:

Run the GraphQL discovery script to get _id and district codes.

Config retrieval — fetch shcLayers (cycles) and bbox for target districts:

Use the public layers API with state_code and district_code.

Harvesting — run the WMS tiler to request tiles across the district bbox:

Supply layer id {SC}_{DC}_shc_{CYCLE}, tile size (Δx, Δy), and overlap (e.g., 90%).

The script will parse GeoJSON, deduplicate by feature ID, and write outputs to output.



**Output format**

Primary outputs: GeoJSON files containing point features with soil attributes (N, P, K, pH, etc.).

A deduplicated CSV export is also generated for analysis workflows (columns depend on source features; inspect produced files).

**Reliability & best practices**

Use overlap when tiling to avoid boundary misses; deduplicate by the unique feature ID returned in GeoJSON. <br>
Respect request rates and consider adding polite sleeps and retries to avoid overloading servers.<br>
Validate GeoJSON/coordinates after harvest

**Ethics, legality & responsible use**

This repository documents a reverse-engineering methodology for extracting publicly served data.Always ensure your data collection complies with the target site’s Terms of Service and applicable laws.
Use the harvested data responsibly for research, analysis, or tools that benefit stakeholders (e.g., farmers, researchers).If in doubt about permitted use, consult legal counsel or the data provider.

**Troubleshooting**

If GraphQL responses change, re-run discovery to refresh valid State/District IDs.

If public/layers returns different fields, inspect the raw JSON — cycle names and bbox are required to build layer IDs.

For missing points, reduce tile size or increase overlap and re-run the tiler; verify deduplication keys.



A set of scraper scripts that automatically extract soil nutrient information—such as Nitrogen, Phosphorus, Potassium, pH, and more. These codes let you quickly fetch and structure soil data for analysis, modeling, and agricultural decision-support applications.
