# Chicago Truck Crashes: When Severity Peaks

**Question:** When are crashes involving trucks in Chicago most likely to involve an injury, and what could a delivery or freight fleet do with that?

**Dashboard:** [View on Tableau Public](https://public.tableau.com/views/ChicagoTruckCrashesWhenSeverityPeaks/TruckCrashSeverity?:language=en-US&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

---

## Summary

Using eight years of Chicago crash data (2018-2025, about 878,000 crashes), I compared how often truck crashes involve an injury at different times of day.

- Truck crashes overnight (12-5 AM)  result in injury **15.2%** of the time, versus **8.3%** at midday (10 AM-3 PM). That is nearly twice as often.
- The gap appears in **every year** from 2018 to 2025.
- The gap is **not just darkness**: it persists on lighted roads.
- The **same crash types** are more severe overnight, so it is not just a different mix of crashes.

Overnight crashes are rare (roughly 3% of truck crashes), so this describes how severe a crash is when it happens, not how likely a crash is.

---

## Data

- **Source:** [Chicago Data Portal](https://data.cityofchicago.org/browse?category=Transportation&sortBy=most_accessed&page=1&pageSize=20), Traffic Crashes datasets (Crashes and Vehicles), downloaded on **11/06/2026**.
- **Tables used:** `crashes` and `vehicles`, joined on `CRASH_RECORD_ID`.
- **Date range used:** January 1, 2018 to December 31, 2025.

### Why 2018-2025 only

The raw data starts in 2013, but the early years are not comparable. Crash counts are tiny through 2014 and well below later levels in 2015-2016, with injury share of only 5-6%. Counts and injury share rise through 2017, which looks like a change in how crashes are reported, not a change on the road. From 2018 on, annual counts are stable (about 92k-119k). 2026 is excluded because the year is incomplete and recent records are often entered late.

### Vehicle groups

Vehicles were grouped using the `vehicle_type` and `vehicle_use` columns:

| Group | Definition |
|---|---|
| `local_truck` | Vehicle type is a single-unit truck |
| `freight` | Vehicle type starts with "tractor" (tractor with or without semi-trailer) |
| `commercial_van` | Van/mini-van with a vehicle use flagged as commercial |

The analysis combines `local_truck` and `freight` as "truck crashes" and keeps them separate where it matters. **Commercial vans are excluded from the time-of-day results** because overnight counts are too small to read (about 109 overnight crashes in the initial exploration). Vans are also the weakest definition, since the use field is officer-entered and delivery vans are often recorded as personal use or left blank.

Each crash is counted once per group, even if several trucks were involved.

---

## Method

1. Loaded the CSVs into **DuckDB** and queried them through **JupySQL** in a Jupyter notebook.
2. Profiled the data: row counts, null shares, category values, date range, and counts by year.
3. Defined vehicle groups in a SQL view and filtered crashes to 2018-2025 in a second view.
4. Calculated injury share (crashes with at least one reported injury divided by all crashes) by time block, lighting condition, and crash type, using `GROUP BY`, `FILTER`, CTEs, and window functions.
5. Checked robustness by recomputing the overnight and midday gap separately for each year.
6. Exported aggregated results to CSV and built the dashboard in **Tableau Public**.

Time blocks: overnight = 12-5 AM, midday = 10 AM-3 PM.

---

## Findings

### 1. Overnight truck crashes are nearly twice as likely to involve injury

- **Evidence:** 15.2% overnight vs 8.2% midday. Overnight beats midday in all eight years, by 2.9 points in the smallest year (2020) and 10.4 points in the largest (2024). Midday stays between 7.8% and 8.9%.
- **Possible action:** Treat overnight runs as the high-severity window. They are infrequent, but a crash is much more likely to cause injury.

### 2. The gap is not explained by darkness

- **Evidence:** On lighted roads after dark, overnight truck crashes that resulted in injury were 16.8% of the time (1,767) versus 8.3% for midday daylight crashes (23,606). The overnight gap remains on lighted roads, so lighting alone does not account for it.
- **Possible action:** Lighting-based route choices are unlikely to address the problem. Speed and driver-fatigue policies for overnight drivers are more consistent with the data.

### 3. The same crash types are more severe overnight

- **Evidence:** Nearly every crash type is more likely to involve injury overnight than at midday. Angle crashes: 40.3% vs 21.3%. Rear end: 24.2% vs 17.5%. Turning: 19.0% vs 12.0%. Parked motor vehicle: 6.9% vs 1.8%. Fixed object: 7.6% vs 2.1%. Sideswipe in the same direction: 8.3% vs 5.3%. The mix of crash types is similar in both blocks (parked-vehicle crashes are about a third of each), so the gap comes mostly from the same kinds of crashes being more severe, not from a different mix.
- **Possible action:** Because the gap shows up across types, measures aimed at a shared cause (speed, fatigue) fit better than type-specific rules.

These actions are worth considering, not proven. The data shows association and cannot separate speed, fatigue, and impairment.

---

## Limitations

- **Injury share is per crash, not per mile or per trip.** I have no mileage or traffic volume data, so this does not show that overnight driving is X times more dangerous.
- **Reported crashes only.** Reporting practices changed over time, and minor crashes may be reported differently for commercial vehicles.
- **Officer-coded fields.** Vehicle use, lighting, and crash type reflect the officer's judgment at the scene. Some categories are noisy (for example, a small number of overnight crashes coded as daylight).
- **A truck being in a crash does not mean the truck driver was at fault.**
- **Small samples.** Overnight counts are a few hundred per year, so single-year percentages are noisy (roughly 4 points either way). Pooled figures are more reliable.
- **Omitted categories.** Lighting and crash-type categories with small counts, and "unknown" lighting, are left out. Pedestrian, cyclist, and head-on crashes are rare overnight but among the most severe, so the crash-type chart does not cover all crash types.
- **The all-crash baseline (15.1%) is context only.** It includes pedestrians, cyclists, and motorcycles, so the fair comparison is overnight vs midday within truck crashes.

---

## Repo contents

```
├── README.md
├── analysis.ipynb      # SQL, checks, and dashboard exports
└── Out/                # aggregated CSVs behind the dashboard
```

Raw data files are not included because of their size. To reproduce, download the Crashes and Vehicles datasets from the Chicago Data Portal, set the file paths at the top of the notebook, and run all cells.

## Tools

DuckDB, JupySQL, Python (pandas), Tableau Public.

## Author

Juan Delgado | [LinkedIn](https://www.linkedin.com/in/jsdelgadomal/)
