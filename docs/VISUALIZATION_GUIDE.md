# Forecast Visualization Guide

## Two Charts Generated Daily

### 1. Forecast Chart (`forecast_YYYYMMDD.png`)
**Shows:** Next 7 days of predicted prices only

**Features:**
- Clean visualization (no historical data)
- Price values labeled on each point
- Easy to read for planning

**Example Output:**
```
Dec 30: 0.159 SEK/kWh
Dec 31: 0.145 SEK/kWh
Jan 01: 0.142 SEK/kWh
...
```

---

### 2. Comparison Chart (`predicted_vs_actual_YYYYMMDD.png`)
**Shows:** Predicted vs Actual prices over time (one line graph with two lines)

**Features:**
- **Red line (circles):** Predicted prices
- **Blue line (squares):** Actual prices
- **Metrics box:** MAE, RMSE, number of days
- **Grows over time:** More data each day

**Example:**
```
Day 1 (Dec 29): No chart yet (no actuals)
Day 2 (Dec 30): 1 point (Dec 30 predicted vs actual)
Day 7 (Jan 4):  6 points (Dec 30 - Jan 4)
```

---

## How It Works

### Daily Flow

```
Day 1 (Dec 29):
  ├─ Make forecast for Dec 30 - Jan 5
  ├─ Save predictions to tracking file
  └─ No comparison yet (need actuals)

Day 2 (Dec 30):
  ├─ Collect actual price for Dec 30
  ├─ Compare: Dec 29 prediction vs Dec 30 actual
  ├─ First comparison chart created! (1 day)
  └─ Make new forecast for Dec 31 - Jan 6

Day 3 (Dec 31):
  ├─ Collect actual for Dec 31
  ├─ Comparison chart grows (2 days now)
  └─ New forecast...

After 1 week:
  └─ Comparison chart shows 7 days of data
     Clear view of model performance!
```

---

## Files Generated

```
outputs/
├── forecast_20251229.png              # Today's 7-day forecast
├── forecast_20251229.csv              # Forecast data (CSV)
├── predicted_vs_actual_20251229.png   # Comparison (grows daily)
└── prediction_tracking.csv            # All predictions history
```

---

## Daily Usage

```bash
# 1. Collect yesterday's data
python pipelines/feature_backfill.py --mode local \
  --start-date $(date -v-1d +%Y-%m-%d) \
  --end-date $(date -v-1d +%Y-%m-%d)

# 2. Generate forecast and comparison
python pipelines/inference_pipeline.py --mode local --days 7

# 3. View charts
open outputs/forecast_$(date +%Y%m%d).png
open outputs/predicted_vs_actual_$(date +%Y%m%d).png
```

---

## What to Look For

### Forecast Chart
✅ **Lowest price day** = Best time to use electricity
✅ **Price trend** = Rising or falling over the week

### Comparison Chart
✅ **Lines close together** = Accurate predictions
✅ **Lines diverging** = Model needs improvement
✅ **MAE < 0.10** = Good performance
✅ **MAE < 0.20** = Acceptable
✅ **MAE > 0.20** = Consider retraining

---

## Demo

Check `outputs/DEMO_predicted_vs_actual.png` to see what the comparison chart looks like after a week of data!

---

## Benefits

✅ **Forecast:** Clean, simple view of next 7 days
✅ **Comparison:** Track model accuracy over time
✅ **Automatic:** Builds history automatically
✅ **Actionable:** See when to retrain model

**Pro Tip:** After 30 days, you'll have excellent visibility into model performance!
