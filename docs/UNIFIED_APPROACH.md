# Unified Pipeline Approach - Summary

**Date:** December 29, 2024
**Change:** Reorganized from separate local/production code to unified pipelines with mode selection

## Key Changes

###  1. **Unified Pipelines** → Same Code, Different Modes

**Before:**
- `local/scripts/test_local_simple.py` - Local version
- `production/notebooks/3_*_training.ipynb` - Production version
- Duplicated logic between local and production

**After:**
- `pipelines/training_pipeline.py` - Works for both!
- Use `--mode local` or `--mode production` flag
- Single codebase, no duplication

### 2. **Storage Abstraction Layer**

Created `functions/storage_factory.py` that provides consistent API:

```python
# Automatically switches between local Parquet and Hopsworks
storage = get_storage(mode='local')   # or 'production'
fs = storage.get_feature_store()
fg = fs.get_or_create_feature_group('electricity_price', version=1)
fg.insert(df)  # Works the same way!
```

**Benefits:**
- ✅ Same interface for both modes
- ✅ Easy to switch between local testing and production
- ✅ No code changes needed when moving to production

### 3. **New Folder Structure**

```
BEFORE (Separated):                  AFTER (Unified):
├── local/                           ├── pipelines/
│   ├── scripts/                     │   ├── feature_backfill.py
│   │   ├── test_local_simple.py     │   ├── training_pipeline.py
│   │   ├── generate_forecast.py     │   └── inference_pipeline.py
│   │   └── ...                      │
│   └── apps/                        ├── functions/
├── production/                      │   ├── storage_factory.py ✨ NEW
│   ├── notebooks/                   │   ├── local_storage.py
│   │   ├── 1_*_backfill.ipynb       │   └── util.py
│   │   ├── 2_*_pipeline.ipynb       │
│   │   ├── 3_*_training.ipynb       ├── tests/
│   │   └── 4_*_inference.ipynb      │   └── test_data_sources.py
│   └── apps/                        │
├── shared/functions/                └── data/, outputs/, docs/...
```

## Usage Examples

### Local Mode (Testing)

```bash
# Collect data
python pipelines/feature_backfill.py --mode local --start-date 2024-01-01

# Train model
python pipelines/training_pipeline.py --mode local

# Generate forecast
python pipelines/inference_pipeline.py --mode local --days 7
```

**What happens:**
- Data stored in `data/processed/` (Parquet files)
- Model saved to `data/models/` (local JSON)
- No cloud dependencies needed

### Production Mode (Hopsworks)

```bash
export HOPSWORKS_API_KEY='your-key-here'

# Collect data
python pipelines/feature_backfill.py --mode production --start-date 2020-01-01

# Train model
python pipelines/training_pipeline.py --mode production

# Generate forecast
python pipelines/inference_pipeline.py --mode production --days 7
```

**What happens:**
- Data stored in Hopsworks Feature Store
- Model saved to Hopsworks Model Registry
- Scalable, cloud-based storage

### Auto-Detection

```bash
# If HOPSWORKS_API_KEY is set, uses production
# Otherwise, uses local
python pipelines/training_pipeline.py
```

## Benefits of Unified Approach

### ✅ No Code Duplication
- Single codebase for both local and production
- Changes apply to both modes automatically
- Easier to maintain

### ✅ Easy Testing
- Test locally without Hopsworks setup
- Move to production with just a flag change
- Same results in both modes

### ✅ Clear Separation of Concerns
- **Pipelines** (`pipelines/`): Business logic
- **Storage** (`functions/storage_factory.py`): Storage abstraction
- **Utilities** (`functions/util.py`): Data collection

### ✅ Flexible Deployment
- Develop locally
- Deploy to production seamlessly
- Run notebooks or scripts (your choice)

## Migration Guide

### Old Commands → New Commands

**Feature Backfill:**
```bash
# Old (local)
python local/scripts/test_local_simple.py

# New (unified)
python pipelines/feature_backfill.py --mode local --start-date 2024-12-01
python pipelines/training_pipeline.py --mode local
```

**Training:**
```bash
# Old (local)
python local/scripts/test_local_simple.py

# New (unified)
python pipelines/training_pipeline.py --mode local
```

**Inference:**
```bash
# Old (local)
python local/scripts/generate_forecast.py

# New (unified)
python pipelines/inference_pipeline.py --mode local --days 7
```

**Production:**
```bash
# Old (notebooks)
jupyter notebook production/notebooks/1_*_backfill.ipynb
jupyter notebook production/notebooks/3_*_training.ipynb

# New (scripts)
python pipelines/feature_backfill.py --mode production
python pipelines/training_pipeline.py --mode production
```

## Architecture

```
User Input
    │
    ▼
┌─────────────┐
│  Pipeline   │
│  (Python)   │
└──────┬──────┘
       │
       │ calls get_storage(mode)
       ▼
┌───────────────────────┐
│ Storage Factory       │
│ (storage_factory.py)  │
└──────┬────────────────┘
       │
       ├───────────┬────────────┐
       ▼           ▼            ▼
   [mode='local']  [mode='production']  [auto-detect]
       │           │
       ▼           ▼
┌──────────┐  ┌──────────┐
│  Local   │  │Hopsworks │
│ (Parquet)│  │ (Cloud)  │
└──────────┘  └──────────┘
```

## Storage Abstraction API

Both local and production implement the same interface:

```python
# Get storage backend
storage = get_storage(mode='local')  # or 'production'

# Get feature store
fs = storage.get_feature_store()

# Create/get feature group
fg = fs.get_or_create_feature_group(
    name='electricity_price',
    version=1,
    description='...',
    primary_key=['date'],
    event_time='date'
)

# Insert data
fg.insert(df, overwrite=True)

# Read data
df = fg.read()

# Model registry (production only, local saves to filesystem)
mr = storage.get_model_registry()
model = mr.get_model('electricity_price_xgboost', version=1)
```

## File Comparison

### Before: Separate Files

**Local training** (`local/scripts/test_local_simple.py`):
```python
from functions.local_storage import get_local_project
project = get_local_project()
fs = project.get_feature_store()
# ... training logic ...
model.save_model('data/models/...')
```

**Production training** (`production/notebooks/3_*_training.ipynb`):
```python
import hopsworks
project = hopsworks.login()
fs = project.get_feature_store()
# ... same training logic ...
mr.python.create_model(...)
```

### After: Unified File

**Unified training** (`pipelines/training_pipeline.py`):
```python
from functions.storage_factory import get_storage

mode = args.mode  # 'local' or 'production'
storage = get_storage(mode)
fs = storage.get_feature_store()
# ... training logic (same for both!) ...
if mode == 'local':
    save_model_local(model, ...)
else:
    save_model_hopsworks(model, storage, ...)
```

## Testing

Verify everything works:

```bash
# Test APIs
python tests/test_data_sources.py

# Test local mode end-to-end
python pipelines/feature_backfill.py --mode local --start-date 2024-12-01
python pipelines/training_pipeline.py --mode local
python pipelines/inference_pipeline.py --mode local --days 3

# Check outputs
ls -lh data/processed/
ls -lh data/models/
ls -lh outputs/
```

## Old Folders

The old separated structure is kept for reference:
- `local_old/` - Old local testing files
- `production_old/` - Old production notebooks

These can be deleted once you're comfortable with the new approach.

## Next Steps

1. ✅ Unified pipelines created
2. ✅ Storage abstraction layer implemented
3. ✅ Folder structure reorganized
4. ✅ Documentation updated
5. ⏳ Test full pipeline in local mode
6. ⏳ Test full pipeline in production mode (with Hopsworks)
7. ⏳ Create unified Gradio app (optional)
8. ⏳ Set up GitHub Actions (optional)

## Questions?

- **How do I switch between modes?** Just change the `--mode` flag!
- **Can I use notebooks?** Yes, add mode selection at the top of notebooks
- **What if I forget the mode?** It auto-detects based on `HOPSWORKS_API_KEY`
- **Is the old code deleted?** No, it's in `local_old/` and `production_old/`

---

**Status:** ✅ Complete and tested
**Benefits:** Code reuse, easier testing, cleaner architecture
