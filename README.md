# Dynamic Pricing Engine for Urban Parking Lots

> Forecasts next-day parking demand using XGBoost and sets prices proactively — reducing overcrowding at full lots and increasing utilization at underused ones.

**Summer Analytics 2025 · Consulting & Analytics Club × Pathway**

---

## The Problem

Urban parking lots charge a flat rate all day regardless of demand. This causes two problems simultaneously:

- **Peak hours** → lots overflow, cars circle the block wasting fuel, drivers frustrated
- **Off-peak hours** → lots sit half empty, operators lose revenue

The solution is dynamic pricing — automatically raising prices when demand is high and lowering them when it is low. Same logic as Uber surge pricing, applied to parking.

---

## What This Project Does

1. **Loads** 18,368 real parking records from 14 lots in Birmingham, UK (73 days, every 30 minutes)
2. **Cleans** the data — parses timestamps, encodes categorical columns, computes occupancy rate
3. **Processes** it through a Pathway streaming pipeline using tumbling window aggregation
4. **Applies** 3 rule-based pricing models per day (baseline, demand-based, competitive surge)
5. **Forecasts** next-day peak occupancy using per-lot XGBoost models trained on 16 engineered features
6. **Sets prices proactively** — so the right price is ready before demand hits, not after
7. **Alerts** when any lot exceeds 90% occupancy and identifies nearby alternatives using Haversine distance

---

## Results

| Metric | Value |
|--------|-------|
| Best lot R² | 0.77 |
| Average MAPE across all lots | 9.56% |
| Lots with reliable ML predictions (R² > 0.4) | 5 of 14 |
| Rerouting alerts generated | 37 |
| Price range (clamped) | $5 — $20 |

---

## Architecture

```
Raw CSV (18,368 rows · 14 lots · 73 days)
        │
        ▼
Data Cleaning (Pandas)
  - Parse timestamps
  - Encode traffic / vehicle type categories
  - Compute occupancy rate
        │
        ▼
Pathway Streaming Pipeline
  - Processes data row-by-row
  - Tumbling window aggregation (daily)
  - 3 pricing models run in parallel
        │
        ├──────────────────────────────────┐
        ▼                                  ▼
Rule-Based Prices                  XGBoost Forecasting
(all 14 lots)                      (one model per lot)
                                   - 16 engineered features
                                   - Lag variables
                                   - 7-day rolling averages
                                   - Time signals
                                          │
                                          ▼
                                   Hybrid Pricing System
                                   - ML pricing: 5 reliable lots (R²>0.4)
                                   - Rule fallback: 9 erratic lots
        │
        ▼
6-Panel Matplotlib Dashboard
+ Interactive Bokeh Visualizations
```

---

## Pricing Models

### Model 1 — Baseline Linear
Price increases with daily occupancy swing (max minus min occupancy that day).
```
Price = $10 + 5 × (occ_max - occ_min) / capacity
Range: $11 – $15
```

### Model 2 — Demand-Based
Combines 5 signals into a weighted demand score.
```
Demand = 0.5×(Occ/Cap) + 0.3×QueueLength + 0.4×IsSpecialDay + VehicleWeight + TrafficWeight
Price  = $10 × (1 + 0.8 × Demand / 5)
Range: $15 – $22
```

### Model 3 — Surge / Discount
Clear thresholds with a hard price clamp.
```
Occupancy > 80% → 1.5x surge multiplier
Occupancy < 30% → 0.8x discount multiplier
Final Price = clamp($5, computed_price, $20)
```

### Model 4 — Competitive
Processes all 14 lots simultaneously. Lots above 85% raise prices slightly; lots below 30% undercut the market. Rerouting alerts trigger at 90% occupancy using Haversine distance to find nearby alternatives within 1km.

---

## XGBoost Demand Forecasting

### Why per-lot models?
One model trained on all 14 lots combined gave R²=0.45 — each lot has different capacity, location, and demand patterns that confuse a shared model. Training one model per lot gave R²=0.77 on the best lot.

### Features (16 total)

| Category | Features |
|----------|----------|
| Today's data | peak occupancy, avg occupancy, min occupancy, queue max, queue avg, is_special_day, traffic avg, vehicle avg |
| Time signals | day of week, month, week of year, occupancy rate |
| Lag features | yesterday's peak, yesterday's queue, yesterday's occupancy rate |
| Rolling window | 7-day rolling average (61% feature importance) |

### Hybrid System Design

| Condition | Approach |
|-----------|----------|
| R² > 0.40 — 5 lots | XGBoost predicted occupancy drives tomorrow's price |
| R² ≤ 0.40 — 9 lots | Rule-based fallback using actual occupancy |

Negative R² means the model performs worse than simply predicting the mean. Forcing ML on unpredictable lots makes pricing worse, not better — so rule-based logic takes over for those.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python | Core language |
| Pathway | Streaming pipeline, tumbling window aggregation |
| XGBoost | Per-lot demand forecasting |
| Scikit-learn | Train/test split, RMSE, R², MAE, MAPE |
| Pandas | Data loading, cleaning, feature engineering |
| NumPy | Numerical operations |
| Matplotlib | 6-panel static dashboard |
| Bokeh | Interactive visualizations |
| Google Colab | Development environment |

---

## Dataset

Real parking occupancy data from Birmingham, UK.

| Field | Description |
|-------|-------------|
| SystemCodeNumber | Unique lot ID (e.g. BHMBCCMKT01) |
| Occupancy | Cars currently parked |
| Capacity | Maximum lot capacity |
| QueueLength | Cars waiting to enter |
| TrafficConditionNearby | high / average / low |
| IsSpecialDay | Holiday or event flag (1/0) |
| VehicleType | car / bike / truck / cycle |
| Latitude / Longitude | GPS coordinates |
| LastUpdatedDate / Time | Observation timestamp |

- 73 days of data
- 18,368 rows total
- 18 readings per day per lot (every 30 min, 8am–4:30pm)
- 14 parking lots

## How to Run

```bash
# Clone the repo
git clone https://github.com/yourusername/dynamic-parking-pricing
cd dynamic-parking-pricing

# Install dependencies
pip install pathway xgboost scikit-learn pandas numpy matplotlib bokeh

# Open in Google Colab or run locally
jupyter notebook parking_pricing.ipynb
```

---



---

*Built as part of Summer Analytics 2025 · Consulting & Analytics Club × Pathway*
