# WiDS Datathon 2023 – Sub-seasonal Temperature Forecasting with CatBoost

![Python](https://img.shields.io/badge/Python-3.9-blue?logo=python)
![CatBoost](https://img.shields.io/badge/Model-CatBoost-yellow)
![Kaggle](https://img.shields.io/badge/Kaggle-WiDS%20Datathon%202023-20BEFF?logo=kaggle)

## 🏆 Result

**Ranked 169 out of 697 teams worldwide** in the Stanford **Women in Data Science (WiDS) Datathon 2023**. The model forecast weather with a **CatBoost** regressor and scored an **RMSE of 0.838** on the Kaggle leaderboard.

🔗 Leaderboard: [kaggle.com/competitions/widsdatathon2023/leaderboard](https://www.kaggle.com/competitions/widsdatathon2023/leaderboard)

---

## 📌 Problem Statement

The goal of the WiDS Datathon 2023 was to improve **sub-seasonal weather forecasting**, which helps communities adapt to climate change. Participants predicted:

> **`contest-tmp2m-14d__tmp2m`** — the arithmetic mean of the maximum and minimum observed temperature over the next 14 days, for each location (`lat`, `lon`) and `startdate`.

Submissions were evaluated using **Root Mean Squared Error (RMSE)**.

---

## 📂 Dataset

| Split | Rows | Columns | Date range |
|-------|------|---------|------------|
| Train | 375,734 | 246 | 2014-09-01 → 2016-08-31 |
| Test  | 31,354  | 245 | 2022-11-01 → 2022-12-31 |

The test set has every training column except the target. The features fall into these groups:

- **Location & time**: `lat`, `lon`, `startdate`, `elevation`, `climateregions__climateregion` (the only categorical feature, with 15 Köppen climate regions)
- **NMME model forecasts**: temperature (`nmme-tmp2m-*`, `nmme0-tmp2m-*`) and precipitation (`nmme-prate-*`, `nmme0-prate-*`) forecasts from climate models such as CFSv2, CCSM, CanCM, GFDL and NASA
- **Contest observations**: 14-day aggregates of sea-level pressure, humidity, precipitation, surface pressure, potential evaporation and wind (`contest-*`)
- **Wind fields**: u/v wind and geopotential height at multiple pressure levels (`wind-uwnd-*`, `wind-vwnd-*`, `wind-hgt-*`)
- **Ocean & climate indices**: sea surface temperature (`sst-*`), sea ice concentration (`icec-*`), MJO phase/amplitude (`mjo1d__*`) and the Multivariate ENSO Index (`mei__*`)

---

## 🔍 Exploratory Data Analysis

The notebook covers:

1. **Data profiling**: automated profile reports for the train and test sets with `pandas-profiling`
2. **Missing values**: 8 columns, all from the CCSM3 / CCSM3.0 NMME models, have missing values (up to ~4.24% of rows)
3. **Target distribution**: the target is complete (no missing values), with a mean of ~11.9 °C, std ~9.9 °C, and range −20.4 °C to 37.2 °C. Plotting it over time shows a clear seasonal pattern.
4. **Categorical analysis**: frequency distribution of climate regions (`BSk`, `Cfa` and `Dfb` dominate)
5. **Outlier detection**: box plots for each feature group (NMME temperature, NMME precipitation, contest, wind and other variables)
6. **Correlation analysis**: correlation of each feature group with the target, plus a full correlation heatmap

### Key findings from the correlation analysis

| Strongest positive correlation | r | Strongest negative correlation | r |
|---|---|---|---|
| `nmme-tmp2m-56w__cfsv2` | 0.955 | `wind-hgt-10-2010-1` | −0.816 |
| `nmme-tmp2m-34w__cfsv2` | 0.954 | `wind-vwnd-925-2010-1` | −0.807 |
| `nmme-tmp2m-56w__nmmemean` | 0.951 | `wind-hgt-500-2010-1` | −0.806 |
| `nmme-tmp2m-34w__nmmemean` | 0.950 | `contest-slp-14d__slp` | −0.708 |

The NMME ensemble temperature forecasts are the most informative predictors. Geopotential height, meridional wind and sea-level pressure are strong negative signals.

---

## 🤖 Modeling Approach

1. **Preprocessing**
   - Converted `startdate` to datetime
   - Dropped rows with missing values (no imputation)
   - Removed non-model columns: `startdate`, `climateregions__climateregion`, `df_index`
2. **Train/validation split**: 60/40 split (`random_state=739`)
3. **Model**: `CatBoostRegressor` (1,000 iterations, auto learning rate ≈ 0.095, `eval_metric='RMSE'`, `random_seed=930`)
4. **Evaluation**: custom RMSE function, matching the Kaggle metric

### Performance

| Dataset | RMSE |
|---|---|
| Local train split | 0.547 |
| Local validation split | 0.573 |
| **Kaggle leaderboard** | **0.838** |

The leaderboard score is higher than the local validation score because the test period (Nov–Dec 2022) lies years after the training period (2014–2016). A random split cannot capture that temporal shift.

---

## 🛠️ Tech Stack

- **Language**: Python
- **Data handling**: pandas, NumPy
- **Visualization**: Matplotlib, Seaborn
- **Profiling**: pandas-profiling (ydata-profiling)
- **Modeling**: scikit-learn, CatBoost
- **Environment**: Jupyter Notebook / Google Colab

---

## 🚀 How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn catboost pandas-profiling
   ```
3. Download `train_data.csv` and `test_data.csv` from the [Kaggle competition page](https://www.kaggle.com/competitions/widsdatathon2023/data).
4. Open `WiDS_Datathon_Notebook.ipynb` in Jupyter or Google Colab and run the cells in order.

> **Note:** The notebook uses `google.colab.files.upload()` to load data. When running locally, replace those cells with `pd.read_csv('train_data.csv')` and `pd.read_csv('test_data.csv')`.

---

## 📁 Repository Structure

```
├── WiDS_Datathon_Notebook.ipynb   # EDA, feature analysis and CatBoost model
└── README.md                      # Project overview
```

---

## 💡 Possible Improvements

- Use a time-based validation split so local scores better reflect leaderboard performance
- Impute missing CCSM values instead of dropping rows
- Feed `climateregions__climateregion` to CatBoost as a native categorical feature
- Engineer seasonal features (day of year, month) from `startdate`
- Tune hyperparameters and ensemble with LightGBM or XGBoost

---

## 👩‍💻 Team

This project was a team effort for the WiDS Datathon 2023:

| Name | Role |
|---|---|
| **Utkarsha Khadke** | Author |
| **Sowmya Keshavaiah** | Contributor · Senior Technical Program Manager at Microsoft |
| **Lilia Georgieva, PhD** | Contributor · Product Engineering Manager, Academic |

---

## 🙏 Acknowledgements

- [Women in Data Science (WiDS) Worldwide](https://www.widsconference.org/), Stanford University
- [Kaggle WiDS Datathon 2023](https://www.kaggle.com/competitions/widsdatathon2023)
