# CRISP-DM Phase 2: Data Understanding

## 1. Collect initial data

Project data comes from three Chicago crash tables:
- `data/Traffic_Crashes_-_Crashes.csv` - main crash table
- `data/Traffic_Crashes_-_Vehicles.csv` - vehicle-level details
- `data/Traffic_Crashes_-_People.csv` - person-level details

Main table for the MVP:
- The core analytical base is the crash table.
- Vehicles and people tables are supporting sources for crash composition and severity analysis.

## 2. Describe data

Current local data size:
- Crashes: `1,045,043` rows, `48` columns
- Vehicles: `2,130,727` rows, `71` columns
- People: `2,293,745` rows, `29` columns

Time coverage of the crash table:
- From `2013-03-03 16:48:00` to `2026-04-13 02:27:00`

Main useful fields in the crash table:
- time: `CRASH_DATE`, `CRASH_HOUR`, `CRASH_DAY_OF_WEEK`, `CRASH_MONTH`
- location: `LATITUDE`, `LONGITUDE`
- conditions: `WEATHER_CONDITION`, `LIGHTING_CONDITION`, `ROADWAY_SURFACE_COND`
- road context: `POSTED_SPEED_LIMIT`, `TRAFFIC_CONTROL_DEVICE`, `TRAFFICWAY_TYPE`, `LANE_CNT`, `ALIGNMENT`, `ROAD_DEFECT`, `INTERSECTION_RELATED_I`, `WORK_ZONE_I`

Dataset overview:

| Dataset | Rows | Columns | Key fields | Time coverage |
| --- | ---: | ---: | --- | --- |
| Crashes | 1,045,043 | 48 | `CRASH_DATE`, `LATITUDE`, `LONGITUDE`, `POSTED_SPEED_LIMIT`, `WEATHER_CONDITION` | `2013-03-03 16:48:00` to `2026-04-13 02:27:00` |
| Vehicles | 2,130,727 | 71 | `CRASH_RECORD_ID`, `UNIT_TYPE`, `VEHICLE_TYPE`, `MANEUVER`, `OCCUPANT_CNT` | linked to crash dates |
| People | 2,293,745 | 29 | `CRASH_RECORD_ID`, `PERSON_TYPE`, `INJURY_CLASSIFICATION`, `AGE`, `SEX` | linked to crash dates |

Crash table key fields:

| Field | Role | Type | Missing % | Summary stats |
| --- | --- | --- | ---: | --- |
| `CRASH_DATE` | crash timestamp | datetime | 0.00 | `2013-03-03 16:48:00` to `2026-04-13 02:27:00` |
| `CRASH_HOUR` | hour-of-day context | integer | 0.00 | min `0`, median `14`, mean `13.19`, max `23` |
| `CRASH_DAY_OF_WEEK` | weekly context | integer | 0.00 | min `1`, median `4`, mean `4.12`, max `7` |
| `LATITUDE` | spatial position | numeric | 0.77 | min `0.00`, median `41.88`, mean `41.86`, max `42.02` |
| `LONGITUDE` | spatial position | numeric | 0.77 | min `-87.94`, median `-87.67`, mean `-87.67`, max `0.00` |
| `POSTED_SPEED_LIMIT` | road-speed context | numeric | 0.00 | min `0`, median `30`, mean `28.42`, max `99` |
| `WEATHER_CONDITION` | environment | categorical | 0.00 | top `CLEAR (818,876)` |
| `LIGHTING_CONDITION` | visibility | categorical | 0.00 | top `DAYLIGHT (668,913)` |

Vehicles table key fields:

| Field | Role | Type | Missing % | Summary stats |
| --- | --- | --- | ---: | --- |
| `UNIT_TYPE` | participant type | categorical | 0.11 | top `DRIVER (1,784,314)` |
| `VEHICLE_TYPE` | vehicle class | categorical | 2.38 | top `PASSENGER (1,296,156)` |
| `MANEUVER` | movement before crash | categorical | 2.38 | top `STRAIGHT AHEAD (968,011)` |
| `TRAVEL_DIRECTION` | travel heading | categorical | 2.38 | top `N (487,961)` |
| `OCCUPANT_CNT` | occupants in unit | numeric | 2.38 | min `0`, median `1`, mean `1.08`, max `99` |
| `EXCEED_SPEED_LIMIT_I` | police speeding flag | categorical | 99.89 | top `Y (1,803)` |

People table key fields:

| Field | Role | Type | Missing % | Summary stats |
| --- | --- | --- | ---: | --- |
| `PERSON_TYPE` | participant role | categorical | 0.00 | top `DRIVER (1,784,314)` |
| `INJURY_CLASSIFICATION` | injury outcome | categorical | 0.03 | top `NO INDICATION OF INJURY (2,085,158)` |
| `SEX` | participant sex | categorical | 1.71 | top `M (1,186,645)` |
| `AGE` | participant age | numeric | 28.98 | min `-177`, median `35`, mean `37.98`, max `110` |
| `DRIVER_ACTION` | pre-crash action | categorical | 20.31 | top `NONE (646,821)` |
| `SAFETY_EQUIPMENT` | protection usage | categorical | 0.28 | top `USAGE UNKNOWN (1,116,609)` |

## 3. Explore data

Main observations from the EDA:
- The dataset is strong enough for a dynamic crash-risk map and hotspot ranking.
- Geolocation is available for almost all crashes, so spatial analysis is feasible.
- Crash patterns change by hour, weather, lighting, surface condition, and road context.
- `RAIN`, `WET` pavement, and dark conditions are associated with higher injury-heavy crash shares than normal daytime clear conditions.
- Intersections and higher speed-limit groups are more dangerous than low-speed or parking-lot contexts.

Year distribution in the crash table:
- `2013`: `2`
- `2014`: `6`
- `2015`: `9,834`
- `2016`: `44,297`
- `2017`: `83,786`
- `2018`: `118,952`
- `2019`: `117,764`
- `2020`: `92,096`
- `2021`: `108,767`
- `2022`: `108,412`
- `2023`: `110,755`
- `2024`: `112,055`
- `2025`: `109,110`
- `2026`: `29,207`

Interpretation of yearly distribution:
- `2013-2014` are negligible and should not be used for analysis.
- `2015-2017` are much smaller than later years, so coverage is less stable.
- `2026` is a partial year and should be treated as incomplete.
- The most reliable modeling window is still `2018-2025`.

Main distribution of key condition fields:
- Weather is dominated by `CLEAR` (`818,876`), followed by `RAIN` (`87,129`) and `UNKNOWN` (`63,630`).
- Lighting is dominated by `DAYLIGHT` (`668,913`) and `DARKNESS, LIGHTED ROAD` (`228,697`).
- Surface condition is dominated by `DRY` (`765,866`), followed by `WET` (`132,984`) and `UNKNOWN` (`100,755`).

Interpretation of condition distribution:
- The key predictors are imbalanced, so common conditions will dominate simple frequency analysis.
- `UNKNOWN` is large enough to keep as an explicit category rather than dropping it.

Interpretation for the project:
- This data is suitable for relative zone risk scoring.
- It is less suitable for true trip-level probability estimation without extra exposure data.

## 4. Verify data quality

Main quality findings:
- Missing geolocation is low: about `0.76%` for latitude and longitude.
- Many categorical fields contain `UNKNOWN`, `OTHER`, or missing values.
- The people and vehicles tables include sparse fields, so they are more useful as secondary inputs than as the main predictive base.
- The data contains crash events only, not traffic volume or trip counts.

Data quality table:

| Issue | Affected columns | Frequency | Impact | Handling |
| --- | --- | --- | --- | --- |
| Missing or invalid geolocation | `LATITUDE`, `LONGITUDE` | `8,015` missing rows (`0.77%`); `70` zero-coordinate rows (`0.01%`) | these crashes cannot be placed reliably on the map | invalid and zero coordinates are filtered during spatial preparation |
| `UNKNOWN` condition labels | `WEATHER_CONDITION`, `LIGHTING_CONDITION`, `ROADWAY_SURFACE_COND` | weather: `63,630` (`6.09%`); lighting: `51,769` (`4.95%`); surface: `100,755` (`9.64%`) | weakens interpretation of external-condition features | `UNKNOWN` is retained as an explicit category |
| Sparse vehicle speeding flag | `EXCEED_SPEED_LIMIT_I` | `2,128,323` missing rows (`99.89%`) | too sparse for reliable use as a core predictor | the field is retained only as secondary context |
| Missing and invalid ages | `AGE` | `664,780` missing rows (`28.98%`); `11` invalid rows (`0.00%`) | limits people-level demographic analysis | missing ages are retained and invalid ages are removed during preparation |
| Uneven temporal coverage | `CRASH_DATE` | `167,132` rows (`15.99%`) outside `2018-2025`; `29,207` rows (`2.79%`) in partial `2026` | early and partial years can bias modeling | the main modeling window is restricted to `2018-2025` |
| No exposure denominator | all tables | not row-countable | prevents exact trip-level crash probability estimation | results are presented as a relative zone-risk model |

Main project limitation:
- Without denominator data, the model should be presented as a relative crash-risk / hotspot model, not as an exact probability of crash for a single trip.

Important feature-selection note:
- `FIRST_CRASH_TYPE` remains a leakage field and is not used as a safe predictor.
- A narrow derived exception may still be retained for scenario profiling: `is_pedbike`, used only for ped/bike scenario layers and downstream compatibility.

## 5. Phase 2 conclusion

The available data is sufficient for the first MVP of the project:
- a zone-based or grid-based risk map can be built,
- risk under rain, darkness, and wet-surface conditions can be compared,
- the crash table serves as the main source and vehicles/people serve as supporting tables.

The next step is the preparation of a modeling table with safe predictors only, with post-crash leakage fields such as injury outcomes and police-assigned causes excluded. A narrow scenario-only auxiliary field such as `is_pedbike` may still be retained for ped/bike layers and downstream compatibility, without being treated as a safe predictor.
