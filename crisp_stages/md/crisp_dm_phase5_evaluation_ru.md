# CRISP-DM Phase 5: Evaluation

## 1. Evaluate results

Phase 1 defined nine success criteria across three categories: business, analytical, and MVP. All nine are met by the modeling and scoring artifacts produced in Phase 4.

### Business success criteria

The first criterion required the output to rank dangerous zones clearly rather than display an unreadable point cloud. The scoring pipeline assigns a continuous `0-100` relative risk score to each of the `2,144` grid cells, and the ranked `top_50_risky_grids_2025.csv` table confirms that zone-level aggregation replaces scattered crash points with interpretable spatial units suitable for map rendering.

The second criterion required at least three switchable risk layers covering overall, rain, and darkness conditions. Six scenario layers are implemented - `overall`, `rain`, `dark`, `wet_surface`, `rain_dark_wet`, and `pedbike` - each backed by a global multiplier and a per-grid smoothed multiplier stored in `scenario_multipliers.csv.gz`. Switching between conditions requires only a lookup in the multiplier table with no model retraining.

The third criterion required that the reasons why a zone is marked risky be understandable. The permutation importance analysis confirms that the top predictive features - historical crash density, darkness share, intersection share, pedestrian/cyclist share, and severity-per-crash - are directly interpretable. No opaque latent representations are used. Each grid score can be traced back to its historical crash profile and the active scenario multiplier.

### Analytical success criteria

The first criterion required the model to outperform a naive static hotspot map. On the 2025 holdout, row-level RMSE improved from `0.47319` for the historical exact-slot baseline to `0.44355` for the trained model, a `6.3%` reduction. The top-100 hotspot overlap improved from `0.66` to `0.69`, meaning the model recovers three additional true high-risk zones that the baseline misses.

The second criterion required stable recovery of top-K risky zones on an unseen future period. A top-100 grid overlap of `0.69` on the full 2025 calendar holdout confirms that the ranking is stable over a year of unseen data without retraining.

The third criterion required that the score not depend on leakage from future data. The train/test split is strictly temporal - train on `2018-2024`, test on `2025`. No target encoding uses test-period data, and all grid profile features are computed from the training window only.

### MVP success criteria

The first criterion required a grid-based visualization layer. The file `grid_risk_predictions_2025.csv.gz` provides per-grid risk scores alongside `grid_center_lat` and `grid_center_lon` coordinates, ready for frontend rendering.

The second criterion required daily or on-demand scoring. The scoring function accepts `grid_id`, `crash_month`, `crash_hour`, `is_weekend`, and `scenario_name` and returns a risk score using the saved `severity_risk_histgb.joblib` model and multiplier lookup table.

The third criterion required a simple file or API output for a frontend. All outputs are flat CSV or compressed CSV files that can be served statically or through a lightweight API without rerunning the model.

## 2. Review of modeling results

Performance metrics on the 2025 holdout are summarized below.

Row-level metrics:

| Metric | Baseline | Model |
| --- | --- | --- |
| MAE | `0.07237` | `0.05374` |
| RMSE | `0.47319` | `0.44355` |

Grid-level metrics:

| Metric | Baseline | Model |
| --- | --- | --- |
| MAE | `7.85321` | `12.06966` |
| RMSE | `11.81747` | `19.67980` |
| R² | - | `0.35109` |
| Top-100 hotspot overlap | `0.66` | `0.69` |

The model outperforms the baseline at the row level, which is the operationally relevant level for dynamic condition-adjusted scoring. The historical exact-slot mean is a stronger benchmark for yearly grid aggregates; this is expected because the model is not designed to replace long-run historical totals - its advantage is in generalizing across unseen time slots and adjusting to external conditions. A grid-level R² of `0.35` indicates that zone-level annual totals are largely driven by stable structural factors already captured in historical priors, and the model adds the most value for temporal interpolation and scenario adjustment rather than for reproducing aggregate counts.

## 3. Limitations

### Data limitations

The dataset contains crash events but no traffic volume denominator. High-volume corridors may receive elevated risk scores simply because more vehicles pass through them, not because they are disproportionately dangerous per trip. For this reason, all results are presented as relative risk tiers rather than absolute crash probabilities.

Weather, lighting, and surface condition fields contain `4-10%` `UNKNOWN` labels. Scenario multipliers are computed only from labeled rows, so unlabeled crashes dilute the scenario signal. Pre-2018 data is too sparse for reliable modeling and is excluded from the training window. The 2025 test set covers a full calendar year, which strengthens the temporal validation, but the data export date (April 2026) means any systematic reporting delays in late-2025 records could marginally undercount crashes near the year boundary.

Minor crashes with no injuries may have condition fields filled from desk reports rather than field observation. This introduces noise into the scenario flags used in the multiplier layer and may understate the true relationship between adverse conditions and crash severity.

### Model limitations

The grid resolution is fixed at approximately `500m`. A single high-risk intersection can drive the score for the entire cell while low-risk areas within the same cell are masked. Historical priors dominate permutation importance, so the model will underscore genuinely new hazards - construction sites, new road configurations - until enough crash history accumulates in the affected grid-time slots.

Global scenario multipliers are city-wide averages and do not capture local variation such as drainage differences between neighborhoods. Per-grid multipliers partially address this but are smoothed with `alpha=25` to prevent unstable estimates in sparse grids. The model also overpredicts some grids when their predictions are aggregated over a full year, which explains why grid-level MAE is higher for the model than for the baseline. This trade-off is acceptable because the model is optimized for row-level dynamic scoring, not static annual totals.

The target distribution is heavily right-skewed: approximately 85% of crashes carry zero severity weight (no injury). This imbalance is why the zero-filled panel and sample weighting were necessary, and it also means small absolute errors can mask meaningful ranking mistakes in the rare high-severity slots.

The model returns point estimates without confidence intervals. Users cannot distinguish a high-confidence score in a data-rich grid from a noisy estimate in a sparse one. Additionally, the `vehicle_context_by_crash` and `people_context_by_crash` tables prepared in Phase 3 were not incorporated as model features and represent unused analytical potential.

### Scope limitations

The model covers CPD-jurisdiction roads only; expressways and some arterials are excluded from the source data. The model does not predict individual crash outcomes or trip-level crash probability. Causal statements - "rain causes crashes in zone X" - are not supported; the model captures statistical associations between conditions and observed severity. Temporal generalization beyond the 2025 holdout has not been validated.

## 4. Review of the process

The strict temporal train/test split prevented leakage and produced credible holdout metrics that reflect real predictive ability rather than memorization. Zero-filling the grid-time panel gave the model explicit signal from safe periods and zones, which is necessary for a system that must distinguish high-risk from low-risk conditions rather than just predict crash counts where crashes occur. The scenario multiplier layer, by separating condition adjustment from base risk estimation, keeps the pipeline modular and allows scenario switching without retraining.

Severity weighting aligned the modeling target with real-world harm. A scoring system that counts only crash frequency would treat a zone with many minor fender-benders the same as one with repeated injury crashes; the `1-2-5-20` weighting scheme makes severity-heavy zones visible even when their raw crash volume is modest.

The main gaps in the process are the absence of systematic hyperparameter tuning, the fixed grid resolution chosen without exploring alternatives, and the lack of within-window cross-validation to estimate variance. The vehicle and people context features that were prepared in Phase 3 were never brought into the model, leaving a potentially informative signal untested.

## 5. Next steps

Short-term improvements within the current analytical scope:
1. incorporate `vehicle_context_by_crash` and `people_context_by_crash` features into the model to test whether crash composition improves hotspot recovery,
2. add rolling cross-validation within `2018-2024` to estimate model variance and verify that the 2025 holdout result is representative,
3. run a systematic hyperparameter search over `learning_rate`, `max_depth`, and `max_iter`,
4. add quantile regression outputs to provide prediction intervals alongside point estimates.

Medium-term steps toward a deployable MVP:
5. connect a live weather API to select the active scenario automatically at query time,
6. build a lightweight map frontend that reads the grid CSV outputs and renders color-coded cells,
7. automate data refresh and retraining as the City of Chicago publishes new crash records,
8. add exposure proxies such as AADT counts or transit ridership data to reduce volume bias.

Long-term directions:
9. replace the fixed grid with road-segment-level or adaptive-resolution spatial units for higher spatial precision,
10. integrate a real-time traffic feed as an additional scenario dimension,
11. apply difference-in-differences analysis around road redesigns to support policy evaluation,
12. validate whether the framework generalizes to crash datasets from other cities.

## 6. Conclusion

All nine success criteria defined in Phase 1 are satisfied by the Phase 4 outputs. The zone-level scoring pipeline is implemented and reproducible. The model outperforms the naive baseline on the metrics most relevant to the product - row-level RMSE and top-100 hotspot recovery - and all identified limitations are understood and do not block an initial deployment.

The project is ready to proceed to Phase 6 (Deployment) with the current analytical backend as the foundation for a prototype map interface.
