# Project Big Mine

This repository contains a CRISP-DM project on Chicago crash-risk analysis and route safety scoring.

## Folder Guide

- `crisp_stages/code/phase2/` contains the Phase 2 notebook for data understanding and EDA.
- `crisp_stages/code/phase3/` contains the Phase 3 notebook for data preparation and feature engineering.
- `crisp_stages/md/` contains the short CRISP-DM report text for phases 1-3.
- `crisp_stages/templates/` contains the LaTeX report template.
- `data/` contains the raw Chicago crash CSV files used by the notebooks.
- `generated/prepared/` contains derived outputs from Phase 3.

## Project Flow

1. Phase 1 defines the business goal and scope.
2. Phase 2 describes the data and checks whether the crash tables are usable.
3. Phase 3 prepares model-ready tables, integrates vehicle and people context, and builds baselines.

## Notes

- The notebooks in `crisp_stages/code/` are the main executable artifacts.
- The Markdown files in `crisp_stages/md/` are the report text for the CRISP-DM stages.
- `data/` and `generated/` are working directories and are not meant to be treated as source code.

