# 🌎 Amazon Pulse

Amazon Pulse is a browser-based monitoring dashboard for the Amazon basin that combines Sentinel-1 radar, NISAR L-band observations, and complementary satellite evidence to surface likely forest loss and flood-related change in a single interactive view.

This project was built for NASA Space Apps 2026 as part of the “Dancing with the SARs” challenge.

## What the app does

- Compares before/after radar views using Sentinel-1 C-band and NISAR L-band data.
- Highlights likely deforestation patches with a month-by-month clearing timeline.
- Shows a 3D terrain view where patch height reflects detected area.
- Lets users filter results by confidence level and inspect hotspot-level evidence.
- Includes NISAR flood and flooded-forest detection layers to separate true water signals from forest change.
- Surfaces hotspot summaries, time-series charts, and a guided-tour workflow for the biggest affected areas.

## Project layout

This repository contains the static viewer and the supporting Python pipeline files used to generate the final site data.

- `index (1) (1).html` — the interactive front-end dashboard
- `requirements_site.txt` — Python dependencies for the data-fetch and map-generation pipeline
- `README_PROJECT.md` — project-specific documentation for the generated site

## Quick start

For a very simple local preview, serve the repository root with Python and open the viewer in a browser:

```bash
cd /home/roaim/Desktop/spaceapps
python -m http.server 8000
```

Then open one of these in your browser:

- `http://localhost:8000/index%20(1)%20(1).html`
- or rename the file to `index.html` and use `http://localhost:8000/`

## How it works

```text
Google Earth Engine (batch, once)                 NASA ASF archive (streamed, nothing big downloaded)
Sentinel-1 time series → clearing date            NISAR L2 GCOV (HV + HH) → hotspot window only
Sentinel-2 NDVI · RADD alerts · slope · water     aligned to the same 20 m grid
            └───────────────┐                     ┌───────────────┘
                            ▼                     ▼
                      build_site.py → fusion · confidence · flood maps · pre-rendered tiles
                                         ▼
                              static website (MapLibre 3D) — no server, no live computation
```

Heavy processing is run once to build static outputs; the final frontend only reads pre-generated layers and data, so browsing the site remains fast and lightweight.

## Detection method in brief

1. Sentinel-1 C-band detection
   - Monthly median VH composites are compared against a reference baseline.
   - A clearing is flagged when radar backscatter drops persistently and fails to recover.
   - Permanent water, forest masks, and patch-size thresholds reduce false alarms.

2. NISAR L-band validation
   - Intact-forest HV baseline values are compared with NISAR observations inside each hotspot.
   - High-confidence detections require independent agreement from multiple signals.
   - L-band-only changes are called out separately for review.

3. Independent evidence checks
   - Sentinel-2 NDVI loss
   - RADD alerts
   - Slope and floodplain context
   - JRC water occurrence and seasonality

4. Confidence scoring
   - High = at least 2 independent sensors agree
   - Medium = 1 independent signal agrees
   - Low = no independent confirmation
   - Steep or flood-prone areas are flagged as needing review

5. Flood and flooded-forest detection
   - Open water is identified from low HV/HH signatures.
   - Flooded forest is identified when HH rises while HV remains comparatively stable.
   - Disturbance areas are excluded to avoid misclassifying actual clearings as floods.

## Rebuilding the site data

If you want to regenerate the dataset locally, install the Python dependencies first:

```bash
pip install -r requirements_site.txt
```

Then use the project’s data pipeline as needed:

```bash
python fetch_nisar.py --export-dir export --areas 0,3,4
python build_site.py --export-dir export --out site
python -m http.server 8000 -d site
```

The exact commands depend on the project pipeline you are using locally, but the general flow is: fetch NISAR scenes → build fused change products → assemble the final static site.

## Data sources and credits

- NISAR L2 GCOV — NASA / ISRO, distributed by ASF DAAC
- Sentinel-1 and Sentinel-2 — Copernicus / ESA, accessed via Google Earth Engine
- RADD alerts — Wageningen University
- ESA WorldCover
- JRC Global Surface Water
- SRTM
- AWS Terrain Tiles (Mapzen)
- Basemap — Sentinel-2 cloudless by EOX
- Map engine — MapLibre GL JS

## Limitations

- Radar changes are candidate detections, not field-verified proof of deforestation.
- NISAR data are still evolving; coverage and maturity vary by scene and hotspot.
- The rules used for confidence and flood detection are practical defaults rather than site-calibrated measurements.
- The latest Sentinel-1 month is still provisional until persistence can be checked.
- The analysis is focused on a curated set of hotspot boxes, not a full-basin monitoring product.

## License and contribution

This project is open source and contributions are welcome.

If you are improving the frontend or inventory logic, keep the project focused on transparent, evidence-based visual analytics for Amazon change detection.
