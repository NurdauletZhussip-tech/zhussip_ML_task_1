# MOP 3231 — Week 2 Lab / Task 1: Data Preprocessing & Cleaning

Cleaning and preparing a dataset of apartment listings in Almaty (`almaty_apartments_raw.csv`) so it can be used to train a model.

The data is synthetic, made for this course — it is not real market prices.

## Repository structure

```
.
├── data/
│   ├── almaty_apartments_raw.csv      # original data (do not touch)
│   └── almaty_apartments_clean.csv    # result of Steps 3-4, missing values still NaN
├── notebook.ipynb                     # full pipeline, steps 1-7
├── requirements.txt
└── README_en.md
```

## How to run

```
python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
jupyter notebook
```

Open `notebook.ipynb`, Restart & Run All.

My ID (last 4 digits, used as `random_state`): **2729**.

## What was done, step by step

**Step 2 — Inspect.** `area_m2`, `price_kzt`, and `rooms` load as `object` because of different thousand/decimal separators and units mixed into the text. Exact duplicates: 29. Reposts (same `listing_id`): 43.

**Step 3 — Clean.** Duplicates removed, and for reposts the row with the newest `listed_date` was kept. 48 different spellings of district names were mapped to 8 canonical names, with no leftover "other" category — every spelling is covered by an explicit rule. `rooms`, `area_m2`, and `price_kzt` were converted to numbers with no new NaN values created.

**Steps 3–4 — Cleaning log.** The notebook prints an audit table of each cleaning operation, with rows before/after and the number of changed cells. In this run it records 29 exact duplicates and 43 reposts removed, 56 sentinel values replaced, 12 impossible floors set to NaN, and 11 area corrections.

**Step 4 — Sentinels & outliers.** Found 3 sentinel columns: `floor == -1` (11 rows), `year_built == 0` (22 rows), `ceiling_height_m == 0` (23 rows) — all replaced with NaN. `floor > total_floors` is impossible (12 rows) — `floor` set to NaN. A global IQR check on `price_per_m2` finds nothing (it's hidden by price differences between districts), but checking IQR per district finds 12 rows; 11 of them clearly have a misplaced decimal point in the area (area-per-room is 8–18 times higher than normal), so they were fixed by dividing by 10. The clean CSV is saved after Steps 3–4 with missing values still NaN.

**Step 5 — Missing data.** Train/test split was done before any filling. `ceiling_height_m` is missing at different rates depending on building type (20.5% in panel buildings vs. 4.5% in monolith) → MAR → filled with the median per group. `year_built` is missing at roughly the same rate everywhere → looks like MCAR → filled with the global median. Checked with a masking experiment: the group median gives about half the error (MAE 0.0738 vs. 0.1450) compared to the global median.

**Step 6 — Scale & encode.** On the uncorrected area column, `RobustScaler` handles outliers well, while `StandardScaler` and `MinMaxScaler` do not. The scaler is fit on the training set only (fitting on train+test would leak information). The final matrix has no NaN values, and a `LinearRegression` model gets a test R² of about 0.90.

**Step 7 — EDA.** Median price per m² in Medeu is about 2.25 times higher than in Nauryzbay. The correlation between area and price is r ≈ 0.74. Median ceiling height in panel buildings is about 0.33 m lower than in brick buildings.

## Known limitations

- The imputation used for the final EDA table (`df_clean`) is not split into train/test — it was only used to make the plots, not for the model.
- `listing_id` was kept as a string (`ALM-xxxxx`), not converted to a number.
