# CRISP-DM Phase 3: Data Preparation

## 1. Select Data

The preparation step uses the three Chicago crash tables:
- `data/Traffic_Crashes_-_Crashes.csv`
- `data/Traffic_Crashes_-_Vehicles.csv`
- `data/Traffic_Crashes_-_People.csv`

Main modeling table:
- The crash table is the core input for the route-risk model.
- Vehicles and people are used as crash-context tables that are merged by `CRASH_RECORD_ID`.
- The final modeling window is `2018-2025`.

Official modeling target:
- `severity_weight_sum`, a zone-level severity-weighted crash risk score.

## 2. Clean Data

Cleaning is conservative:
- invalid timestamps and invalid coordinates are removed,
- coordinates are limited to Chicago-like bounds,
- categorical fields are normalized with an explicit `MISSING` value,
- sparse or noisy fields are not imputed,
- post-crash leakage fields are excluded from the predictor set.

## 3. Construct Data

New features are created from the crash table:
- time features: `crash_year`, `crash_month`, `crash_day`, `crash_hour`, `day_of_week`, `hour_of_week`, `is_weekend`, `is_rush_hour`, `is_night`, `season`
- spatial features: `grid_id`, `grid_center_lat`, `grid_center_lon`
- road context features: `POSTED_SPEED_LIMIT`, `speed_limit_bin`, `TRAFFIC_CONTROL_DEVICE`, `DEVICE_CONDITION`, `WEATHER_CONDITION`, `LIGHTING_CONDITION`, `TRAFFICWAY_TYPE`, `LANE_CNT`, `ALIGNMENT`, `ROADWAY_SURFACE_COND`, `ROAD_DEFECT`, `INTERSECTION_RELATED_I`, `WORK_ZONE_I`, `WORK_ZONE_TYPE`, `WORKERS_PRESENT_I`
- derived scenario flags: `is_rain`, `is_snow`, `is_dark`, `is_wet_surface`, `is_intersection`, `is_work_zone`

Crash targets are also created:
- `injury_crash`
- `serious_injury_crash`
- `fatal_crash`
- `severity_weight`

Severity weights used in the project:
- `reported_not_evident = 1`
- `non_incapacitating = 2`
- `incapacitating = 5`
- `fatal = 20`

Vehicle-context features are aggregated by crash:
- `vehicle_rows`
- `vehicle_driver_rows`
- `vehicle_parked_rows`
- `vehicle_pedestrian_rows`
- `vehicle_bicycle_rows`
- `vehicle_driverless_rows`
- `vehicle_non_motor_rows`
- `vehicle_non_contact_rows`
- `vehicle_disabled_rows`
- `vehicle_equestrian_rows`
- `vehicle_occupant_cnt_sum`
- `vehicle_num_passengers_sum`
- `vehicle_speed_exceed_count`
- `vehicle_towed_count`
- `vehicle_fire_count`
- `vehicle_cmrc_count`

People-context features are aggregated by crash:
- `person_rows`
- `person_driver_rows`
- `person_passenger_rows`
- `person_pedestrian_rows`
- `person_bicycle_rows`
- `person_non_motor_rows`
- `person_non_contact_rows`
- `person_male_rows`
- `person_female_rows`
- `person_age_mean`
- `person_reported_not_evident_rows`
- `person_nonincapacitating_rows`
- `person_incapacitating_rows`
- `person_fatal_rows`
- `person_airbag_deployed_rows`
- `person_severity_weight`
- `person_injury_rate`

## 4. Integrate Data

All prepared parts are merged by `CRASH_RECORD_ID`.

Integration outputs:
- `prepared_crash_features.csv.gz`
- `vehicle_context_by_crash.csv.gz`
- `people_context_by_crash.csv.gz`
- `integrated_crash_context.csv.gz`
- `baseline_grid_time.csv.gz`
- `baseline_scenario_layers.csv.gz`
- `preparation_metadata.json`

Final integration summary:
- crash features: `870,705` rows
- vehicle context: `877,911` rows
- people context: `875,925` rows
- integrated context: `870,705` rows
- grid-time baseline: `754,678` rows
- scenario-layer baseline: `12,288` rows

Match coverage:
- vehicle context matches `100%` of modeled crash rows,
- people context matches about `99.77%` of modeled crash rows.

The integrated crash-context table is useful for analysis and severity interpretation, but the main route-risk model should still use safe pre-crash predictors from the crash table.

## 5. Format Data

The final prepared data is stored in `generated/prepared/` and organized into:
- one crash-level modeling table,
- one crash-context table for vehicles,
- one crash-context table for people,
- one merged crash-context table,
- one grid-time baseline table,
- one scenario-layer baseline table.

This preparation keeps the project aligned with the business goal:
- dynamic zone risk ranking,
- scenario-based risk layers,
- severity-weighted risk as the final official target.

