# Deployment Guide: Hopsworks + HuggingFace Spaces + GitHub Actions

This guide walks you through deploying the complete electricity price prediction system with daily automation.

## Architecture Overview

```
┌─────────────────┐
│  GitHub Actions │  ← Runs daily at 06:00 UTC
│  (Daily Cron)   │
└────────┬────────┘
         │
         ├──> 1. Collect yesterday's data
         ├──> 2. Save to Hopsworks Feature Store
         ├──> 3. Generate 7-day forecast
         ├──> 4. Commit outputs to GitHub
         │
         v
┌─────────────────┐
│ HuggingFace     │  ← Serves Gradio UI
│ Spaces          │  ← Reads forecast from GitHub
└─────────────────┘
```

## Prerequisites

- [x] GitHub account (you have: maxdougly)
- [x] Hopsworks account (create at https://app.hopsworks.ai/)
- [ ] HuggingFace account (create at https://huggingface.co/)

---

## Part 1: Hopsworks Setup

### 1. Create Hopsworks Project

1. Go to https://app.hopsworks.ai/
2. Sign in (or create account)
3. Click **"New Project"**
4. Name: `electricity_price_predictor`
5. Click **Create**

### 2. Get API Key

1. Click your profile (top right) → **Settings**
2. Go to **API Keys** tab
3. Click **Generate New API Key**
4. **Copy the key** (save it securely!)

### 3. Configure Locally

Create `.env` file:
```bash
cd ~/electricity-price-predictor
cat > .env << EOF
HOPSWORKS_API_KEY=your-api-key-here
HOPSWORKS_PROJECT_NAME=electricity_price_predictor
EOF
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

### 5. Test Hopsworks Connection

```bash
# Small test (last 7 days)
python pipelines/feature_backfill.py --mode production --start-date 2024-12-22 --end-date 2024-12-28

# Check Hopsworks dashboard - you should see:
# - Feature Group: "electricity_prices"
```

### 6. Backfill Historical Data

```bash
# Recommended: At least 1 year of data for good model performance
python pipelines/feature_backfill.py --mode production --start-date 2024-01-01 --end-date 2024-12-28
```

**Note:** This may take 10-30 minutes depending on date range.

### 7. Train Initial Model

```bash
python pipelines/training_pipeline.py --mode production
```

Check Hopsworks dashboard → **Model Registry** → You should see `electricity_price_xgboost_model`

### 8. Generate First Forecast

```bash
python pipelines/inference_pipeline.py --mode production --days 7
```

Check `outputs/` folder for:
- `forecast_YYYYMMDD.png` ✓
- `forecast_YYYYMMDD.csv` ✓
- `prediction_tracking.csv` ✓

---

## Part 2: GitHub Actions Automation

### 1. Add Hopsworks API Key to GitHub Secrets

1. Go to your repo: https://github.com/maxdougly/sml_project
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `HOPSWORKS_API_KEY`
5. Value: Paste your Hopsworks API key
6. Click **Add secret**

### 2. Enable GitHub Actions

1. Go to **Actions** tab in your repo
2. Click **"I understand my workflows, go ahead and enable them"**
3. The workflow is already in `.github/workflows/electricity-price-daily.yml`

### 3. Test Manual Run

1. Go to **Actions** tab
2. Click **"Electricity Price Daily Pipeline"**
3. Click **"Run workflow"** dropdown
4. Select `production` mode
5. Click **"Run workflow"**
6. Wait 5-10 minutes for completion
7. Check **outputs/** folder in repo for new forecasts

### 4. Daily Automation (Automatic)

The workflow runs automatically **daily at 06:00 UTC**:
- Collects yesterday's data
- Generates new 7-day forecast
- Commits outputs to GitHub

You can see scheduled runs in **Actions** tab.

---

## Part 3: HuggingFace Spaces Deployment

### 1. Create HuggingFace Account

1. Go to https://huggingface.co/join
2. Sign up with email or GitHub
3. Verify your email

### 2. Create New Space

1. Go to https://huggingface.co/new-space
2. Fill in:
   - **Owner**: Your username
   - **Space name**: `electricity-price-predictor`
   - **License**: MIT
   - **SDK**: Gradio
   - **Hardware**: CPU (Basic, free)
   - **Visibility**: Public
3. Click **Create Space**

### 3. Link GitHub Repository to Space

HuggingFace Spaces can sync directly from GitHub:

**Option A: Git Push (Recommended)**

```bash
cd ~/electricity-price-predictor

# Add HuggingFace remote
git remote add hf https://huggingface.co/spaces/YOUR_USERNAME/electricity-price-predictor

# Push to HuggingFace
git push hf main
```

**Option B: Upload Files via Web UI**

1. In your Space, click **"Files and versions"**
2. Click **"Add file"** → **"Upload files"**
3. Upload:
   - `app.py`
   - `requirements.txt`
   - `functions/` folder
   - `outputs/` folder (with latest forecasts)

### 4. Configure Space Secrets

1. In your Space, click **Settings** → **Variables and secrets**
2. Add secrets:
   - `HOPSWORKS_API_KEY`: Your Hopsworks API key

### 5. Test Gradio App Locally First

```bash
cd ~/electricity-price-predictor
python app.py
```

Open http://localhost:7860 to test the UI.

### 6. Deploy to HuggingFace

After pushing to HuggingFace:
1. Space will build automatically (takes 2-5 minutes)
2. Check **Logs** tab for any errors
3. Once running, click **App** tab to view UI

Your app will be live at: `https://huggingface.co/spaces/YOUR_USERNAME/electricity-price-predictor`

---

## Part 4: Automatic UI Updates

To keep HuggingFace UI updated with daily forecasts, add this step to GitHub Actions:

### Update `.github/workflows/electricity-price-daily.yml`

Add after the "Commit forecast outputs" step:

```yaml
      - name: Push outputs to HuggingFace Space
        if: success()
        env:
          HF_TOKEN: ${{ secrets.HF_TOKEN }}
        run: |
          # Install HuggingFace CLI
          pip install huggingface_hub

          # Upload outputs to Space
          python -c "
          from huggingface_hub import HfApi, upload_folder
          api = HfApi()
          upload_folder(
              folder_path='outputs',
              path_in_repo='outputs',
              repo_id='YOUR_USERNAME/electricity-price-predictor',
              repo_type='space',
              token='$HF_TOKEN'
          )
          "
```

### Add HuggingFace Token to GitHub Secrets

1. Go to https://huggingface.co/settings/tokens
2. Click **New token**
3. Name: `GitHub Actions`
4. Role: **Write**
5. Copy the token
6. Add to GitHub Secrets:
   - Name: `HF_TOKEN`
   - Value: Your HuggingFace token

---

## Complete Daily Workflow

Once everything is set up:

```
06:00 UTC (Daily)
├─> GitHub Actions starts
├─> Collect yesterday's electricity + weather data
├─> Save to Hopsworks Feature Store
├─> Load model from Hopsworks Model Registry
├─> Generate 7-day forecast
├─> Create visualizations
├─> Commit outputs to GitHub
└─> Push outputs to HuggingFace Space
    └─> Gradio UI auto-refreshes with new forecast
```

---

## Monitoring & Maintenance

### Check Pipeline Health

1. **GitHub Actions**: Check for failed runs in Actions tab
2. **Hopsworks**: Monitor Feature Store for data freshness
3. **Gradio UI**: Verify "Last updated" timestamp

### Weekly Model Retraining (Optional)

Add a second workflow for weekly retraining:

```bash
# Create .github/workflows/weekly-retrain.yml
```

```yaml
name: Weekly Model Retrain

on:
  schedule:
    - cron: '0 2 * * 0'  # Sunday 02:00 UTC
  workflow_dispatch:

jobs:
  retrain:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - name: Retrain model
        env:
          HOPSWORKS_API_KEY: ${{ secrets.HOPSWORKS_API_KEY }}
        run: |
          python pipelines/training_pipeline.py --mode production
```

### Troubleshooting

**Issue**: GitHub Actions fails with "Invalid API key"
- Check: HOPSWORKS_API_KEY is correctly set in GitHub Secrets

**Issue**: Gradio UI shows "No forecast data available"
- Check: `outputs/` folder has `forecast_*.csv` files
- Run: `python pipelines/inference_pipeline.py --mode production --days 7`

**Issue**: Hopsworks connection timeout
- Check: Internet connection
- Check: Hopsworks service status at https://status.hopsworks.ai/

---

## Summary Checklist

### Hopsworks
- [ ] Account created
- [ ] API key obtained
- [ ] Historical data backfilled (1+ year)
- [ ] Model trained
- [ ] First forecast generated

### GitHub
- [ ] HOPSWORKS_API_KEY secret added
- [ ] GitHub Actions enabled
- [ ] Manual test run successful
- [ ] Daily cron scheduled

### HuggingFace
- [ ] Account created
- [ ] Space created
- [ ] Repository linked/uploaded
- [ ] App running successfully
- [ ] Auto-update configured (optional)

---

**Estimated Setup Time**: 1-2 hours (mostly waiting for data backfill)

**Recurring Costs**: $0 (all free tiers)

**Maintenance**: Fully automated after setup!
