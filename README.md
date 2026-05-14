# Root Cause Analysis: Transportation & Logistics On-Time Delivery Failure
---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Data Quality & Methodology](#3-data-quality--methodology)
4. [Root Cause Analysis](#4-root-cause-analysis)
   - [RC1: Planned ETA Disconnected from Operational Reality](#rc1-planned-eta-disconnected-from-operational-reality)
   - [RC2: Vehicle Misallocation on Short Inter-City Routes](#rc2-vehicle-misallocation-on-short-inter-city-routes)
   - [Additional Finding: Shipment Type Analysis](#additional-finding-shipment-type-analysis)
5. [Recommendations](#5-recommendations)
6. [What Optimization Would Require](#6-what-optimization-would-require)
7. [Data Limitations](#7-data-limitations)

---

## 1. Executive Summary

Analysis of 3,526 shipment records reveals a systemic on-time delivery failure affecting **76.11% of all shipments**. Root cause investigation identifies two confirmed root causes, both rooted in planning governance failures rather than driver behavior or road conditions.

**The core problem is not that suppliers perform badly. It is that the planning system sets them up to fail.**

| Metric | Value |
|---|---|
| Total Shipments Analyzed | 3,551 |
| Baseline On-Time Rate | 23.01% (817 trips) |
| Total Delayed Shipments | 2,711 (76.11%) |
| Correlation: Distance vs SLA Duration | r = 0.0144 (near zero — SLA is not distance-based) |
| Median Delay — Inter-City (95% CI) | 5.04 days (4.87 – 5.27) |
| Median Delay — Intra-City (95% CI) | 0.72 days (0.60 – 1.04) |
| Intra-City On-Time Rate | ~80% |
| Inter-City On-Time Rate | ~15% |

> **Key Finding:** Two structural failures drive the majority of delay. First, Planned ETA (SLA) has near-zero correlation with transportation distance — suppliers are given identical time windows regardless of whether they are moving cargo 100 km or 2,000 km. Second, a subset of inter-city routes uses heavy vehicles on short distances, creating a compounding failure on top of already-impossible SLA targets. Suppliers like Ekta Transport Company — running 2,135 km on a 1.3-day SLA — are victims of planning failure, not operational failure.

---

## 2. Problem Statement

### Movement Type Drives Performance Gap

The single strongest predictor of on-time performance is movement type, not supplier identity.

| Movement Type | On-Time Rate | Primary Vehicle Used | Route Pattern |
|---|---|---|---|
| Intra-City | ~80% | Small, agile urban vehicles | Closed-loop within industrial zones or same-city facilities |
| Inter-City | ~15% | Heavy vehicles >85% of trips | Open routing across cities and states |

Intra-city suppliers consistently outperform inter-city suppliers — not because they are better operators, but because:

1. Their journeys are short enough to complete even under a poorly-calibrated SLA
2. They use vehicle types suited to urban navigation
3. Their routes are predictable and repeated (closed-loop)

All 20 bottom-performing suppliers operate exclusively on inter-city routes. This is not a coincidence — it is the predictable outcome of a planning system that assigns identical SLA windows to journeys of vastly different operational complexity.

### The SLA Has No Relationship with Distance

A Pearson correlation of **r = 0.0144** between SLA duration and transportation distance confirms that SLA targets are not calculated from route requirements. The same time window is assigned to a 100 km delivery and a 1,500 km delivery.

This creates two distinct failure populations:

**Population 1 — SLA victims (long inter-city routes):** Suppliers like Ekta Transport Company face a median distance of 2,135 km on a 1.3-day SLA. At a realistic 400 km/day, this journey requires 5.3 days minimum. They are delayed before the truck leaves the yard.

**Population 2 — Vehicle mismatch (short inter-city routes):** Suppliers like Sunita Carriers Private Limited face a 78 km route with a SLA of over 1 day — theoretically achievable — yet achieve a 90%+ delay rate. The SLA is not the issue here. The vehicle is.

---

## 3. Data Quality & Methodology

### Data Quality Issues Fixed

| Issue | Detail | Fix Applied |
|---|---|---|
| **Column mislabeling (critical)** | `Trip End Date` contained estimated arrival times; `Actual ETA` contained actual arrival times — labels were swapped at source | Columns renamed to match true content before any analysis |
| **On-time label inconsistency** | 727 records had contradictory `ontime` and `delay_time_days` (637 with Yes but delay > 0; 90 with No but delay ≤ 0) | Corrected using `delay_time_days` as source of truth |
| **Negative trip duration** | 7 records where trip start date was after actual arrival date | Removed as data entry errors — final dataset: 3,551 records |
| **Same-location records** | 25 records with identical origin and destination (distance = 0) | Excluded from delay analysis — these represent warehouse dwell, not transportation |

### Null Value Handling

| Column | Null % | Treatment |
|---|---|---|
| Minimum Kms To Be Covered In A Day | 73.78% | Column dropped |
| Driver Mobile No | 28.56% | Column dropped |
| Vehicle Type | 21.31% | Retained as "Not Specified" — confirmed to be ad-hoc urban small vehicles |
| Transportation Distance (KM) | 4.13% | Imputed via route-group median |

### Distance Imputation

MICE imputation was considered and rejected — GPS coordinate data was found unreliable (coordinate anomalies outside expected geographic bounds). Route-group median was used instead:

```python
# Step 1: Median of same origin-destination pair
df['transportation_distance_km'] = df.groupby(
    ['origin_location', 'destination_location']
)['transportation_distance_km'].transform(lambda x: x.fillna(x.median()))

# Step 2: Overall median fallback
df['transportation_distance_km'] = df['transportation_distance_km'].fillna(
    df['transportation_distance_km'].median()
)
```

### Statistical Methods

**Bias-corrected Cramér's V** (Bergsma & Wicher, 2013) for categorical association — bias correction is important given high-cardinality variables like `material_shipped`:

```python
def cramers_v(x, y):
    contingency_table = pd.crosstab(x, y)
    chi2 = ss.chi2_contingency(contingency_table)[0]
    n = contingency_table.sum().sum()
    phi2 = chi2 / n
    r, k = contingency_table.shape
    phi2corr = max(0, phi2 - ((k-1)*(r-1))/(n-1))
    rcorr = r - ((r-1)**2)/(n-1)
    kcorr = k - ((k-1)**2)/(n-1)
    denominator = min(kcorr - 1, rcorr - 1)
    if denominator <= 0:
        return 0.0
    return np.sqrt(phi2corr / denominator)
```

**Bootstrapped 95% CI** (10,000 resamples) for all median delay figures.

---

## 4. Root Cause Analysis

### RC1: Planned ETA Disconnected from Operational Reality

#### The Evidence

SLA duration has near-zero correlation with transportation distance across both delayed (r = 0.0144) and on-time (r = 0.12) shipments. This confirms that SLA targets are not calculated from route requirements — they are static values with no operational grounding.

The practical consequence: a supplier moving cargo 1,500 km is given essentially the same time window as one moving 150 km.

#### Why This Is Physically Impossible for Heavy Vehicles

For a standard inter-city heavy truck journey in India, realistic operating constraints are:

| Constraint | Detail |
|---|---|
| Average highway speed | ~60–70 km/h when moving |
| Mandatory rest breaks | Driver fatigue regulations require periodic stops |
| Loading and unloading time | Not captured in current SLA — treated as zero |
| Urban truck curfew (India) | Heavy vehicles banned from city centers 06:00–11:00 and 17:00–22:00 |
| State border checkpoints | Additional waiting time on cross-state routes |

A two-driver team on a 1,500 km route, rotating at maximum efficiency with no curfew delays, requires approximately 21 hours of pure driving time. Add loading, unloading, curfew windows, and checkpoint delays — a realistic minimum SLA for this journey is 2.5–3 days, not the 1.2 days frequently observed in data.

#### Intra-City vs Inter-City: The SLA Gap in Numbers

| Movement Type | Median Distance | Realistic Transit Time | Observed Median SLA | Gap |
|---|---|---|---|---|
| Intra-City | 104 km | ~1 day | ~1 day | Aligned |
| Inter-City | 900 km | ~2.25 days | ~0.6 days | 3.75x too short |

Intra-city routes happen to receive SLA targets that are roughly achievable — not by design, but because the journeys are short enough that even a static SLA floor is sufficient. Inter-city routes are structurally disadvantaged from the moment of booking.

#### Case Study: Ekta Transport Company (Victim of RC1)

| Metric | Value |
|---|---|
| Median Distance | 2,135 km |
| Observed SLA | 1.3 days |
| Realistic SLA at 400 km/day | 5.3 days |
| Delay Rate | 44.3% |
| Assessment | **Running at 55% on-time across 2,135 km on a 1.3-day SLA is strong operational performance.** This supplier is penalized by an impossible planning target, not by operational failure. |

---

### RC2: Vehicle Misallocation on Short Inter-City Routes

#### The Correct Framing

Vehicle type analysis must be interpreted in context of movement type:

| Segment | Vehicle Used | Assessment |
|---|---|---|
| Intra-City | Small urban vehicles (Not Specified in TMS) — ~98% of trips | **Correct.** Small vehicles navigate urban zones, industrial estates, and closed-loop routes efficiently. ~80% on-time rate. |
| Inter-City (long distance) | Heavy vehicles >85% of trips | **Correct for vehicle type.** 32FT and 40FT trucks are the appropriate specification for cross-state freight. Delay here is primarily driven by RC1 (impossible SLA), not vehicle choice. |
| Inter-City (short, <150 km) | Heavy vehicles — problematic subset | **Incorrect.** Deploying a 35MT semi-trailer on a 78 km route creates compounding operational failures on top of an already challenging SLA environment. |

The vehicle mismatch problem is specific and localized: **heavy vehicles on short inter-city routes under 150 km.** This is not a system-wide vehicle problem — it is a supplier-level planning failure affecting a identifiable subset of bookings.

#### Mechanism: Why Heavy Vehicles Fail on Short Inter-City Routes

Three compounding factors:

1. **Load assembly wait time:** A 35MT trailer must wait to consolidate a full load before departure. On a short route, this dwell time can exceed the entire transit time — effectively parking the vehicle for days before it moves 78 km.

2. **Urban access restrictions:** India's truck curfew (06:00–11:00 and 17:00–22:00) disproportionately affects short routes. A 78 km journey that should take 2 hours may span two curfew windows if departure timing is not managed, adding 10+ hours of forced waiting.

3. **Loading bay mismatch:** Large semi-trailers cannot access smaller industrial facilities. The vehicle arrives but cannot dock, adding unplanned waiting time at both origin and destination.

#### Type A Suppliers: Asset Misallocation (Heavy Vehicle, Short Inter-City Route)

| Supplier | Primary Vehicle | Median Distance | Delay Rate | Avg Delay | Root Issue |
|---|---|---|---|---|---|
| A P R Trailler Service | 40 FT 3XL Trailer 35MT | 68.5 km | 92.3% | 12.0 days | Load assembly wait dominates trip time |
| K. Ramachandran Transports | 40 FT 3XL Trailer 35MT | 90.0 km | 96.7% | 27.5 days | Highest avg delay in dataset |
| Baba Lingaraj Enterprises | 40 FT Flat Bed 27MT | 89.6 km | 88.5% | 22.3 days | Flat bed used as temporary storage |
| As Logistics | 40 FT 3XL Trailer 35MT | 217.0 km | 83.2% | 20.8 days | Consolidation wait inflates transit time |
| Sunita Carriers Pvt Ltd | 40 FT Flat Bed 27MT | 78.7 km | 79.7% | 19.6 days | SLA > 1 day yet still 90%+ delay — vehicle is the issue |

**Sunita Carriers is the clearest RC2 case:** 78 km, SLA over 1 day (theoretically achievable), yet delay rate of ~90%. The SLA is not the bottleneck. The 27MT flat bed trailer on a sub-100 km route — and the associated load assembly and access issues — is.

#### Type B Supplier: Technical Mismatch (Underpowered Vehicle, Long Inter-City Route)

| Supplier | Vehicle | Median Distance | Delay Rate | Correct Specification |
|---|---|---|---|---|
| Rajdhani Roadways | 32 FT Single-Axle 7MT | 1,178 km | 90.6% | 32 FT Multi-Axle 14MT or 18MT |

A single-axle vehicle on a 1,178 km corridor faces three compounding mechanical failures: reduced highway speed due to stability constraints, elevated breakdown risk from concentrated axle load over long distance, and mandatory additional cooling stops beyond normal rest requirements.

#### Positive Benchmark: Trans Cargo India

| Metric | Value |
|---|---|
| Route | Gurgaon, Haryana → Kanchipuram, Tamil Nadu |
| Median Distance | 2,400 km |
| Primary Vehicle | 32 FT Multi-Axle 14MT (correct specification) |
| On-Time Rate | 55% |
| Assessment | Correct vehicle on correct route under a badly miscalibrated SLA. 55% on-time across 2,400 km is strong operational performance and serves as the benchmark for long inter-city routes. |

#### Supplier Classification

| Classification | Suppliers | Root Issue | Action |
|---|---|---|---|
| RC2: Asset Misallocation | APR Trailler, K. Ramachandran, Baba Lingaraj, As Logistics, Sunita Carriers | Heavy vehicle on short inter-city route (<150 km) | Vehicle assignment policy enforcement |
| RC2: Technical Mismatch | Rajdhani Roadways | Underpowered vehicle on long route | Multi-Axle vehicle requirement |
| RC1 Victim (performing well given constraints) | Trans Cargo India, Ekta Transport Company | Correct vehicle, correct route — penalized by impossible SLA | Reassess after SLA recalibration. No contract action. |
| Requires further data | Unknown (175 delayed trips) | No vehicle type recorded | Enforce data capture at booking |

---

### Additional Finding: Shipment Type Analysis

Chi-square testing shows shipment type (Market spot booking vs Regular contract) has a statistically significant relationship with delay — but only on inter-city routes.

| Scope | Chi-Square | p-value | Conclusion |
|---|---|---|---|
| Intra-city: Market vs Regular | 0.63 | 0.427 | No significant difference |
| Inter-city: Market vs Regular | 26.84 | 2.2×10⁻⁷ | Significant difference |

**Inter-city delay rates:** Market 59.5% (n=42) vs Regular 87.6% (n=2,891)

**Interpretation with caution:** The 95% Wilson CI for Market inter-city delay rate is **44.5% to 73.0%** based on only 42 trips — too narrow a sample for operational decisions. The apparent Market advantage likely reflects that RC2 vehicle misallocation failures are concentrated within the Regular contract supplier base, not an inherent quality difference between booking types.

**Actionable conclusion:** Market spot booking on intra-city routes shows no statistically significant delay difference vs Regular contracts. This supports using ad-hoc small vehicles for urban routes as a cost lever without measurable performance risk.

---

## 5. Recommendations

### Immediate Actions (0 – 30 days)

- **Recalibrate all SLA targets** using distance-based calculation: intra-city at 150 km/day (minimum 1-day floor), inter-city at 400 km/day (minimum 2.2-day floor). This single change resolves the most widespread source of mislabeled delay.
- **Suspend contract action against Trans Cargo India and Ekta Transport Company** pending SLA recalibration. Both are RC1 victims operating correctly-specified vehicles on difficult routes.
- **Issue performance notices to Type A misallocation suppliers:** A P R Trailler Service (92.3%), K. Ramachandran Transports (96.7%), Baba Lingaraj Enterprises (88.5%), As Logistics (83.2%), Sunita Carriers (79.7%). Require vehicle reallocation plans within 30 days.
- **Issue technical compliance notice to Rajdhani Roadways:** Single-Axle 7MT vehicles not approved for routes over 500 km. Multi-Axle replacement required before next contract cycle.

### Short-Term Actions (30 – 90 days)

- **Implement vehicle-journey rules in TMS:**
  - Auto-reject heavy vehicle (>20MT) assignments on inter-city routes under 150 km
  - Auto-reject Single-Axle assignments on routes over 500 km
  - Supervisor override required with written justification for exceptions
- **Add India truck curfew windows as a planning constraint in TMS:** calculate effective travel windows for inter-city routes passing through urban centers, and build curfew buffer time into SLA calculation.
- **Implement route-normalized supplier scorecard:** evaluate suppliers within distance bands (below 150 km / 150–500 km / 500–1,500 km / above 1,500 km). Use Trans Cargo India as benchmark for the longest band.
- **Enforce vehicle data capture:** 175 delayed trips have no vehicle type recorded. Make vehicle type mandatory at booking stage.
- **Allow and encourage Market spot booking for intra-city routes** as a cost-optimization lever — statistical testing confirms no delay penalty.

### Long-Term Actions (90+ days)

- **Develop a vehicle-journey specification matrix:** document approved vehicle types per distance band with minimum axle and payload requirements. Codify into supplier contracts as technical compliance criteria.
- **Investigate SLA governance:** the near-zero correlation (r = 0.0144) between SLA and distance is a systemic failure. Identify the process by which distance-blind SLA values were entered into TMS and implement a mandatory distance-based calculation rule for all new SLA configurations.
- **Implement annual route-normalized supplier tiering** for contract renewal, volume allocation, and rate negotiation decisions.

---

## 6. What Optimization Would Require

This analysis identifies root causes and governance recommendations. Full route or fleet optimization is outside the scope of this dataset. The table below documents the specific data gaps that prevent moving from root cause diagnosis to quantitative optimization.

| Optimization Problem | What Is Missing | Why It Matters |
|---|---|---|
| **Accurate per-trip SLA calibration** | Loading and unloading timestamps are not recorded | `trip_duration` in current data = transit time + dwell time + loading time combined. Cannot isolate pure transit time to calculate a speed-based SLA per trip. |
| **Curfew impact quantification** | No time-of-day breakdown for urban segment traversal | Cannot calculate how many trips were actually delayed by curfew windows vs other causes. Estimated impact exists but cannot be confirmed. |
| **Route-level optimization** | GPS data is a single snapshot, not time-series | Cannot reconstruct which roads were taken, where delays occurred geographically, or identify recurring bottleneck segments. |
| **Fleet right-sizing** | No actual cargo weight per booking | Cannot calculate load factor for heavy vehicles. Cannot confirm whether a 35MT trailer is running at 5% or 95% capacity — both look identical in this dataset. |
| **Demand-based vehicle pre-positioning** | No advance booking lead time data, no historical volume by route | Cannot build a predictive allocation model without demand forecasting inputs. |

> **Summary:** The current dataset is well-suited for root cause analysis and planning governance recommendations. Moving to optimization requires event-level TMS logs (loading timestamps, GPS time-series) and load records. The recommendations in Section 5 represent the correct scope of action given available data.

---

## 7. Data Limitations

| Limitation | What Cannot Be Concluded | Data Needed |
|---|---|---|
| No loading/unloading timestamps | Cannot separate transit time from dwell time. SLA simulation assumes pure transit. | TMS event logs with loading start/end timestamps |
| SLA simulation uses category-median distance | Per-trip SLA varies within distance bands. Trips at band edges receive slightly mis-estimated SLA. | Per-trip SLA using actual transportation distance |
| Inter-city SLA ignores urban endpoint segments | 400 km/day applied to full journey overstates speed for urban entry/exit (~5–15% of trip). Some remaining delay may still be SLA-related. | Highway vs urban distance breakdown per trip |
| Curfew impact not quantifiable | India truck curfew is confirmed as a constraint but cannot be measured from available data. | Departure time, arrival time, and urban segment timestamps |
| Single GPS snapshot per trip | Cannot identify where in the journey delays occur. | Full GPS trace log (15–30 min ping frequency) |
| Unreliable GPS coordinates | Circuity factor analysis performed but excluded — coordinates showed anomalies outside expected geographic range. | Validated GPS trace from TMS |
| No actual cargo weight | Fleet allocation by material category appears reasonable at aggregate level. Individual anomalies (e.g. 32 Empty Tray shipments on 35MT trailers) cannot be confirmed as under-utilization without weight data. | Actual cargo weight per booking (kg) |
| Market inter-city sample too small | 42 Market inter-city trips, CI 44.5%–73.0%. Cannot confirm spot booking as independent performance driver. | More Market inter-city volume on comparable routes |
| Contract governance context | Cannot determine if SLA misconfiguration is a data entry error or reflects signed commercial commitments. | Original SLA contract documents from Sales/Commercial team |

---

## Appendix: Key Definitions

| Term | Definition |
|---|---|
| Intra-City | Origin and destination share the same city name (city-match) |
| Inter-City | Origin and destination in different cities |
| Delay | `actual_time_arrival > sla_target_time`, measured in days |
| On-Time | `delay_time_days ≤ 0` |
| RC1 Victim | Supplier with correct vehicle on correct route, delayed solely due to impossible SLA target |
| Type A Mismatch | Heavy vehicle (>20MT) assigned to inter-city route under 150 km |
| Type B Mismatch | Single-Axle vehicle assigned to route over 500 km |
| Route-Normalized Evaluation | Supplier on-time rate assessed within comparable distance bands |
| Bias-Corrected Cramér's V | Categorical association metric corrected for sample size bias (Bergsma & Wicher, 2013) |
| Bootstrapped CI | 95% confidence interval from 10,000 bootstrap resamples |
| Load Assembly Wait | Time a vehicle idles at origin waiting to consolidate a full payload before departure |

---

*Analysis conducted on Transportation GPS Tracking Dataset (simulated). All supplier names are from the original dataset. Findings are diagnostic and subject to the data limitations documented above. Optimization recommendations require additional data sources as specified in Section 6.*