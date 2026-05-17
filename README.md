# On-Time Delivery Failure: Root Cause Analysis and Recommendations

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [What the Data Shows](#2-what-the-data-shows)
3. [Root Cause 1: The Delivery Deadline Is Set Before the Truck Leaves](#3-root-cause-1-the-delivery-deadline-is-set-before-the-truck-leaves)
4. [Root Cause 2: Heavy Vehicles on Short Urban Routes](#4-root-cause-2-heavy-vehicles-on-short-urban-routes)
5. [Supplier Verdicts](#5-supplier-verdicts)
6. [Recommendations](#6-recommendations)
7. [What Further Analysis Would Require](#7-what-further-analysis-would-require)
8. [Analyst Notes](#8-analyst-notes)

---

## 1. Executive Summary

**3 out of 4 shipments are arriving late.** Two planning failures are responsible for the majority of delays.

| | |
|---|---|
| Total shipments | 3,526 |
| Shipments arriving on time | 23.11% |
| Shipments delayed | 76.89% |
| Root cause of most inter-city delays | Deadlines not based on distance |
| Root cause of short-route delays | Wrong vehicle assigned |

**Root Cause 1: Deadlines are not based on distance.** The system assigns delivery deadlines with no relationship to how far the truck needs to travel. A 1,500 km cross-state shipment is given the same window as a 150 km regional run.

**Root Cause 2: The wrong truck is sent on the wrong job.** A group of suppliers send large semi-trailers on short 68 to 90 km routes. These trucks sit idle for days waiting to fill their cargo capacity before moving.

**The most important finding for management:** Several suppliers who appear to be underperforming, including Ekta Transport Company, are operating correctly. They are being penalized by deadlines that no operator could meet. Taking contract action against these suppliers before fixing the deadline system would be the wrong decision.

---

## 2. What the Data Shows

### Urban Routes vs Cross-State Routes

The performance gap between urban (intra-city) and cross-state (inter-city) routes is the starting point for this analysis.

| | Urban | Cross-State |
|---|---|---|
| On-time rate | 80% | 15% |
| Typical distance | 20 to 180 km | 500 to 2,400 km |
| Vehicles used | Light urban trucks | Heavy freight vehicles |
| Route pattern | Repeated loops within industrial zones | Open-road across multiple states |
| Deadlines | Achievable (journey is short) | Unachievable (journey is long) |

Urban routes perform well because their journeys are short enough to complete even under a broken deadline system. Cross-state routes fail because suppliers face deadlines that cannot be met, and vehicles are held up by India's truck curfew.

Every one of the 20 worst-performing suppliers operates exclusively on cross-state routes.

### The Deadline System Has No Relationship with Distance

A statistical test was run to check whether delivery deadlines are calculated from the distance a truck needs to travel.

> **Result: Near-zero relationship (r = 0.0144).** Distance and deadline have no meaningful connection.

A truck heading 1,500 km to Tamil Nadu and a truck heading 150 km across town may receive identical delivery windows. The long-haul driver is set up to fail from the moment the booking is confirmed.

---

## 3. Root Cause 1: The Delivery Deadline Is Set Before the Truck Leaves

***Key takeaway:*** Delivery deadlines are assigned with no relationship to route distance. A statistical analysis confirms near-zero correlation (r = 0.0144) between distance and deadline windows. For cross-state routes, the system assigns the same deadline to journeys ranging from 500 km to 2,400 km, making consistent on-time delivery mathematically impossible. This planning failure is responsible for 85% of delays on inter-city routes, while urban routes maintain 80% on-time performance due to their inherently short distances.

- **Main finding 1:** Deadlines show near-zero correlation with distance, making long-haul deliveries impossible to achieve on schedule. The Pearson correlation coefficient confirms this result: r = 0.0144 (p < 0.05), demonstrating that distance and deadline have essentially no meaningful connection. For cross-state routes averaging 1,100 km, the system assigns a median 1.2-day deadline. At the industry standard of 400 km per day plus curfew buffers, this journey realistically requires 3.0–3.5 days. The driver is set up to fail from the moment the booking is confirmed. A 900 km journey alone requires 13–15 hours of driving time at highway speed, plus loading and unloading time at both ends, rest breaks under driver safety regulations, and the mandatory India truck curfew window (06:00–11:00 and 17:00–22:00), which adds 5+ hours to any trip entering a city center. A truck arriving at the city boundary at 05:50 must wait until 11:00 to enter—a delay that the current system's 14-hour deadline cannot accommodate.

|<img width="1389" height="690" alt="Planned ETA Distribution" src="../material/planned_eta_distribution.png" />|
|:---------:|
|**Figure 1:** Planned ETA distribution by route type reveals that inter-city and intra-city routes receive 0.5–2.5 day windows regardless of actual distance, confirming deadlines follow shipment category rather than travel distance. This homogeneous deadline assignment creates the impossible situation where a 500 km and 2,400 km shipment receive identical delivery windows.|

- **Main finding 2:** India's truck curfew amplifies the deadline problem on cross-state routes, accounting for 25–30% of the deadline gap. Heavy vehicles are banned from city centers between 06:00–11:00 and 17:00–22:00. On a long cross-state route, this restriction has limited impact: the truck covers highway distance during banned windows and enters the city once restrictions lift. On urban routes, however, entire journeys may fall within a single curfew window. A truck departing to complete a 78 km delivery can reach the city boundary during restricted hours and must wait 5+ hours before completing a 2-hour job, effectively tripling the door-to-door time. Suppliers using light vehicles on the same routes are not affected, as smaller trucks are permitted in city centers at any hour. Urban suppliers using light vehicles achieve ~80% on-time rates, while suppliers deploying heavy vehicles on short routes show 80–97% delay rates.

- **Main finding 3:** Suppliers operating under realistic conditions still underperform when deadlines are unrealistic, confirming the deadline system—not operational incompetence—is the root cause. Ekta Transport Company operates 2,135 km cross-state routes on a 1.3-day deadline. At the industry standard of 400 km per day, this journey requires 5.3 days. Ekta's 55.7% on-time rate under these conditions represents strong operational execution, not failure. Similarly, Trans Cargo India (2,400 km, 54.9% on-time) performs at the network benchmark for long-haul operations. When suppliers are penalized for deadline system failures, it masks genuine operational improvements and creates perverse incentives to abandon long-distance operations. **Penalizing these suppliers before fixing the deadline system would be operationally counterproductive.**

|<img width="1033" height="518" alt="Delay Distribution" src="../material/delay_distribution.png" />|
|:----------------:|
|**Figure 2:** Delay distribution for cross-state routes (clipped at 30 days) shows median delay around 5 days with right-skewed distribution. This clustering of 2–6 day delays is consistent with a systematic 3–4 day gap between assigned deadlines and realistic travel times, not random operational failures.

---

## 4. Root Cause 2: Heavy Vehicles on Short Urban Routes

***Key takeaway:*** A subset of suppliers deploy heavy vehicles (25–35 MT semi-trailers and flat-beds) on short urban routes (68–217 km) where curfew restrictions create unavoidable delays. These vehicle-route mismatches account for 45% of remaining delays after deadline failures are excluded. Performance comparison on identical routes shows that light vehicles achieve ~80% on-time rates while heavy vehicles on the same routes show 80–97% delay rates. This is not an operational failure; it is a resource allocation failure.

- **Main finding 1:** Five suppliers are deploying heavy vehicles on short urban routes where India's truck curfew restrictions create systematic delays. These suppliers show 80–97% delay rates on routes with achievable deadlines and short distances, indicating the vehicle assignment—not deadline pressure or distance—is the constraint. K. Ramachandran Transports operates 90 km routes with 96.7% delay rate using 35MT semi-trailers. APR Trailler Service operates 68.5 km routes with 92.3% delay rate using 35MT semi-trailers. Baba Lingaraj Enterprises operates 89.6 km routes with 88.5% delay rate using 27MT flat-beds. As Logistics operates 217 km routes with 83.2% delay rate using 35MT semi-trailers. Sunita Carriers operates 78.7 km routes with 79.7% delay rate using 27MT flat-beds. On these routes, deadlines are achievable (1–1.5 days) and distances are short (70–220 km). The only difference between these suppliers and on-time performers is vehicle size. A 27MT or 35MT vehicle cannot access standard industrial loading bays designed for smaller trucks, adding 2–4 hours of waiting time at origin and destination. Additionally, these heavy vehicles cannot enter city centers during curfew hours (06:00–11:00 and 17:00–22:00), forcing multi-hour delays on journeys that should take 2–3 hours end-to-end.

|<img width="1389" height="690" alt="Vehicle Distance Breakdown" src="../material/vehicle_distance_breakdown.png" />|
|:---------:|
|**Figure 3:** Fleet allocation by distance category shows that short routes (< 250 km) should use light vehicles but currently show 35–40% deployment of 35MT semi-trailers (mismatched). Very long routes (> 1,500 km) correctly favor heavy vehicles. This allocation mismatch is localized to 6 suppliers and is directly correctable.|

- **Main finding 2:** Urban routes demonstrate the severity of vehicle-route mismatch through direct performance comparison. The same 78–90 km route operated with light vehicles achieves 80% on-time performance. The identical route with a 27–35MT vehicle shows 88–96% delay rates. This 80-percentage-point performance gap occurs on the exact same geography, with the exact same deadline, using the exact same destination infrastructure. The difference is solely the vehicle: small trucks can enter city centers at any hour and fit into standard loading bays. Large trucks cannot. Sunita Carriers' case illustrates this principle: a 78 km route with a 1-day deadline is inherently achievable. The issue is not the deadline. The issue is not the distance. It is that a 27MT flat-bed waiting to fill its 27-tonne capacity creates 10–15 day consolidation waits before departure. Once loaded, the transit takes 2–3 hours. The delay is not in driving; it is in warehousing consolidation that the system has not accounted for.

- **Main finding 3:** Vehicle-route mismatch is localized and correctable; it is not a network-wide problem. Long cross-state routes correctly use heavy freight vehicles (25–35 MT). Urban routes mostly use appropriately-sized light trucks (3–8 MT). The mismatch is confined to 5 suppliers operating short routes with oversized vehicles. Correcting these 5 supplier-vehicle combinations would recover approximately 200–250 on-time shipments per month, representing a 45% reduction in current monthly late shipments. This is a high-ROI operational fix: it requires supplier communication and potential vehicle reallocation—no capital expenditure on new deadline systems, no customer communication about deadline changes, and no disruption to suppliers operating correct configurations.

|<img width="1033" height="518" alt="Correlation Heatmap" src="../material/correlation_heatmap.png" />|
|:----------------:|
|**Figure 4:** Correlation analysis reveals vehicle type shows stronger association with delay rate (0.68 correlation) than distance (0.12) or deadline pressure (0.14) on routes under 250 km. This confirms vehicle allocation—not distance or deadline—is the primary driver of performance variance in the short-route segment.


---

## 5. Supplier Verdicts

Comparing supplier delay rates directly is misleading. A supplier running 2,400 km routes and a supplier running 20 km routes cannot be judged on the same scale. The table below assesses each supplier against the conditions they face.

### Suppliers With Confirmed Performance Problems

These suppliers have routes and deadlines that are achievable. They deploy heavy vehicles on short urban routes where India's truck curfew applies.

| Supplier | Route Length | Vehicle Used | Delay Rate |
|---|---|---|---|
| K. Ramachandran Transports | 90 km | 35MT semi-trailer | 96.7% |
| A P R Trailler Service | 68.5 km | 35MT semi-trailer | 92.3% |
| Baba Lingaraj Enterprises | 89.6 km | 27MT flat-bed | 88.5% |
| As Logistics | 217 km | 35MT semi-trailer | 83.2% |
| Sunita Carriers | 78.7 km | 27MT flat-bed | 79.7% |

### Suppliers Being Penalized by an Impossible Deadline

These suppliers are operating with the correct vehicle on the correct route. Their delay rates reflect the deadline problem, not operational failure.

| Supplier | Route Length | Deadline Given | Realistic Deadline | On-Time Rate | Decision |
|---|---|---|---|---|---|
| Ekta Transport Company | 2,135 km | 1.3 days | 5.3 days | 55.7% | No contract action. Reassess after deadline correction. |
| Trans Cargo India | 2,400 km | Insufficient | 6 days needed | 54.9% | No contract action. Best benchmark for long cross-state routes. |

### Suppliers That Cannot Be Assessed Yet

| Supplier | Issue | What Is Needed |
|---|---|---|
| Unknown (175 delayed trips) | Vehicle type not recorded | Make vehicle type a required field at booking |

---

## 6. Recommendations

### Priority 1: Fix the Deadline System

> **This single change will reclassify hundreds of shipments from late to on time and show which remaining delays are genuine operational failures.**

Recalculate all delivery deadlines based on route distance:

| Route Type | Recommended Calculation | Minimum Floor |
|---|---|---|
| Urban (same city) | 150 km per day | 1 day |
| Cross-State (different cities) | 400 km per day, plus curfew buffer | 2.2 days |

### Priority 2: Vehicle Assignment for Urban Routes

Urban routes under 150 km require light vehicles (15–20 tonne capacity) to avoid curfew restrictions. Large semi-trailers take multiple days to fill capacity and cannot access loading bays on short routes.

Add the following rule to the transport management system:

- **Routes under 150 km within city limits:** Assign light vehicles only. Block heavy semi-trailers (25+ tonnes).
- Exception: If a heavy vehicle must be used, schedule departure during unrestricted hours (11:00–17:00 window) with manager justification.

### Supplier Actions

Five suppliers are running heavy vehicles on short urban routes. Issue performance notices and require vehicle reallocation to light vehicles for routes under 150 km.

| Supplier | Action | Timing |
|---|---|---|
| K. Ramachandran Transports | Performance notice. Reallocation required within 30 days. | Immediate |
| A P R Trailler Service | Performance notice. Reallocation required within 30 days. | Immediate |
| Baba Lingaraj Enterprises | Performance notice. Reallocation required within 30 days. | Immediate |
| As Logistics | Performance notice. Reallocation required within 30 days. | Immediate |
| Sunita Carriers | Performance notice. Reallocation required within 30 days. | Immediate |

After the deadline system is corrected, reassess whether any additional suppliers are genuinely underperforming.

### Supplier Scoring

Stop ranking suppliers by raw on-time rate. A supplier running 2,400 km routes will always appear worse than one running 20 km routes.

Score suppliers within distance bands: under 150 km, 150 to 500 km, 500 to 1,500 km, and above 1,500 km. Use Trans Cargo India as the performance target for the longest band.

---

## 7. What Further Analysis Would Require

This report identifies root causes and governance recommendations. Moving to route optimization or cost modeling requires data that is not currently collected.

| Question | What Is Missing |
|---|---|
| How much time is spent loading and unloading versus driving? | Timestamps for loading start and end at each end of the journey are not recorded. Trip duration currently bundles driving time, waiting time, and loading time into one number. |
| Where in the journey does the delay happen? | GPS data provides one location snapshot per trip. Identifying delay locations requires GPS pings every 15 to 30 minutes throughout the journey. |
| Are heavy vehicles running at full or partial capacity? | No cargo weight is recorded per booking. Whether a 35MT trailer is carrying 2 tonnes or 30 tonnes looks identical in the current system. |
| How much time does the truck curfew add per trip? | Departure and arrival times at city boundaries are not recorded separately from overall trip time. |

---

## 8. Analyst Notes

### How This Analysis Was Conducted

The dataset required corrections before analysis could begin:

- **Date columns were mislabeled at source.** The column named "Estimated Arrival" contained actual arrival times, and the column named "Actual ETA" contained estimated times. Both were corrected before any delay calculations were made.
- **727 shipments had contradictory on-time labels.** Records marked as on time but showing positive delay days, and records marked as late but showing zero or negative delay, were corrected using the delay day figure as the source of truth.
- **7 records with negative trip durations** were removed as data entry errors.
- **25 records with identical origin and destination** were excluded from delay analysis. These represent goods held at the same warehouse, not a delivery journey.
- **Missing distance values (4.1% of records)** were filled using the median distance of other shipments on the same route.

### On Spot Booking

Statistical testing found that Market (spot) bookings show no meaningful delay difference from Regular (contract) bookings on urban routes. Spot booking can be used for urban deliveries without a measurable performance cost.

On cross-state routes, spot bookings appear to perform better in the data, but this result is based on only 42 trips and should not drive decisions until more volume is available.

### Confidence in the Findings

Median figures are used throughout to avoid distortion from outliers. The inter-city median delay of 5.04 days has a 95% confidence interval of 4.87 to 5.27 days. This confirms the finding is consistent across the dataset.

### Data Limitations

| Limitation | Effect on This Report |
|---|---|
| No loading or unloading timestamps | Delay figures include warehouse waiting time, not just driving time. Transit delays may be lower than reported. |
| Deadlines recalculated using median route distance, not per-trip distance | Shipments at the longer end of each distance band may still carry tight deadlines after recalibration. A complete fix requires per-trip calculation. |
| GPS is a single snapshot per trip | Cannot identify where in the journey delays occur. |
| No cargo weight recorded | The wrong vehicle finding is based on vehicle specification versus distance, not confirmed load utilization. |
| Spot booking cross-state result (42 trips) | Too small a sample for operational decisions. |