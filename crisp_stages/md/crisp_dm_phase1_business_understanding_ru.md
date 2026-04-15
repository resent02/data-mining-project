# CRISP-DM Phase 1: Business Understanding

## 1. Project context

Project idea:
- Build an interactive map of dangerous zones in Chicago that changes under external conditions such as rain, darkness, wet pavement, and later traffic conditions.

Current input data:
- Core crash dataset: `Traffic Crashes - Crashes`
- Source used in this workspace: `data/Traffic_Crashes_-_Crashes.csv`
- Current local analytical base: `1,045,043` crash records with date range `2013-03-03 16:48:00` to `2026-04-13 02:27:00`

Why this project matters:
- Crash points by themselves are hard to interpret.
- Users need zone-level understanding, not raw incident dots.
- A dynamic map can support route choice, awareness, and safety analytics better than a static hotspot layer.

## 2. Business objective

Primary objective:
- Create a map that identifies zones with elevated relative crash risk and updates the displayed risk when external conditions change.

Practical objective for MVP:
- Show which zones are historically risky overall and which zones become more risky specifically in rain, darkness, wet-surface, or vulnerable-road-user scenarios.

Target product statement:
- The MVP should answer: "Given current or selected conditions, which Chicago zones are relatively more dangerous than usual?"

## 3. Stakeholders and users

Potential end users:
1. Drivers choosing routes.
2. Cyclists and pedestrians looking for safer paths.
3. Students / researchers analyzing urban safety.
4. City-analytics or Vision Zero style stakeholders.

Key stakeholder needs:
1. Interpretability: why a zone is marked risky.
2. Freshness: map should react to external conditions.
3. Spatial clarity: danger should be shown as zones/segments, not just points.
4. Credibility: the app must not claim more certainty than the data supports.

## 4. Business questions

Core questions:
1. Which zones are consistently dangerous over time?
2. Which zones become worse in rain, snow, darkness, or wet-surface conditions?
3. Which road contexts are most associated with injury-heavy crashes?
4. Can the product distinguish between volume-heavy hotspots and severity-heavy hotspots?
5. Can the user filter by mode-specific danger, for example pedestrian/cyclist or dooring?

## 5. Success criteria

Business success criteria:
1. The map clearly ranks dangerous zones rather than showing an unreadable point cloud.
2. Users can switch between at least `overall`, `rain`, and `darkness` risk layers.
3. The explanations for risky zones are understandable: intersections, speed context, injury history, crash types.

Analytical success criteria:
1. The model or scoring logic performs better than a naive static hotspot map.
2. Top-K risky zones on future periods are recovered with stable precision.
3. The score remains interpretable and does not depend on hidden leakage from future data.

MVP success criteria:
1. Grid-based or segment-based map layer is generated.
2. Daily or on-demand scoring is possible.
3. The system can expose a simple API or file output for frontend rendering.

## 6. Analytical formulation

Recommended framing:
- This is a **relative spatial risk ranking** problem, not a pure binary crash-prediction problem for an individual trip.

Recommended first targets:
1. Zone-level crash intensity for the next time window.
2. Zone-level injury-weighted severity score for the next time window.

Recommended risk score concept:
- Combine crash frequency and severity into one interpretable zone score, then condition that score on weather and visibility.

## 7. Scope for MVP

In scope:
1. Chicago city crash data under CPD jurisdiction.
2. Historical hotspot analysis by zone.
3. Scenario layers for weather / surface / lighting.
4. Time-aware validation.
5. Grid-cell visualization as the first spatial unit.

Out of scope for the first MVP:
1. True trip-level crash probability.
2. Full coverage of interstate/freeway crashes not included in the source.
3. Perfect exposure-adjusted safety rates.
4. Causal claims such as "rain causes crash X at location Y".

## 8. Assumptions

Working assumptions:
1. Historical crash patterns contain usable signal for near-future relative hotspot ranking.
2. Weather, surface, and lighting fields are noisy but still informative.
3. Zone-level aggregation is more stable than point-level prediction.
4. Users value comparative risk tiers more than exact probability values.

## 9. Constraints

Data constraints:
1. The dataset contains crash events, not denominator traffic exposure.
2. Some condition fields include substantial `UNKNOWN` / `UNABLE TO DETERMINE`.
3. Coverage before late 2017 is incomplete for citywide modeling.
4. The official source excludes some roads outside CPD jurisdiction.

Product constraints:
1. The UI must avoid overclaiming precision.
2. Sparse zones need smoothing or minimum-support thresholds.
3. A dynamic layer requires external live data for weather and ideally traffic.

## 10. Risks

Primary business/analytics risks:
1. Exposure bias: high-volume corridors may look "dangerous" simply because more travel occurs there.
2. Reporting bias: desk reports and minor crashes distort some condition fields.
3. Temporal drift: crash patterns changed materially around 2020 and continue to evolve.
4. Spatial leakage: random train/test splits will overstate performance.
5. Trust risk: if the product presents hard probabilities, users may over-trust noisy estimates.

Mitigations:
1. Present results as relative risk tiers.
2. Use time-based validation only.
3. Keep `UNKNOWN` as explicit categories instead of silently dropping them.
4. Add exposure proxies and weather history in the next iteration.
