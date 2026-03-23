# 🚗 Dynamic Pricing Engine for Urban Parking Lots
### Summer Analytics 2025 — Capstone Project | Consulting & Analytics Club × Pathway

> **Built a real-time dynamic parking pricing engine that automatically adjusts prices across 14 urban parking lots based on live demand signals, replacing static pricing to reduce overcrowding and maximize lot utilization.**



## 📘 Overview

Urban parking spaces are limited and often poorly utilized due to static pricing. This project introduces a **real-time dynamic pricing engine** for 14 urban parking lots using streaming data and demand modeling.

The system processes live data every 30 minutes and automatically adjusts prices based on:
- How full the lot is (occupancy rate)
- How many cars are waiting (queue length)
- Nearby traffic conditions
- Special events and holidays
- Type of vehicle entering

This is the final capstone project for **Summer Analytics 2025**, hosted by the **Consulting & Analytics Club × Pathway**.



## 🛠 Tech Stack

| Category | Tools |
|---|---|
| Language | Python (Google Colab) |
| Streaming Framework | Pathway |
| Data Processing | Pandas, NumPy |
| Visualization | Matplotlib, Bokeh, Panel |
| Algorithms | Tumbling Window Aggregation, Haversine Distance, Demand Function Modeling, Surge/Discount Pricing |

---

## 📁 Repository Structure

```
📁 Dynamic-Pricing-Urban-Parking/
├── dynamicparking_notebook.ipynb   ← Main notebook with all models
├── README.md                       ← This file
├── dataset.csv                     ← Raw parking data (14 lots, 73 days)
├── out_model1.csv                  ← Baseline model output
├── out_model2.csv                  ← Demand-based model output
├── out_model3.csv                  ← Surge/discount model output
├── out_competitive.csv             ← Competitive model output (all 14 lots)
├── rerouting_alerts.csv            ← Lots flagged for rerouting
└── pricing_dashboard.png           ← Final 4-panel visualization
```


## 📊 Dataset

Data collected from **14 real urban parking lots in Birmingham, UK** over **73 days**, sampled every **30 minutes from 8:00 AM to 4:30 PM**.

| Feature | Description |
|---|---|
| `Occupancy` | Number of vehicles currently parked |
| `Capacity` | Maximum vehicles the lot can hold |
| `QueueLength` | Vehicles waiting outside the lot |
| `TrafficConditionNearby` | low / average / high |
| `IsSpecialDay` | 1 = holiday or event, 0 = normal |
| `VehicleType` | car, bike, truck, cycle |
| `Latitude / Longitude` | GPS coordinates for competitive analysis |



## 🧠 Architecture

```
Raw CSV (14 lots × 73 days × 18 readings/day)
        ↓
Timestamp parsing + data cleaning (Pandas)
        ↓
Pathway streams data row by row (real-time simulation)
        ↓
Tumbling windows group data by day per lot
        ↓
┌─────────────────────────────────────────┐
│  Model 1 → Baseline Linear Pricing      │
│  Model 2 → Demand-Based Pricing         │
│  Model 3 → Surge / Discount Pricing     │
│  Model 4 → Competitive Pricing          │
└─────────────────────────────────────────┘
        ↓
pw.run() executes the full pipeline
        ↓
Matplotlib 4-panel dashboard → pricing_dashboard.png
```



## 📈 Pricing Models

### ✅ Model 1 — Baseline Linear

A simple reference model where price increases with daily occupancy fluctuation.

```
Price = $10 + 5 × (occ_max - occ_min) / capacity
```

- Smooth and predictable
- Acts as a benchmark for comparison
- Price range: ~$11–$15

---

### ✅ Model 2 — Demand-Based Pricing

Constructs a multi-factor demand score using all available signals.

```
Demand = α×(Occ/Cap) + β×QueueLength + δ×IsSpecialDay + VehicleWeight + TrafficWeight

Price = $10 × (1 + 0.8 × NormalizedDemand / 5)
```

**Signal weights:**
| Signal | Weight |
|---|---|
| Occupancy rate | α = 0.5 |
| Queue length | β = 0.3 |
| Special day | δ = 0.4 |
| Vehicle: truck | 1.4× |
| Vehicle: car | 1.0× |
| Vehicle: bike | 0.6× |
| Traffic: high | 1.4× |
| Traffic: average | 1.0× |
| Traffic: low | 0.6× |

- Price range: ~$15–$22
- More adaptive and realistic than Model 1



### ✅ Model 3 — Surge / Discount Pricing

Applies Uber-style surge pricing with clear occupancy thresholds.

```
Occupancy > 80%  →  surge multiplier (1.5×)
Occupancy < 30%  →  discount multiplier (0.8×)
Otherwise        →  normal (1.0×)

Final Price = base × multiplier × special_day × traffic × vehicle × queue_factor
Price clamped between $5 and $20
```

- Prevents erratic pricing with hard clamp ($5–$20)
- Clear and explainable business logic



### ✅ Model 4 — Competitive Pricing (All 14 Lots)

The most advanced model — processes all 14 lots simultaneously and applies competitive logic.

```
occupancy_rate > 85%  →  1.1× (slight increase, avoid losing cars to competitors)
occupancy_rate < 30%  →  0.85× (undercut to attract traffic)
Otherwise             →  1.0× (hold demand-based price)

Price clamped between $5 and $20
```

**Rerouting logic:**
```
Occupancy > 90%  →  REROUTE_SUGGESTED  (red ✕ on dashboard)
Occupancy > 75%  →  HIGH_DEMAND
Otherwise        →  NORMAL
```

- Haversine distance function defined for geographic proximity filtering
- Automated rerouting alerts written to `rerouting_alerts.csv`



## 📊 Visualization

A **4-panel real-time Matplotlib dashboard** saved as `pricing_dashboard.png`:

| Panel | Model | Color |
|---|---|---|
| Panel 1 | Baseline Linear Price | Blue |
| Panel 2 | Demand-Based Price | Green |
| Panel 3 | Surge / Discount Price | Purple |
| Panel 4 | Competitive Pricing — all 14 lots | Multicolor + Red ✕ alerts |


## ▶️ How to Run

1. Open `dynamicparking_notebook.ipynb` in **Google Colab**
2. Run the install cell:
```python
!pip install pathway bokeh --quiet
```
3. Run all cells **sequentially from top to bottom**
4. After `pw.run()` completes, run the matplotlib plotting cell
5. View `pricing_dashboard.png` in the output


## 📜 Assumptions

- Base price = **$10**
- Price range bounded between **$5 and $20** (hard clamp)
- Vehicle type weights: truck (1.4×) > car (1.0×) > bike (0.6×) > cycle (0.5×)
- Traffic weights: high (1.4×) > average (1.0×) > low (0.6×)
- Rerouting triggered at **90% occupancy**
- Competitive radius = **1.0 km** (Haversine-based, defined for future cross-lot comparison)



## 🔑 Key Results

| Problem | Solution |
|---|---|
| Fixed prices cause overcrowding | Dynamic prices respond to real occupancy |
| One signal is insufficient | 5 signals combined into demand score |
| Prices can spike unrealistically | Hard clamp between $5–$20 |
| Lots operate independently | Competitive model handles all 14 together |
| Full lots frustrate drivers | Automated rerouting alerts at 90% |
| No visibility into pricing | 4-panel real-time dashboard |



## 🏁 Conclusion

This project demonstrates how intelligent, data-driven pricing models can improve urban parking utilization. By combining real-time stream processing (Pathway), multi-factor demand modeling, competitive pricing logic, and live visualization, the system replaces static flat pricing with a fully automated, explainable, and bounded dynamic pricing engine.


