# CapstoneFinalProject

---
# 🚗 Dynamic Pricing for Urban Parking Lots

## 📘 Overview

Urban parking spaces are limited and often poorly utilized due to static pricing. This project introduces a **real-time dynamic pricing engine** for 14 urban parking lots using streaming data and demand modeling. The system adjusts prices based on occupancy, queue length, vehicle type, traffic, and special day indicators.

This is the final project for **Summer Analytics 2025**, hosted by the **Consulting & Analytics Club × Pathway**.

---

## 🛠 Tech Stack

- **Python** (Google Colab)
- **NumPy & Pandas** – for model development and data handling
- **Pathway** – for real-time data simulation and processing
- **Bokeh** – for live visualization of price dynamics
- **Mermaid.js** – for architecture diagrams

---

## 🧠 Architecture Diagram


graph TD
    A[Real-time Data Stream] --> B[Pathway Streaming Processor]
    B --> C[Feature Extractor]
    C --> D1[Model 1: Linear Pricing]
    C --> D2[Model 2: Demand-Based Pricing]
    D1 --> E[Price Output]
    D2 --> E
    E --> F[Bokeh Visualization]



## 🔄 Workflow

1. **Data Simulation** using Pathway’s delay-injection system.
2. **Real-Time Feature Processing** like occupancy, queue length, and traffic.
3. **Pricing Models** that adjust prices dynamically.
4. **Bokeh Visualization** to display evolving prices across all 14 lots.

---

## 📈 Pricing Models

### ✅ Model 1: Baseline Linear

A simple model where prices rise as occupancy increases.

```python
Price(t+1) = Price(t) + α * (Occupancy / Capacity)
```

* Smooth and predictable
* Sets a benchmark for comparison

---

### ✅ Model 2: Demand-Based Pricing

Constructs a demand function using:

* Occupancy / Capacity
* Queue Length
* Traffic Congestion Level
* Special Event Indicator
* Vehicle Type Weighting

```python
Demand = α*(Occ/Cap) + β*Queue − γ*Traffic + δ*IsSpecialDay + ε*VehicleWeight
Price(t) = BasePrice * (1 + λ * NormalizedDemand)
```

* Ensures price stays between 0.5× and 2× of base
* More realistic and adaptive

---

## 📊 Visualization

* Live Bokeh dashboards for each parking lot
* Price trends updated every 30 minutes
* Smooth curves, easily interpretable by city planners

---

## 📜 Assumptions

* Base price = **\$10**
* Price range bounded between **\$5 to \$20**
* Vehicle types have weighted influence (e.g., trucks > cars > bikes)
* All features normalized for fair contribution

---

## 📂 Repository Contents

```
📁 Dynamic-Pricing-Urban-Parking/
├── CapstoneProjectFinalNotebook.ipynb
├── README.md
├── Dynamic_Pricing_Project_Report.docx
├── architecture_diagram.png (optional, export from Mermaid)
└── any other visual outputs / scripts
```

---

## ✅ How to Run

1. Open the Colab notebook.
2. Install `pathway` and `bokeh` if not already.
3. Run all cells sequentially to simulate the data stream and pricing.
4. View real-time plots in the output.

---

## 🔓 Access

* This repository is **public** and accessible to all reviewers.
* All models are implemented from scratch as per the guidelines.
* Report and notebook are well-commented and structured.

---

## 🏁 Conclusion

This system demonstrates how intelligent, data-driven pricing models can improve parking space utilization. It reflects real-world challenges and solutions using ML logic, streaming frameworks, and live visualizations.

---

> 📌 Note: Model 3 (Competitive Pricing with rerouting) is under development

