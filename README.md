# Weather Pattern Analysis and Prediction System

**BCA 5th-semester group project by Shubham and Ritik**

## Introduction and problem statement

This project explores weather observations with data cleaning, visual analysis, clustering, and cautious machine-learning estimates. Weather records can be incomplete, noisy, and difficult to compare over time. The dashboard makes data quality visible and evaluates models on observations that occur later in time than their training data.

The supplied initial CSV is `C:\Users\Administrator\Desktop\weather.csv`. Its inspected contents have 366 rows and 22 fields, including temperature, rainfall, humidity, wind, pressure, and rain-tomorrow labels. Pandas detects 47 missing values across `Sunshine` (3), `WindGustDir` (3), `WindGustSpeed` (2), `WindDir9am` (31), `WindDir3pm` (1), and `WindSpeed9am` (7); there are no exact duplicate records. The fields have numeric, direction-category, and yes/no types. It has no date column, so it cannot be used for chronological evaluation or date-based forecasts. The dashboard never invents dates. Upload or download dated observations to use those functions.

## Objectives

- Load the provided CSV or another user-uploaded dataset.
- Retrieve historical observations from the free Open-Meteo Archive API.
- Identify missing values, duplicates, invalid measurements, and IQR outliers.
- Visualize supported measurements and explore correlations and clusters.
- Compare a seasonal-naive baseline with a Random Forest on a chronological holdout.
- Make project outputs available as CSV downloads.

## Technologies

Python, Pandas, NumPy, Scikit-learn, Plotly, Requests, and Streamlit.

## Dataset sources and attribution

- **Initial local dataset:** the user-provided `weather.csv`. The CSV contains no source, date, or unit metadata; provenance and exact units should be confirmed with its provider. Column names and values are inspected at runtime, and unavailable dates are not inferred.
- **Historical dataset:** Open-Meteo Historical Weather API, <https://open-meteo.com/en/docs/historical-weather-api>. API observations are historical observations/reanalysis, not live forecasts. Follow the current Open-Meteo attribution and terms for reuse and publication.

## Historical API use

The dashboard defaults to Patna, Bihar (25.5941, 85.1376). Search for any Indian city or settlement in Open-Meteo's geocoding catalogue, choose a match, or enter coordinates manually. The search is country-filtered to India and returns matching locations with their state/administrative area and coordinates; it is not a hand-maintained list of only major cities. Select a past date range and choose **Download historical observations**. Identical successful weather and city-search requests are cached in-process for six hours. A successful response is saved to `data/weather_data.csv` with location and coordinate columns so exported observations remain attributable to the selected place. A failed response is reported and does not create a success result. The sidebar CSV upload remains available when the service or internet is unavailable.

## Data dictionary and units

The Open-Meteo download contains:

| App field | Open-Meteo variable | Unit |
|---|---|---|
| `date` | `daily.time` | local calendar date |
| `temperature_mean` | `temperature_2m_mean` | °C |
| `temperature_max` | `temperature_2m_max` | °C |
| `temperature_min` | `temperature_2m_min` | °C |
| `precipitation` | `precipitation_sum` | mm/day |
| `wind_speed_max` | `wind_speed_10m_max` | m/s |
| `humidity_mean` | daily mean of hourly `relative_humidity_2m` | % |
| `surface_pressure_mean` | daily mean of hourly `surface_pressure` | hPa |

Open-Meteo units are explicitly requested for temperature, precipitation, and wind. Local CSV units are not assumed to be authoritative when their source provides no unit metadata. The app can derive mean temperature as the midpoint of minimum and maximum temperature and mean humidity/pressure from paired morning/afternoon readings; these derived values are labeled as derived in the app schema.

## Preprocessing

The application preserves the source columns and adds canonical weather fields where aliases are recognized. It parses and sorts dates when supplied; removes exact duplicate records; reports missing values; converts invalid numeric input to missing; flags implausible negative precipitation/wind and out-of-range humidity; median-imputes missing numeric predictors only; and does not impute target measurements. IQR outliers are displayed for inspection and are not automatically removed.

## Data mining and clustering

Descriptive statistics, correlation analysis, and monthly summaries are available where suitable columns/dates exist. Optional K-Means clustering imputes incomplete numeric predictors, scales them, and groups at least 20 records. Clusters are exploratory groups, not forecasts.

## Prediction methodology

Daily mean temperature, precipitation, and mean relative humidity are evaluated independently when the dataset has valid dates, target values, and at least 40 calendar days. Features include cyclical calendar fields and the target's prior-day, prior-week, and trailing-week measurements. Lags use only previous dates. The latest 20% of usable dated observations is held out chronologically. A median-imputed Random Forest pipeline is fit only on the training partition; the baseline is a seasonal-naive seven-day lag. Test evaluation is one-day-ahead and may use previously observed target values as they become available.

Future estimates are recursively generated for up to 14 days from date features and earlier observations/estimates. They are **model-generated estimates**, not official weather forecasts. Later estimates depend on earlier estimates, uncertainty generally increases with horizon, and historical-only models cannot reliably predict sudden storms or unusual events. The dashboard reports measured results rather than claiming accuracy in advance.

## MAE and RMSE

- **MAE (Mean Absolute Error):** the average size of the prediction error, ignoring whether the prediction was too high or low. It is expressed in the target's units.
- **RMSE (Root Mean Squared Error):** also expressed in target units, but penalizes large errors more strongly.
- Lower is better on the same target and test period. Comparing values across targets with different units is not meaningful.

## Installation and execution (Windows PowerShell)

From the project folder:

```powershell
cd "$HOME\Desktop\Student_Data\weather-pattern-analysis"
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
streamlit run app.py
```

If PowerShell blocks virtual-environment activation, use `.\.venv\Scripts\python.exe -m pip install -r requirements.txt` and `.\.venv\Scripts\python.exe -m streamlit run app.py`.

Run the focused automated checks with `python -m pytest -q`.

## Limitations and future enhancements

- The supplied CSV has no dates; chronological evaluation is correctly disabled for it.
- Local CSV units and source attribution are not embedded in the provided file.
- Open-Meteo access requires internet; CSV upload keeps local analysis usable offline.
- Estimates are constrained to historical date/lag patterns and are not an operational forecast.
- Future work could add verified local-data provenance, time-series cross-validation, uncertainty intervals, and comparison with official forecast services.

## BCA viva questions and answers

1. **What is the aim of this project?** To analyze weather records and demonstrate evaluated, date-aware estimates.
2. **What is data mining?** Finding useful patterns or relationships in data using computational methods.
3. **Why is preprocessing needed?** To identify missing, duplicated, invalid, or inconsistent observations before analysis.
4. **What is Pandas used for?** Reading, transforming, and analyzing tabular data.
5. **What is NumPy used for?** Numerical operations such as arrays and metric calculations.
6. **What is Streamlit?** A Python framework for building interactive data applications.
7. **What is Plotly used for?** Interactive charts and visual exploration.
8. **What is a Random Forest?** An ensemble of decision trees whose predictions are combined.
9. **Why compare a baseline model?** It shows whether a more complex model improves on a simple reference.
10. **Why split chronologically?** A random split can train on later observations and leak future information into evaluation.
11. **What is data leakage?** Using information during training that would not be available when making the prediction.
12. **What is a lag feature?** A previous observation used as an input for estimating a later observation.
13. **What is MAE?** The average absolute difference between actual and predicted values.
14. **What is RMSE?** The square root of average squared error, which penalizes large errors more.
15. **What does K-Means do?** It groups observations by distance to learned cluster centers.
16. **Why scale data for K-Means?** Features with larger numeric ranges could otherwise dominate distance calculations.
17. **What is an IQR outlier rule?** Values below Q1 − 1.5×IQR or above Q3 + 1.5×IQR are flagged for review.
18. **Why can this project's provided CSV not be chronologically evaluated?** It has no date column, and inventing dates would create false temporal information.
19. **Is an Open-Meteo Archive result a live forecast?** No; it provides past weather observations/reanalysis.
20. **Why can future estimates be unreliable?** Weather is nonlinear, and sudden or unusual events are not captured reliably by a historical-only model.
"# Weather-Pattern-Analysis-and-Prediction-System" 
