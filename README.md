# FM-AQ-KL — Malaysia air-quality data for Aurora fine-tuning

A basic, reproducible pipeline for turning **Malaysian Department of Environment (DoE)** ground-station air-quality
data into fine-tuning inputs for the **Aurora** machine-learning foundation model (air-quality / PM2.5
forecasting). It tidies the raw agency spreadsheets, runs a QA/QC pass, explores the data, and emits the
observation table in the schema Aurora's station fine-tune expects.

Worked example: **DoE Malaysia hourly PM2.5, 2023** — 65 stations across 15 states.

## Pipeline

| step | script | does |
|---|---|---|
| tidy | `tidy_malaysia_pm.py` | 15 per-state Excel sheets → one long table; local time (MYT) → UTC; raw PM2.5 kept (no API back-conversion). |
| QC | `qc_malaysia_pm.py` | physical + temporal checks (PM2.5 ≤ PM10, flatline, completeness); isolated spikes **flagged and kept** (may be real local fires); only impossibilities dropped. Writes a report + per-station scorecard. |
| explore | `malaysia_pm_2023_explore.ipynb` | network / per-station views, per-station monthly stats, and a station × day heatmap that surfaces regional haze episodes. Figures saved to `img/`. |
| FT prep | `prep_malaysia_ft.py` | emits the Aurora `load_obs`-schema obs (`datetime, pm25, lat, lon`) + the CAMS-IC init-date list the model side needs. |

## Data policy

Raw DoE spreadsheets and observation-bearing CSVs stay **local** (see `.gitignore`) — only code, the notebook,
figures, and derived summary statistics are versioned.

## Not included here (model side)

Fine-tuning also needs CAMS analysis initial conditions over the Malaysia domain on Aurora's 0.4° grid; that is
prepared in the main Aurora project, not this repo. This repo covers the **observation** half only.
