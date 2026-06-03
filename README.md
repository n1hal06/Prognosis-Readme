# ELROI Predictive Maintenance Platform

This repository contains:
- A Next.js web dashboard (frontend + API routes)
- A Python TCN prediction backend used by the cooling/analysis flows

Repository URL:
- https://github.com/rocklef/Elroi_AG_New1

---

## Web Dashboard

### Readme
- This file is the main setup/run reference.

### GitHub repo
- https://github.com/rocklef/Elroi_AG_New1

### Environment secrets
Create `.env.local` at repo root with:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
GEMINI_API_KEY=your_google_ai_key
RESEND_API_KEY=your_resend_key
```

Where these are used:
- Supabase: `src/lib/supabase.js`
- Gemini insights API: `src/app/api/analyze-temperature/route.js`
- Email alerts API: `src/app/api/send-alert/route.js`

Notes:
- `GEMINI_API_KEY` and `RESEND_API_KEY` are optional if you do not use AI insights/email alerts.
- `.env.local` is intentionally ignored by Git.

### Additional setup guide and versions
Runtime versions in current project:
- Node.js: 20+ recommended
- npm: 10+ recommended
- Next.js: 16.0.1
- React: 19.2.0

Setup steps:
1. Clone repo
2. Install packages
3. Create `.env.local`
4. Start dev server

Commands:

```bash
npm install
npm run dev
```

### Command to run dev server

```bash
npm run dev
```

Open: http://localhost:3000

---

## AI (TCN Backend)

### Readme
- AI backend code is under `backend/`
- Main runtime script: `backend/newton_prediction.py`

### Input format and sample input
The predictor accepts CSV with either headers or no headers.

Expected required fields:
- date/time column
- temperature column
- pressure column is optional

If no header, it assumes first columns are:
1. date
2. temperature
3. pressure (optional)

Sample CSV:

```csv
DATE,TEMPERATURE,PRESSURE
04-01-2026 10.30,89.67,-54.61
04-01-2026 10.31,88.89,-54.61
04-01-2026 10.32,87.90,-54.61
```

### Training dataset link (or file)
Files referenced by training scripts (`backend/training20.py`) are:
- `Report_20250604-2.csv`
- `Report_20250605-1.csv`
- `Report_20250605-2.csv`
- `Report_20250611-1.csv`
- `Report_20250611-2.csv`
- `Report_20250611-3.csv`
- `Report_20250604-1.csv`

These files are not guaranteed to be committed in this repo. If missing, place them in `backend/`.

### Model link (or file)
Current TCN runtime in `backend/newton_prediction.py` expects artifacts from:
- `PrognosisIQ_v15_outputs` directory

or via ZIP setup argument:
- `PrognosisIQ_v15_outputs.zip`

Expected artifact files include:
- `model_config_v15.json`
- `ensemble/scalers_v15.pkl`
- `ensemble/seed_42.pt`
- `ensemble/seed_123.pt`
- `ensemble/seed_7.pt`

Legacy artifacts (older flow) also exist under:
- `backend/saved_models/`

### Requirement file
Use these requirement files for backend:
- `backend/requirements_ml.txt` (primary for current ML runtime)
- `backend/requirements2.txt` (torch/pandas/sklearn stack)

Optional/legacy:
- `backend/requirements.txt`

### Environment secrets
The pure TCN CLI predictor itself does not require external API secrets.

For full app features, secrets are still required at root `.env.local`:
- `GEMINI_API_KEY` (insights endpoint)
- `RESEND_API_KEY` (email alert endpoint)
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`

### Python version
Current local backend virtual environment was created with:
- Python 3.12.2

Recommended:
- Python 3.10+ (3.12.2 tested in this workspace)

### Additional setup guide
From repo root:

```bash
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements_ml.txt
pip install -r requirements2.txt
```

If you have a model ZIP and need extraction during run, pass ZIP path in CLI command.

### Full command to run the app
Run full web app (recommended):

```bash
npm install
npm run dev
```

Run AI predictor directly (CLI):

```bash
cd backend
venv\Scripts\python.exe newton_prediction.py uploads\your_file.csv 40 2
```

With model ZIP extraction:

```bash
cd backend
venv\Scripts\python.exe newton_prediction.py uploads\your_file.csv 40 2 C:\path\to\PrognosisIQ_v15_outputs.zip
```

CLI arguments:
1. CSV path
2. threshold temperature (use 40)
3. band index (default 2)
4. optional model ZIP path

---

## Quick Start (One Pass)

```bash
git clone https://github.com/rocklef/Elroi_AG_New1
cd Elroi_AG_New1
npm install
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements_ml.txt
pip install -r requirements2.txt
cd ..
npm run dev
```
