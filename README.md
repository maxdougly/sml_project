# ⚡ Electricity Price Predictor

ML system for predicting Swedish electricity prices (Stockholm/SE3) using weather data and XGBoost.

## 🎯 Features

- **Unified Pipelines**: One codebase for local testing and production (Hopsworks)
- **Mode Switching**: `--mode local` or `--mode production`
- **7-Day Forecasts**: Predict prices a week ahead
- **Performance Tracking**: Compare predictions vs actuals over time
- **Auto-Detection**: Detects mode from `HOPSWORKS_API_KEY` environment variable

## 📁 Structure

```
├── pipelines/           # Main pipeline scripts (3 files)
│   ├── feature_backfill.py    # Collect data
│   ├── training_pipeline.py   # Train model
│   └── inference_pipeline.py  # Generate forecasts
│
├── functions/           # Utilities (4 files)
│   ├── storage_factory.py     # Mode switching logic
│   ├── local_storage.py       # Local Parquet storage
│   ├── util.py                # Data collection (APIs)
│   └── electricity_data_retrieval.py
│
├── tests/               # Tests
│   └── test_data_sources.py
│
├── data/                # Data & models
├── outputs/             # Generated forecasts
├── docs/                # Documentation
└── requirements.txt
```

## 🚀 Quick Start

### 1. Install

```bash
pip install -r requirements.txt
```

### 2. Test APIs

```bash
python tests/test_data_sources.py
```

### 3. Run Pipeline (Local Mode)

```bash
# Collect data (last 30 days)
python pipelines/feature_backfill.py --mode local --start-date 2024-12-01

# Train model
python pipelines/training_pipeline.py --mode local

# Generate 7-day forecast
python pipelines/inference_pipeline.py --mode local --days 7
```

**Output:**
- `outputs/forecast_YYYYMMDD.png` - 7-day forecast chart
- `outputs/forecast_YYYYMMDD.csv` - Prediction data
- `outputs/predicted_vs_actual_YYYYMMDD.png` - Performance tracking (after actuals available)

### 4. Production Mode (Optional)

```bash
export HOPSWORKS_API_KEY='your-key'
python pipelines/feature_backfill.py --mode production --start-date 2020-01-01
python pipelines/training_pipeline.py --mode production
python pipelines/inference_pipeline.py --mode production --days 7
```

## 📊 Visualizations

### 1. Forecast Chart
Shows next 7 days of predicted prices:
- Clean visualization (no historical clutter)
- Price values on each point
- Updated daily

### 2. Comparison Chart
Tracks predicted vs actual prices over time:
- **Red line:** Predicted prices
- **Blue line:** Actual prices
- **Metrics:** MAE, RMSE
- **Grows daily** as you add more data

## 🔄 Daily Workflow

```bash
# Collect yesterday's data
python pipelines/feature_backfill.py --mode local \
  --start-date $(date -v-1d +%Y-%m-%d) \
  --end-date $(date -v-1d +%Y-%m-%d)

# Generate forecast
python pipelines/inference_pipeline.py --mode local --days 7
```

**Or automate with GitHub Actions** - see `.github/workflows/`

## 📈 Model Performance

Typical results with 30 days of training data:
- **R²:** ~0.93 (excellent)
- **RMSE:** ~0.12 SEK/kWh
- **MAE:** ~0.08 SEK/kWh

## 🔧 Mode Selection

### Local Mode (`--mode local`)
- ✅ No cloud setup needed
- ✅ Fast testing
- ✅ Stores in Parquet files (`data/processed/`)
- ✅ Models saved to `data/models/`

### Production Mode (`--mode production`)
- Requires Hopsworks account + API key
- Cloud Feature Store & Model Registry
- Scalable, production-ready

### Auto-Detection
If you don't specify `--mode`, it auto-detects:
- `HOPSWORKS_API_KEY` set → production mode
- Otherwise → local mode

## 🧪 Testing

```bash
# Test APIs
python tests/test_data_sources.py

# Quick test (3 days)
python pipelines/feature_backfill.py --mode local --start-date 2024-12-26
python pipelines/training_pipeline.py --mode local
python pipelines/inference_pipeline.py --mode local --days 3
```

## 🔍 Troubleshooting

### Import Errors
**Error:** `ModuleNotFoundError: No module named 'functions'`

**Fix:** Run from project root:
```bash
cd ~/electricity-price-predictor
python pipelines/training_pipeline.py --mode local
```

### API Errors
- **OpenMeteo timeout:** Rate limit (10k requests/day), wait 1 min
- **elprisetjustnu.se:** Updates 13:00 CET daily

### Model Errors
- **"Model not found":** Train first with `training_pipeline.py`
- **Low performance (R² < 0.7):** Need more historical data

### Hopsworks Errors
- **"Invalid API key":** Set `HOPSWORKS_API_KEY` environment variable
- **"Feature Group not found":** Run `feature_backfill.py` first

## 📖 Documentation

- **README.md** (this file) - Quick start
- **docs/UNIFIED_APPROACH.md** - Architecture details
- **docs/VISUALIZATION_GUIDE.md** - Chart explanations

## 📊 Data Sources

- **Weather:** [OpenMeteo API](https://open-meteo.com/) (free, no key)
- **Electricity Prices:** [elprisetjustnu.se](https://www.elprisetjustnu.se/) (free)
- **Feature Store:** [Hopsworks](https://www.hopsworks.ai/) (optional, free tier)

## 🤝 About

Educational project for MLOps course (ID2223) at KTH.

**Region:** SE3 (Stockholm, Sweden)
**Model:** XGBoost Regression
**Forecast Horizon:** 7 days ahead

---

**Last Updated:** December 2024
