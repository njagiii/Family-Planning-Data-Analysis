# 📊 Family Planning Data Analysis

End-to-end Python analysis pipeline for historical family planning (FP) commodity data. Cleans raw HMIS data, explores county-level variability, checks consumption-service alignment, and produces 12-month demand forecasts for three focal products.


## ⚙️ Requirements

```bash
pip install pandas numpy scipy matplotlib seaborn scikit-learn openpyxl
```

| Package | Purpose |
|---|---|
| `pandas` | Data loading, wrangling, aggregation |
| `numpy` | Array math |
| `scipy` | Parameter optimisation for Holt-Winters ETS |
| `matplotlib` / `seaborn` | Charts |
| `scikit-learn` | MAE / RMSE evaluation helpers |
| `openpyxl` | Reading `.xlsx` input file |

---

## 🚀 Usage

1. Update the file path on line 49 of `analysis.py` to point to your raw Excel file:

```python
FILE = "path/to/your/FP-historical-data.xlsx"
```

2. Run the script:

```bash
python analysis.py
```

---

## 📦 Input Data

The script expects a raw `.xlsx` file with at least these columns:

| Column | Description |
|---|---|
| `period` | Date string in `DD-Month/YYYY` format |
| `county` | County name or code |
| `product` | FP commodity name (e.g. `COCs`, `DMPA-SC`, `Jadelle`) |
| `method` | Either `Consumption` or `Service` |
| `value` | Numeric quantity |

---

## 🔬 Analysis Pipeline

### Block 2 — Data Cleaning
Four issues are addressed before any analysis:

- **Duplicates** — identical rows dropped (keep first)
- **Negatives** — data-entry errors converted to `NaN`
- **Outliers** — Winsorised at Q3 + 4×IQR per (product, method) group
- **Missing values** — filled with county-product-method median; then forward-filled; then zero-filled as a last resort

### Block 3 — National Aggregation
All 47 counties are summed to produce a national monthly figure for each (product, method) pair.

### Block 4 — Focal Product Selection
Three products are selected for deep analysis:

| Product | Reason for Selection |
|---|---|
| **COCs** | Highest absolute national consumption — disruption would affect the most users |
| **DMPA-SC** | Fastest year-on-year growth — self-injectable with community health worker scale-up |
| **Jadelle** | Highest median county-level coefficient of variation — implant with high demand volatility |

### Block 5 — County-Level Variability (CV)
Coefficient of variation (CV = std / mean) is computed per county-product pair.

- **CV < 0.3** → predictable; standard monthly replenishment is adequate
- **CV > 0.8** → volatile; safety stock buffers and frequent cycle counts recommended

→ **Output:** `chart1_county_variability.png` (heatmap)

### Block 6 — Consumption vs Service Alignment
Pearson correlation (r) between monthly consumption and service uptake per product.

- **r ≈ 1.0** → tight alignment; healthy data quality signal
- **r < 0.7** → divergence; possible stockouts, parallel procurement, or HMIS recording gaps

→ **Output:** `chart2_consumption_vs_service.png`

### Blocks 7–8 — Forecasting (12-Month Horizon)

Two models are fitted and compared per product using an 80/20 train/test split:

**Model 1 — Holt-Winters Additive ETS**
Fits three smoothing parameters optimised via Nelder-Mead (scipy):
- `α` (alpha) — level: how quickly the model adapts to new level changes
- `β` (beta) — trend: sensitivity to trend direction changes
- `γ` (gamma) — seasonal: weight given to updating seasonal factors

Best suited for: series with consistent trend and stable seasonality.

**Model 2 — Trend + Seasonal-Index Decomposition (MA Baseline)**
A transparent 4-step decomposition:
1. 12-month centred moving average to isolate trend
2. Seasonal indices = average (observation / moving-average) per calendar month
3. OLS linear regression extrapolates the trend forward
4. Forecast = projected trend × seasonal index

Best suited for: communicating methodology to non-technical stakeholders.

**Model selection:** The model with the lowest MAPE (%) on the test set is used for the final 12-month projection.

→ **Output:** `chart3_forecasts.png`, `metrics_summary.csv`

### Block 9 — Metrics Summary
Consolidates MAE, RMSE, and MAPE for both models across all three focal products into a single CSV.

### Block 10 — 12-Month Forecast Table
Produces a forward projection using the best-performing model per product.

→ **Output:** `twelve_month_forecast.csv`

### Block 11 — Top Counties Bar Chart
Horizontal bar chart of the 10 counties with highest total consumption per focal product over the full study period.

→ **Output:** `chart4_top_counties.png`

### Block 12 — Year-over-Year Growth
Annual YoY % change in national consumption per focal product. Partial years are excluded (2024 is excluded as data runs Jan–Sep only).

→ **Output:** `chart5_yoy_growth.png`

---

## 📤 Outputs Summary

| File | Description |
|---|---|
| `chart1_county_variability.png` | CV heatmap — all counties × 3 focal products |
| `chart2_consumption_vs_service.png` | Consumption vs service uptake time series |
| `chart3_forecasts.png` | ETS and MA forecasts with train/test split |
| `chart4_top_counties.png` | Top 10 counties by total consumption |
| `chart5_yoy_growth.png` | Year-over-year growth trends |
| `metrics_summary.csv` | MAE, RMSE, MAPE for both models per product |
| `twelve_month_forecast.csv` | Monthly forecast units for each focal product |

---

## 📌 Notes & TODOs

- [ ] Add a `requirements.txt`
- [ ] Partial-year handling for 2024 (currently excluded entirely from YoY chart)
