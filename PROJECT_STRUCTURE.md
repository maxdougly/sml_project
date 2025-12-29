# Project Structure

## Final Organization

```
electricity-price-predictor/
│
├── pipelines/                 # Main Scripts (3 files)
│   ├── feature_backfill.py    # Collect historical data
│   ├── training_pipeline.py   # Train XGBoost model
│   └── inference_pipeline.py  # Generate forecasts
│
├── functions/                 # Utilities (4 files)
│   ├── storage_factory.py     # Storage abstraction (local/production)
│   ├── local_storage.py       # Local Parquet storage
│   ├── util.py                # Data collection (APIs)
│   └── electricity_data_retrieval.py  # Hopsworks utilities
│
├── tests/                     # Testing
│   └── test_data_sources.py   # API connection tests
│
├── docs/                      # Documentation
│   ├── UNIFIED_APPROACH.md    # Architecture explanation
│   └── VISUALIZATION_GUIDE.md # Chart guide
│
├── .github/workflows/         # CI/CD
│   └── electricity-price-daily.yml  # Daily automation
│
├── data/                      # Data storage (gitignored)
│   ├── processed/             # Feature-engineered Parquet files
│   └── models/                # Trained models
│
├── outputs/                   # Generated files (gitignored)
│   ├── forecast_*.png         # Forecast charts
│   ├── forecast_*.csv         # Prediction data
│   ├── predicted_vs_actual_*.png  # Comparison charts
│   └── prediction_tracking.csv    # Historical predictions
│
├── venv/                      # Virtual environment (gitignored)
│
├── .gitignore                 # Git ignore rules
├── README.md                  # Main documentation
├── requirements.txt           # Python dependencies
└── PROJECT_STRUCTURE.md       # This file
```

## File Counts

- **Python files:** 8 total
  - Pipelines: 3
  - Functions: 4
  - Tests: 1

- **Documentation:** 4 files
  - README.md
  - docs/UNIFIED_APPROACH.md
  - docs/VISUALIZATION_GUIDE.md
  - PROJECT_STRUCTURE.md

- **Config:** 2 files
  - requirements.txt
  - .gitignore

## Design Principles

1. **Unified Pipelines**: Same code works for local and production modes
2. **Mode Switching**: `--mode local` or `--mode production` flag
3. **Storage Abstraction**: Transparent switching between Parquet and Hopsworks
4. **Minimal Files**: Only essential files, no duplication
5. **Clear Organization**: Logical folder structure

## Key Features

✅ **3 main pipelines** - Data collection, training, inference
✅ **Mode selection** - Local testing or production deployment
✅ **Auto-detection** - Detects mode from environment variables
✅ **Performance tracking** - Compare predictions vs actuals over time
✅ **Clean visualizations** - Forecast + comparison charts

## Generated Files (Not in Git)

All generated files are gitignored:
- `data/` - Parquet files and models
- `outputs/` - Charts and predictions
- `venv/` - Virtual environment
- `*.pyc`, `__pycache__/` - Python cache

## Commands

### Test Everything
```bash
python tests/test_data_sources.py
```

### Run Pipeline (Local)
```bash
python pipelines/feature_backfill.py --mode local --start-date 2024-12-01
python pipelines/training_pipeline.py --mode local
python pipelines/inference_pipeline.py --mode local --days 7
```

### View Outputs
```bash
ls -lh outputs/
open outputs/forecast_$(date +%Y%m%d).png
```

---

**Total:** 5 folders, 12 code files, clean and concise!
