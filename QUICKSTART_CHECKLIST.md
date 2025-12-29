# Quick Start Checklist

Follow these steps in order to get your electricity price prediction system running with daily automation.

---

## ✅ Step 1: Hopsworks Setup (10 minutes)

### 1.1 Create Account & Get API Key
```bash
# 1. Go to: https://app.hopsworks.ai/
# 2. Sign up / Sign in
# 3. Create new project: "electricity_price_predictor"
# 4. Get API key: Profile → Settings → API Keys → Generate
```

### 1.2 Configure Locally
```bash
cd ~/electricity-price-predictor

# Create .env file with your API key
echo "HOPSWORKS_API_KEY=your-key-here" > .env
echo "HOPSWORKS_PROJECT_NAME=electricity_price_predictor" >> .env

# Install dependencies (including Hopsworks)
pip install -r requirements.txt
```

---

## ✅ Step 2: Load Data to Hopsworks (20-30 minutes)

### 2.1 Test Connection (Small Dataset)
```bash
# Test with last 7 days
python pipelines/feature_backfill.py --mode production --start-date 2024-12-22 --end-date 2024-12-28

# ✓ Check Hopsworks dashboard → Feature Groups → "electricity_prices"
```

### 2.2 Backfill Historical Data
```bash
# Load 1 year of data (minimum recommended)
python pipelines/feature_backfill.py --mode production --start-date 2024-01-01 --end-date 2024-12-28

# This takes 10-30 minutes - go get coffee! ☕
```

---

## ✅ Step 3: Train Model (5 minutes)

```bash
python pipelines/training_pipeline.py --mode production

# ✓ Check Hopsworks dashboard → Model Registry → "electricity_price_xgboost_model"
```

---

## ✅ Step 4: Generate First Forecast (2 minutes)

```bash
python pipelines/inference_pipeline.py --mode production --days 7

# ✓ Check outputs/ folder:
ls -lh outputs/
# You should see:
# - forecast_20241229.png
# - forecast_20241229.csv
# - prediction_tracking.csv
```

---

## ✅ Step 5: Test Gradio UI Locally (2 minutes)

```bash
python app.py

# Open browser: http://localhost:7860
# ✓ You should see:
#   - 7-day forecast chart
#   - Forecast data table
#   - Predicted vs Actual tab (will populate over time)
```

Press Ctrl+C to stop the app.

---

## ✅ Step 6: Enable GitHub Actions (5 minutes)

### 6.1 Add Hopsworks Secret to GitHub
```bash
# 1. Go to: https://github.com/maxdougly/sml_project/settings/secrets/actions
# 2. Click "New repository secret"
# 3. Name: HOPSWORKS_API_KEY
# 4. Value: [paste your Hopsworks API key]
# 5. Click "Add secret"
```

### 6.2 Enable Actions
```bash
# 1. Go to: https://github.com/maxdougly/sml_project/actions
# 2. Click "I understand my workflows, go ahead and enable them"
```

### 6.3 Test Manual Run
```bash
# 1. Go to Actions tab
# 2. Click "Electricity Price Daily Pipeline"
# 3. Click "Run workflow" → select "production" → "Run workflow"
# 4. Wait 5-10 minutes
# 5. ✓ Check that new files appear in outputs/ folder
```

---

## ✅ Step 7: Deploy to HuggingFace Spaces (15 minutes)

### 7.1 Create HuggingFace Account
```bash
# Go to: https://huggingface.co/join
```

### 7.2 Create New Space
```bash
# 1. Go to: https://huggingface.co/new-space
# 2. Space name: electricity-price-predictor
# 3. SDK: Gradio
# 4. Hardware: CPU (Basic, free)
# 5. Click "Create Space"
```

### 7.3 Push to HuggingFace
```bash
cd ~/electricity-price-predictor

# Add HuggingFace remote (replace YOUR_USERNAME)
git remote add hf https://huggingface.co/spaces/YOUR_USERNAME/electricity-price-predictor

# First, commit recent changes
git add .
git commit -m "Add Gradio UI and deployment configs"
git push origin main

# Push to HuggingFace
git push hf main

# ✓ Wait 2-5 minutes for build
# ✓ Visit: https://huggingface.co/spaces/YOUR_USERNAME/electricity-price-predictor
```

---

## ✅ Step 8: Auto-Update HuggingFace from GitHub Actions (10 minutes)

### 8.1 Get HuggingFace Token
```bash
# 1. Go to: https://huggingface.co/settings/tokens
# 2. Click "New token"
# 3. Name: GitHub Actions
# 4. Role: Write
# 5. Copy the token
```

### 8.2 Add Token to GitHub Secrets
```bash
# 1. Go to: https://github.com/maxdougly/sml_project/settings/secrets/actions
# 2. Click "New repository secret"
# 3. Name: HF_TOKEN
# 4. Value: [paste HuggingFace token]
# 5. Click "Add secret"
```

### 8.3 Update Workflow (I'll help you with this)
```bash
# Edit .github/workflows/electricity-price-daily.yml
# Add HuggingFace upload step
```

---

## 🎉 Done! Your System is Live

### What Happens Now?

**Every day at 06:00 UTC (automatically):**
1. 📊 GitHub Actions collects yesterday's data
2. 💾 Saves to Hopsworks Feature Store
3. 🤖 Generates new 7-day forecast
4. 📈 Creates visualization charts
5. ☁️ Pushes to HuggingFace Space
6. 🌐 Gradio UI updates automatically

---

## 📊 Monitoring Your System

### Check Health
- **GitHub Actions**: https://github.com/maxdougly/sml_project/actions
- **Hopsworks**: https://app.hopsworks.ai/
- **Gradio UI**: https://huggingface.co/spaces/YOUR_USERNAME/electricity-price-predictor

### View Logs
```bash
# GitHub Actions logs: Click on any workflow run
# HuggingFace logs: Settings → Logs
```

---

## 🆘 Troubleshooting

| Problem | Solution |
|---------|----------|
| "Module 'hopsworks' not found" | Run: `pip install -r requirements.txt` |
| "Invalid API key" | Check `.env` file has correct key |
| "Feature Group not found" | Run backfill first (Step 2) |
| "Model not found" | Run training first (Step 3) |
| Gradio shows "No data" | Run inference first (Step 4) |
| GitHub Actions fails | Check HOPSWORKS_API_KEY secret is set |

---

## 📖 Full Documentation

- **Detailed Guide**: `docs/DEPLOYMENT_GUIDE.md`
- **Architecture**: `docs/UNIFIED_APPROACH.md`
- **Visualizations**: `docs/VISUALIZATION_GUIDE.md`

---

**Total Setup Time**: ~1-2 hours (mostly waiting for data backfill)

**Recurring Cost**: $0 (all free tiers)

**Maintenance**: Fully automated! ✨
