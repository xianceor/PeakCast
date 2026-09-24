# Electricity Peak Demand Prediction System

A full-stack machine learning web application that predicts electricity peak demand at both the daily and hourly level, built on the AEP transmission-zone load series merged with New York City weather data (Oct 2012 - Nov 2017).

Flask REST API backend, vanilla HTML/CSS/JS frontend, dual Random Forest classifiers.

Based on the research paper "Peak Electricity Demand Prediction Using Dual-Granularity Random Forest Classification: A Case Study on the AEP Transmission Zone" - Mayank, Aishwarya Shelke, Dr. Seema Shukla, Sharda University.

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Flask](https://img.shields.io/badge/backend-Flask-black)
![scikit--learn](https://img.shields.io/badge/ML-scikit--learn-orange)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Features

- Daily Peak-Day Prediction - calibrated probability that any given day will be a monthly peak-load day (ROC-AUC 0.907)
- Hourly Peak-Hour Localization - live model inference to flag which hours of the selected day are most likely to spike (ROC-AUC 0.910)
- Demand-Based Cost Risk Index - a per-hour economic risk score derived from the load ratio
- Grid Stress Indicators - colour-coded High / Moderate / Normal risk classification
- Interactive dashboard - date picker, load curve, cost-risk curve, peak-hour probability chart, feature-importance chart, all rendered client-side with Chart.js
- Clean REST API - the backend is fully decoupled from the frontend, so the same endpoints could power a mobile app or a different UI

---

## Architecture

```
┌─────────────────────┐        JSON over HTTP        ┌──────────────────────────┐
│   Frontend (SPA)     │ ───────────────────────────► │   Backend (Flask API)     │
│  HTML + CSS + JS      │ ◄─────────────────────────── │  RandomForestClassifier   │
│  Chart.js visuals     │                              │  (peak_day / peak_hour)   │
└─────────────────────┘                              └──────────────────────────┘
```

- Backend - Python + Flask. Loads both trained models once at startup, exposes `/api/meta`, `/api/day`, and `/api/feature-importance`, and serves the static frontend.
- Frontend - no framework, no build step. Fetches the API and draws everything with [Chart.js](https://www.chartjs.org/) (CDN-loaded).

---

## Project structure

```
electricity-peak-demand-fullstack/
│
├── backend/
│   ├── app.py                  # Flask REST API + static file server
│   ├── requirements.txt
│   ├── models/
│   │   ├── peak_day_model.pkl  # Trained Random Forest (daily)
│   │   └── peak_hour_model.pkl # Trained Random Forest (hourly)
│   └── data/
│       ├── daily_predictions.csv
│       ├── hourly_data.csv
│       └── top_peak_hours.csv
│
├── frontend/
│   ├── index.html              # Dashboard layout
│   ├── style.css                # Dark dashboard theme
│   └── script.js                # Fetches API data, renders Chart.js charts
│
└── README.md
```

---

## Getting started

### Prerequisites
- Python 3.10+

### 1. Clone the repo

```bash
git clone https://github.com/<your-username>/electricity-peak-demand-fullstack.git
cd electricity-peak-demand-fullstack
```

### 2. Install backend dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 3. Run the app

```bash
python app.py
```

### 4. Open it

Go to `http://127.0.0.1:5000` - the Flask server serves both the API and the frontend, so this one command is all you need.

Use the date picker (top-right) to explore any date between 1 Oct 2012 and 30 Nov 2017, the range covered by the training data.

---

## API reference

| Endpoint | Method | Description |
|---|---|---|
| `/api/meta` | GET | Dataset date range and summary stats |
| `/api/day?date=YYYY-MM-DD` | GET | Full bundle for one date: peak-day probability, load curve, cost-risk curve, top peak hours, live hourly peak-probability curve, critical-hour alerts |
| `/api/feature-importance` | GET | Feature importances (MDI) from the hourly Random Forest |

Example:

```bash
curl "http://127.0.0.1:5000/api/day?date=2017-07-19"
```

---

## Methodology

### Peak labelling
- Peak Day - the day with the highest `Daily_Max_Load` within each calendar month, producing 62 positive cases out of 1,887 days (3.3%)
- Peak Hour - the hour with the maximum load on each calendar day, roughly 6.9% positive rate

### Models
- Random Forest (primary) - `scikit-learn` `RandomForestClassifier`, class-weighted to handle the rare positive class
  - Daily model: 400 trees, `class_weight={0: 1, 1: 15}`
  - Hourly model: 200 trees, `class_weight="balanced"`

### Features

| Feature | Level | Rationale |
|---|---|---|
| Daily_Max_Load | Daily | Direct demand ceiling |
| Max_Temperature | Daily | AC load spikes above ~295 K |
| Avg_Humidity | Daily | Heat-index amplification |
| Month | Both | Seasonal demand pattern |
| IsWeekend | Both | Commercial load drops 15-25% on weekends |
| Hour | Hourly | Dominant intra-day feature |
| Load | Hourly | Strongest hourly discriminator |

### Results

| Model | ROC-AUC | Notes |
|---|---|---|
| Daily peak-day RF | 0.907 | Recall 66.7%, Precision 34.8% on 12 true peak days in the test set |
| Hourly peak-hour RF | 0.910 | Accuracy 95.8% |

---

## Limitations

- Trained on a single transmission zone (AEP) and a single city's weather (New York); retraining is required for other regions
- The test set contains only 12 peak-day instances, so precision/recall estimates are sensitive to individual errors
- Random Forests do not explicitly model temporal autocorrelation across multiple days
- The Cost Risk Index coefficient (0.6) is a simplification of real demand-tariff structures

## Roadmap

- [ ] Add lagged load features (prior-day max, 7-day rolling mean)
- [ ] Explore SMOTE oversampling for the daily model
- [ ] Explore a Temporal Fusion Transformer for explicit sequential modelling
- [ ] Connect to a live grid telemetry API for real-time alerts
- [ ] Containerize with Docker for one-command deployment

---

## License

MIT License. Feel free to use, modify, and distribute with attribution.

## Citation

If you use this work, please cite:

```
Mayank, A. Shelke, and S. Shukla, "Peak Electricity Demand Prediction Using
Dual-Granularity Random Forest Classification: A Case Study on the AEP
Transmission Zone," Sharda University, Greater Noida, India.
```
