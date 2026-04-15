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

Main project limitation:
- Without denominator data, the model should be presented as a relative crash-risk / hotspot model, not as an exact probability of crash for a single trip.

## 5. Phase 2 conclusion

The available data is sufficient for the first MVP of the project:
- build a zone-based or grid-based risk map,
- compare risk under rain, darkness, and wet-surface conditions,
- use the crash table as the main source and vehicles/people as supporting tables.

The next step is to prepare a modeling table with safe predictors only and avoid post-crash leakage fields such as injury outcomes and police-assigned causes.
