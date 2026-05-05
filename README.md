# Project Big Mine

This repository contains a CRISP-DM project on Chicago crash-risk analysis and route safety scoring.

## Setup

Use Python 3.11+ or 3.12.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Folder Guide

- `crisp_stages/code/phase2/` contains the Phase 2 notebook for data understanding and EDA.
- `crisp_stages/code/phase3/` contains the Phase 3 notebook for data preparation and feature engineering.
- `crisp_stages/code/phase4/` contains the Phase 4 modeling notebook.
- `crisp_stages/md/` contains the short CRISP-DM report text for phases 1-4.
- `data/` contains the raw Chicago crash CSV files used by the notebooks.
- `generated/prepared/` contains derived outputs from Phase 3.
- `generated/modeling/` contains fitted modeling artifacts, predictions, metrics, and scenario multipliers from Phase 4.

## Data Access

Raw CSV files are not tracked in Git because they are large. Download the Chicago Traffic Crashes datasets and place these exact files under `data/`:

- `data/Traffic_Crashes_-_Crashes.csv`
- `data/Traffic_Crashes_-_Vehicles.csv`
- `data/Traffic_Crashes_-_People.csv`

If the files already exist elsewhere, keep one copy and create symlinks:

```bash
mkdir -p data
ln -s /path/to/Traffic_Crashes_-_Crashes.csv data/Traffic_Crashes_-_Crashes.csv
ln -s /path/to/Traffic_Crashes_-_Vehicles.csv data/Traffic_Crashes_-_Vehicles.csv
ln -s /path/to/Traffic_Crashes_-_People.csv data/Traffic_Crashes_-_People.csv
```

Phase 2 and Phase 3 read the raw files from `data/`. Phase 4 reads the prepared crash table written by Phase 3, so run Phase 3 before Phase 4.

## Project Flow

1. Phase 1 defines the business goal and scope.
2. Phase 2 describes the data and checks whether the crash tables are usable.
3. Phase 3 prepares model-ready tables, integrates vehicle and people context, and builds baselines.
4. Phase 4 trains a severity-risk model, evaluates it on a 2025 holdout, and saves scenario-aware scoring artifacts.

## Run Instructions

Run the notebooks from the repository root in CRISP-DM order:

```bash
jupyter nbconvert --to notebook --execute --inplace crisp_stages/code/phase2/crisp_dm_phase2_data_understanding.ipynb
jupyter nbconvert --to notebook --execute --inplace crisp_stages/code/phase3/crisp_dm_phase3_data_preparation.ipynb
jupyter nbconvert --to notebook --execute --inplace crisp_stages/code/phase4/crisp_dm_phase4_modeling.ipynb
```

## Expected Outputs

Phase 3 writes:

- `generated/prepared/prepared_crash_features.csv.gz`
- `generated/prepared/vehicle_context_by_crash.csv.gz`
- `generated/prepared/people_context_by_crash.csv.gz`
- `generated/prepared/integrated_crash_context.csv.gz`
- `generated/prepared/baseline_grid_time.csv.gz`
- `generated/prepared/baseline_scenario_layers.csv.gz`
- `generated/prepared/preparation_metadata.json`

Phase 4 writes:

- `generated/modeling/severity_risk_histgb.joblib`
- `generated/modeling/grid_time_predictions_2025.csv.gz`
- `generated/modeling/grid_risk_predictions_2025.csv.gz`
- `generated/modeling/scenario_multipliers.csv.gz`
- `generated/modeling/scenario_global_multipliers.csv`
- `generated/modeling/feature_importance.csv`
- `generated/modeling/top_50_risky_grids_2025.csv`
- `generated/modeling/model_metrics.json`
- `generated/modeling/modeling_metadata.json`

## Notes

- The notebooks in `crisp_stages/code/` are the main executable artifacts.
- The Markdown files in `crisp_stages/md/` are the report text for the CRISP-DM stages.
- `data/` and `generated/` are working directories and are not meant to be treated as source code.
