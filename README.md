# ELROI – Predictive Maintenance Platform

A full-stack predictive maintenance web dashboard built for an industrial startup. ELROI ingests sensor data from ESP32-based IoT hardware, runs an ML + physics ensemble to forecast cooling behaviour, and surfaces results through an interactive UI with real-time alerts and AI-driven insights.

> **Stack:** Next.js · Python · PyTorch · Supabase · Google Gemini · Resend

---

## Overview

Industrial equipment failure is costly and difficult to anticipate. ELROI continuously monitors sensor telemetry, predicts the time required for equipment to cool to a safe operational threshold, and alerts engineers before thresholds are breached — reducing unplanned downtime for the client's production line.

### Key Capabilities

- **CSV Upload & Auto-detection** — Accepts sensor exports with or without headers; auto-maps date, temperature, and pressure columns
- **Cooling-Time Prediction** — Ensemble of LSTM, XGBoost, Ridge Regression, and Newton's Law of Cooling for robust, physics-grounded forecasts
- **Production Gating** — Reliability checks and safety buffers prevent overconfident predictions from reaching operators
- **Results Visualisation** — Interactive cooling timeline, experiment-match view, and trend charts
- **AI Insights** — Optional Gemini Pro integration surfaces natural-language explanations of temperature anomalies
- **Email Alerts** — Automated threshold-breach notifications via Resend
- **Auth & Storage** — Supabase-backed authentication and session management

---

## Architecture

```
ESP32 Sensors
     │  HTTP polling / CSV export
     ▼
Next.js Dashboard  ──────────────────────────────────────┐
 ├── /api/analyze-temperature  →  Gemini Pro insights     │
 ├── /api/send-alert           →  Resend email alerts     │
 └── Supabase Auth / DB                                   │
                                                          │
Python TCN Backend  (backend/newton_prediction.py)        │
 ├── LSTM  (seed ensemble: 42, 123, 7)                    │
 ├── XGBoost                                              │
 ├── Ridge Regression                                     │
 └── Newton's Law of Cooling (physics baseline)  ◄────────┘
          │
     Cooling-time forecast + reliability score
```

---

## Repository

- **GitHub:** [https://github.com/n1hal06/Prognosis_Full_Integration.git](https://github.com/n1hal06/Prognosis_Full_Integration.git) *(private)*

---

## Web Dashboard

### Environment Setup

Create `.env.local` at the repo root:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
GEMINI_API_KEY=your_google_ai_key
RESEND_API_KEY=your_resend_key
```

| Variable | Used In | Required |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | `src/lib/supabase.js` | ✅ |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | `src/lib/supabase.js` | ✅ |
| `GEMINI_API_KEY` | `src/app/api/analyze-temperature/route.js` | Optional |
| `RESEND_API_KEY` | `src/app/api/send-alert/route.js` | Optional |

> `.env.local` is gitignored and must be created manually.

### Runtime Versions

| Tool | Version |
|---|---|
| Node.js | 20+ |
| npm | 10+ |
| Next.js | 16.0.1 |
| React | 19.2.0 |

### Run

```bash
npm install
npm run dev
```

Open: [http://localhost:3000](http://localhost:3000)

---

## AI / TCN Backend

Backend code lives under `backend/`. The main prediction script is `backend/newton_prediction.py`.

### Input Format

Accepts CSV with or without headers.

**Required columns (in order if no header):**
1. `DATE` — datetime string (e.g. `04-01-2026 10.30`)
2. `TEMPERATURE` — float, degrees Celsius
3. `PRESSURE` — float, optional

**Sample input:**

```csv
DATE,TEMPERATURE,PRESSURE
04-01-2026 10.30,89.67,-54.61
04-01-2026 10.31,88.89,-54.61
04-01-2026 10.32,87.90,-54.61
```

### Model Artifacts

The runtime expects the `PrognosisIQ_v15_outputs` directory (or a ZIP) with:

```
PrognosisIQ_v15_outputs/
├── model_config_v15.json
└── ensemble/
    ├── scalers_v15.pkl
    ├── seed_42.pt
    ├── seed_123.pt
    └── seed_7.pt
```

Place this directory (or pass the ZIP path as a CLI argument) in `backend/` before running.

### Training Data

Training scripts (`backend/training20.py`) reference the following CSV files — place them in `backend/` if retraining:

- `Report_20250604-1.csv`
- `Report_20250604-2.csv`
- `Report_20250605-1.csv`
- `Report_20250605-2.csv`
- `Report_20250611-1.csv`
- `Report_20250611-2.csv`
- `Report_20250611-3.csv`

### Python Setup

```bash
cd backend
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS / Linux
pip install -r requirements_ml.txt
pip install -r requirements2.txt
```

**Python version:** 3.10+ (tested on 3.12.2)

### CLI Usage

```bash
# Basic prediction
cd backend
venv\Scripts\python.exe newton_prediction.py uploads\your_file.csv 40 2

# With model ZIP extraction
venv\Scripts\python.exe newton_prediction.py uploads\your_file.csv 40 2 C:\path\to\PrognosisIQ_v15_outputs.zip
```

**Arguments:**

| Position | Argument | Default | Description |
|---|---|---|---|
| 1 | `csv_path` | — | Path to input CSV |
| 2 | `threshold` | `40` | Target cooling temperature (°C) |
| 3 | `band_index` | `2` | Prediction band selection |
| 4 | `zip_path` | *(optional)* | Path to model artifacts ZIP |

---

## Quick Start (Full Stack)

```bash
git clone https://github.com/rocklef/Elroi_AG_New1
cd Elroi_AG_New1

# Frontend
npm install

# Backend
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements_ml.txt
pip install -r requirements2.txt
cd ..

# Create .env.local with your keys, then:
npm run dev
```

---

## Requirement Files

| File | Purpose |
|---|---|
| `backend/requirements_ml.txt` | Primary — current ML runtime |
| `backend/requirements2.txt` | PyTorch / pandas / scikit-learn stack |
| `backend/requirements.txt` | Legacy / optional |

---

## Notes

- The pure TCN CLI predictor does not require any API keys.
- Legacy model artifacts are stored under `backend/saved_models/` for reference.
- Full app features (AI insights, email alerts) require `GEMINI_API_KEY` and `RESEND_API_KEY` respectively.
