# CRISP-DM Phase 4: Modeling

## 1. Select modeling technique

Official modeling target:
- `severity_weight_sum` on `grid_id + crash_year + crash_month + crash_hour + is_weekend`

Main modeling decision:
- the project uses a relative zone-risk model rather than a trip-probability model,
- the training target stays aligned with Phase 3 and keeps severity weighting,
- the model reads `generated/prepared/prepared_crash_features.csv.gz` from Phase 3,
- `is_pedbike` is read from Phase 3 as an auxiliary scenario field derived from `FIRST_CRASH_TYPE`; it is retained for ped/bike scenario profiling and is not part of the safe predictor set,
- the model is built on a zero-filled grid-time panel so that low-risk and no-crash cells are represented explicitly.

Techniques used in Phase 4:
- baseline: historical exact-slot mean for `grid_id + crash_month + crash_hour + is_weekend`,
- main ML model: `HistGradientBoostingRegressor` trained on `log1p(severity_weight_sum)`,
- scenario adjustment layer: smoothed multipliers for `rain`, `dark`, `wet_surface`, `rain_dark_wet`, and `pedbike`.

Model settings table:

| Setting | Value |
| --- | --- |
| Target | `severity_weight_sum` on `grid_id + crash_year + crash_month + crash_hour + is_weekend` |
| Model | `HistGradientBoostingRegressor` trained on `log1p(severity_weight_sum)` |
| Hyperparameters | `loss=squared_error`; `learning_rate=0.05`; `max_iter=250`; `max_depth=6`; `min_samples_leaf=100`; `l2_regularization=0.1`; `random_state=42` |
| Train/test split | Train: `2018-2024`; holdout test: `2025` |
| Zero-row sampling | Full train panel: `8,636,544` rows; positive train rows: `661,398`; zero rows: `7,975,146`; sampled zero rows: `750,000` |
| Weights | Positive grid-time rows use `sample_weight=1.0`; sampled zero rows use `sample_weight=10.63353` |
| Scenario multipliers | Grid-level smoothed severity-per-crash ratios with `alpha=25`, clipped to `0.5-3.0`; `pedbike` combinations clipped to `0.5-4.0`; global multipliers: `rain=1.2209`, `dark=1.3087`, `wet_surface=1.0931`, `rain_dark_wet=1.4428`, `pedbike=3.0` |

## 2. Generate test design

Time split:
- train window: `2018-2024`
- test window: `2025`

Modeling panel summary:
- modeled crash rows: `870,705`
- unique grids: `2,142` in the train-window grid universe
- positive train grid-time rows: `661,398`
- full train panel size: `8,636,544`
- zero rows in full train panel: `7,975,146`
- sampled zero rows for fitting: `750,000`
- zero-row sample weight: `10.63353`
- full 2025 test panel: `1,233,792` rows

The grid lookup is fit from `2018-2024` training crashes only, so 2025 holdout-only grid cells do not define the modeled panel.

Reason for zero-row sampling:
- the full train panel is highly sparse,
- sampled zeros keep the fit computationally manageable,
- sample weights preserve the original zero-heavy structure of the panel.

## 3. Build model

Feature groups used in the model:
- spatial: `grid_center_lat`, `grid_center_lon`
- calendar: `crash_month`, `crash_hour`, `is_weekend`, `is_night`, `is_rush_hour`, sine/cosine time encodings
- historical priors: `grid_time_mean_target`, `grid_mean_target`, `grid_nonzero_slot_share`, `grid_month_mean_target`, `grid_hour_mean_target`, `grid_weekend_mean_target`
- global priors: `global_mean_target`, `global_time_mean_target`, `global_month_mean_target`, `global_hour_mean_target`, `global_weekend_mean_target`
- grid profile: `avg_speed_limit`, `rain_share`, `dark_share`, `wet_surface_share`, `intersection_share`, `work_zone_share`, `pedbike_share` (auxiliary ped/bike scenario profile), `injury_share`, `serious_injury_share`, `fatal_share`, `train_crash_total`, `train_severity_per_crash`

Saved modeling artifacts:
- `generated/modeling/severity_risk_histgb.joblib`
- `generated/modeling/grid_time_predictions_2025.csv.gz`
- `generated/modeling/grid_risk_predictions_2025.csv.gz`
- `generated/modeling/scenario_multipliers.csv.gz`
- `generated/modeling/scenario_global_multipliers.csv`
- `generated/modeling/feature_importance.csv`
- `generated/modeling/model_metrics.json`
- `generated/modeling/modeling_metadata.json`

## 4. Assess model

Row-level 2025 results:
- baseline MAE: `0.07243`
- baseline RMSE: `0.47341`
- model MAE: `0.05398`
- model RMSE: `0.44389`

Grid-level 2025 results:
- baseline MAE: `7.86054`
- baseline RMSE: `11.82299`
- model MAE: `11.94731`
- model RMSE: `19.49520`
- model grid-level `R²`: `0.36334`

Hotspot ranking quality:
- top-100 overlap for baseline: `0.66`
- top-100 overlap for model: `0.67`

Interpretation of the results:
- the learned model improves dynamic row-level risk estimation compared with the historical baseline,
- the historical exact-slot mean is still a very strong benchmark for yearly grid aggregates,
- the learned model is more useful for dynamic risk scoring and hotspot search than for replacing long-run historical totals.

Main drivers from permutation importance:
- `grid_time_mean_target`
- `grid_mean_target`
- `grid_nonzero_slot_share`
- `dark_share`
- `train_severity_per_crash`

## 5. Final modeling outputs

Global scenario multipliers:
- `rain = 1.2209`
- `dark = 1.3087`
- `wet_surface = 1.0931`
- `rain_dark_wet = 1.4428`
- `pedbike = 3.0` (clipped upper bound)

Final scoring logic:
- predict base grid-time risk with the trained gradient-boosting model,
- apply a smoothed scenario multiplier for the requested external condition,
- convert the adjusted prediction to a `0-100` relative risk score using the 99th percentile of model predictions as the scale reference.

Example score from the saved pipeline:
- `grid_id = 58_52`
- weekday `18:00`, month `11`, scenario `rain_dark_wet`
- base prediction: `0.50673`
- adjusted prediction: `0.79713`
- final risk score: `100`

Phase 4 conclusion:
- the modeling part is implemented and reproducible,
- the project now has a fitted severity-risk model, a historical comparison baseline, and scenario-specific risk multipliers,
- the final prototype can rank dangerous zones and adjust the score for rain, darkness, and wet-surface conditions.
