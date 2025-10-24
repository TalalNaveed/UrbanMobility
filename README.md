
# Bike Availability vs. Weather: Exploratory Data Analysis (EDA)

**Course Midterm – Exploratory Data Analysis Project**  
**Author:** Saad Iftikhar, Talal Naveed, Ahmed Arkam Mohamed Faisaar

---

## 1) Executive Summary

This project investigates how short‑horizon weather conditions relate to Citi Bike station utilization in New York City. Using an integrated panel of hourly bike station metrics and hour‑matched weather observations, I examine whether temperature, precipitation, wind speed, and time‑of‑day patterns help explain variation in **utilization rate** (the share of docks in use). The analysis couples descriptive visualizations with a set of simple linear models and a quadratic temperature specification to probe for potential non‑linearities (e.g., comfort bands).

**Key takeaways:**

- **Temperature matters:** A 1 °C increase is associated with a **−2.78 percentage‑point** change in utilization (_p_=0.012). In this particular sample and specification, the slope is negative; with richer controls and proper non‑linear modeling across seasons this effect may flip signs (people ride more in mild weather but less in very hot/cold extremes).  
- **Wind has a weak negative relationship:** A 1‑unit rise in windspeed is associated with about **−2.26 pp** change; however, it is **not statistically significant** at 5% (_p_=0.099).  
- **Hour and weekend indicators (as specified here) did not load strongly.** The weekend and precipitation dummies are zero in this regression output because of collinearity or data transformation choices (see **Diagnostics & Limitations**).  
- **Model fit is modest (R²≈0.051):** Weather and simple timing variables alone provide limited explanatory power for station‑level utilization; station capacity, neighborhood effects, seasonality, and network dynamics likely play substantial roles.

While the overall model is intentionally simple (to emphasize EDA goals and interpretability), the workflow is reproducible, well‑documented, and designed so that additional features (e.g., station fixed effects, seasonal dummies, lagged weather, interaction terms) can be layered in future iterations.

### APIs Used (3 Total)

| **API** | **Purpose** | **Output Type** |
|----------|--------------|-----------------|
| **Open-Meteo Geocoding API** | Provides city coordinates, elevation, and population metadata. | JSON |
| **Wikidata SPARQL API** | Retrieves administrative area boundaries, land area, and OpenStreetMap relation IDs. | JSON |
| **OpenStreetMap Overpass API** | Aggregates spatial queries for total bike-lane length and metro line counts within each city’s OSM boundary. | JSON |

**API Practices:**  
- All APIs are **public and non-authenticated**.  
- Queries follow **fair-use guidelines**: ≤ 1 request / sec, with built-in timeouts and automatic retries.  
- Responses are parsed, validated, and cached locally for reproducibility.  
- Each endpoint returns lightweight JSON responses integrated into the analysis pipeline.

---

## 2) Research Question

**Primary question:**  
> _Do short‑run weather conditions (temperature, wind, rain) and time‑of‑day patterns systematically relate to Citi Bike station utilization rates in NYC?_

**Motivation:**  
- Urban mobility demand is sensitive to comfort and safety. Understanding how weather affects station utilization can inform rebalancing strategies, capacity planning, and UX design (alerts, incentives).  
- The question is tractable with public, non‑auth APIs and well‑structured open data, satisfying the project’s EDA and reproducibility objectives.

---

## 3) Data Sources & Coverage

- **Citi Bike station snapshots / usage** (hourly station‑level metrics).  
- **Weather observations** matched to each hour and location (temperature in °C, windspeed, precipitation indicator).  
- **Temporal features** derived from timestamp (hour of day, weekend flag).

> **Row count & schema:** The merged dataset used in the regression excerpt below contains **6,056 hourly observations** and **≥12 features** including numeric and categorical variables (e.g., `utilization_rate`, `temp_c`, `windspeed`, `hour`, `is_weekend`, `precip_gt0`, and capacity measures such as `total_docks`).

**Data Engineering Notes (high‑level):**
- Harmonize timestamps to a common timezone; drop or impute malformed rows.  
- Enforce numeric types; coerce and check for NaNs.  
- Construct **utilization rate** as a percentage (or pp) consistent across stations.  
- Build feature flags: `is_weekend`, `precip_gt0` (rainy hour), etc.  
- Ensure station metadata (capacity) is merged correctly to avoid collinearity with fixed effects.

---

## 4) Methods Overview

The analysis proceeds in four layers:

1. **Exploratory Visuals:**  
   - Distributions (histograms/KDEs) of utilization and weather features.  
   - Scatter plots of utilization vs. temperature (with smoothed curves), faceted by precipitation/no‑precipitation.  
   - Time‑of‑day profiles (hourly means) for weekdays vs. weekends.

2. **Feature Engineering:**  
   - Normalization/standardization where helpful for stability in fitting.  
   - Binary indicators (precipitation, weekend).  
   - Optional polynomial terms, e.g., `I(temp_c**2)`.

3. **Modeling (Illustrative):**  
   - **OLS (HC1 robust)** baseline: `utilization_rate ~ temp_c + precip_gt0 + windspeed + total_docks + hour + is_weekend`  
   - **Quadratic spec:** `utilization_rate ~ temp_c + I(temp_c**2) + precip_gt0 + windspeed + hour + is_weekend (+ capacity)`  
   - Cluster or heteroskedasticity‑consistent covariance as appropriate.

4. **Automated Summaries:**  
   - A compact Python utility converts `statsmodels` result objects into **Markdown summaries with plain‑language interpretations**, ensuring every deliverable includes both numbers and a simple verbal “so what.”

---

## 5) Core Results

Below is the exact summary generated from the notebook (N=6,056). In this run, quadratic temperature was included in the formula, but only the linear temperature coefficient appears in the printed coefficient table. The precipitation and weekend indicators show zeros due to modeling diagnostics discussed below.

> **Model overview:**  
> **N = 6,056 | R² = 0.051 (Adj. 0.051) | AIC = 58,591**  
> **Overall F-test:** F = 2.50, p = 0.174

| Variable    |   Coef.  |  Std. Err. |  P>\|z\|  |  [0.025  |  0.975]  |
|:-------------|---------:|-----------:|----------:|----------:|----------:|
| **const**       | 101.7610 |   28.7093 | 0.00039 |  45.4917 | 158.0300 |
| **temp_c**      |  -2.7822 |    1.1133 | 0.01246 |  -4.9643 |  -0.6001 |
| **windspeed**   |  -2.2629 |    1.3708 | 0.09879 |  -4.9496 |   0.4239 |
| **hour**        |  -0.0291 |    0.3022 | 0.92338 |  -0.6213 |   0.5632 |
| **is_weekend**  |   0.0000 |    0.0000 |     nan |   0.0000 |   0.0000 |
| **precip_gt0**  |   0.0000 |    0.0000 |     nan |   0.0000 |   0.0000 |


**Plain‑English interpretations:**  
- **Temperature (°C):** Each +1 °C is associated with a **−2.78 percentage‑point** change in utilization rate (95% CI −4.96 to −0.60; **significant**, _p_=0.012).  
- **Wind speed:** Each +1 unit increase is associated with a **−2.26 pp** change (95% CI −4.95 to +0.42; **not significant**, _p_=0.099).  
- **Hour of day:** Effect is essentially zero (95% CI spans zero; **not significant**).  
- **Weekend and precipitation indicators:** Returned as zeros with `nan` p‑values in this run due to collinearity or feature construction that dropped their variation (see next section).

> **Interpretation:** In this simple spec, warmer hours correspond to lower utilization. This may reflect sampling across hotter months/locations in which high temperature depresses ride propensity, or residual confounding (e.g., time‑of‑day × season interactions). With broader seasonal controls, the expected pattern is often non‑linear: usage increases from cold to mild temperatures, then decreases at extreme heat—hence the quadratic term explored. The overall R² (~0.05) suggests weather alone explains a modest fraction of station‑level variation; operational and spatial factors matter a lot.

---

## 6) Diagnostics, Limitations & Next Steps

**Why are `is_weekend` and `precip_gt0` zeros here?**  
- In this particular transformation, one of the following occurred: (1) perfect or near‑perfect collinearity with other included terms; (2) encoding produced a constant column (all 0s or all 1s after filtering); (3) the regression package dropped the column internally when using certain fixed effects or dummies; or (4) join/merge filters removed rainy rows in the analyzed subset.  
- **Action:** I re‑audit the preprocessing: check value counts, confirm non‑zero variance (`df[col].var()>0`), and ensure fixed‑effects design matrices do not subsume binary indicators.

**Model fit and omitted variables:**  
- **R²≈0.051** is expected for a minimal weather‑only model. Usage depends on **station capacity (`total_docks`)**, local demand (residential vs. commercial zones), proximity to transit, seasonality, holidays, and rebalancing operations.  
- **Next:** Add station fixed effects, monthly dummies, interaction terms (e.g., `temp × precip`), and lags (yesterday’s weather, previous hour’s utilization) to capture persistence.

**Functional form:**  
- The negative linear slope on temperature suggests a possible **inverted‑U** relationship (riders avoid very hot hours). To capture this properly, retain both `temp_c` and `I(temp_c**2)` with non‑collinear scaling (e.g., standardize temperature) and compute the turning point at **−b/(2c)**.  
- Consider splines or GAM‑style smoothers to allow flexible non‑linear fits without strong parametric assumptions.

**Sampling & external validity:**  
- The dataset reflects the specific stations/hours included after cleaning. Different months, neighborhoods, or years could exhibit different magnitudes. Robustness checks across seasons are recommended.

---

## 7) Reproducibility & How to Run

1. **Environment**
   - Python ≥ 3.10
   - Packages: `pandas`, `numpy`, `requests`, `statsmodels`, `matplotlib`/`plotly`
   - Optional for maps: `geopandas`, `shapely`

2. **Data Retrieval**
   - Run the provided scripts/notebooks to pull Citi Bike station status and weather via public endpoints.  
   - The pipeline retries requests, validates JSON, and logs missing fields.

3. **Preprocessing**
   - Convert timestamps to timezone‑aware datetimes (America/New_York).  
   - Drop rows with missing critical fields; enforce numeric types.  
   - Construct: `utilization_rate`, `is_weekend`, `hour`, `precip_gt0`, `windspeed`, `temp_c`, and capacity features.  
   - Save intermediate CSVs (e.g., `/data/clean/merged_hourly.csv`) for reuse.

4. **Analysis**
   - Open the notebook `notebooks/eda_bike_weather.ipynb`.  
   - Execute EDA cells to render figures and diagnostic tables.  
   - Fit the OLS models. Example:
     ```python
     import statsmodels.formula.api as smf
     res = smf.ols(
         "utilization_rate ~ temp_c + I(temp_c**2) + precip_gt0 + windspeed + hour + is_weekend + total_docks",
         data=df
     ).fit(cov_type="HC1")
     ```
   - Use the summarizer helper to generate the **Results + Interpretation** block:
     ```python
     md = summarize_models({"Quadratic Weather Spec": res},
                           target_label="utilization rate (pp)",
                           var_labels={"precip_gt0": "Rain (1=rainy hour)",
                                       "temp_c": "Temperature (°C)",
                                       "I(temp_c ** 2)": "Temperature²",
                                       "windspeed": "Wind speed",
                                       "hour": "Clock hour",
                                       "is_weekend": "Weekend (1=Sat/Sun)"})
     ```
   - Include resulting Markdown in the README or export to `RESULTS_SUMMARY.md`.

5. **Figures**
   - All plots include titles, axis labels with units, legible ticks, and clear legends.  
   - Key visuals:
     - Utilization vs. temperature (with smoothed fit).  
     - Utilization distributions by precip/no‑precip.  
     - Hourly profiles: weekday vs. weekend.  
     - Residual plots to check heteroskedasticity or leverage.

6. **Re‑running**
   - The project includes deterministic seeds where applicable.  
   - Data pulls are cached; re‑running the notebook reproduces the same merged dataset and figures barring API changes.

---

## 8) Discussion & Interpretation

- **Operational planning:** Even modest relationships with temperature and wind can inform **rebalancing** (anticipating surges/dips) and dynamic pricing/incentives.  
- **UX & safety:** If wind/precip meaningfully reduce usage in larger samples, operators can surface proactive guidance (route alternatives, helmets, rain covers, speed limiting during gusts).  
- **Equity & access:** Weather impacts may differ by neighborhood (exposure to shade, protected lanes). Adding spatial effects could reveal heterogeneous burdens and guide infrastructure targeting.  
- **Policy:** Results motivate richer, policy‑relevant models including holidays, school calendars, and event schedules.

---

## 9) Limitations

- **Model parsimony:** This is an intentionally small model for clarity; it omits important sources of variation (spatial fixed effects, seasonal trends).  
- **Measurement:** Station status feeds and weather matching can introduce timing mismatches or missingness; I mitigate with cleaning and robustness checks.  
- **Causal claims:** This is **observational EDA**; associations are not causal effects. Causal identification would require instruments, exogenous shocks, or experimental variation.

---

## 10) Ethical & Privacy Considerations

- All data are **public** and aggregated at station/time levels—no personally identifiable information.  
- The analysis aims to improve service reliability and safety, not to penalize riders or neighborhoods.



---

## Appendix A: Reproducible Result Block

**Results Summary & Interpretation – Quadratic Weather Spec**  
**Interpretations:****  
- **Temperature:** +1 °C → **−2.78 pp** (CI −4.96, −0.60), significant.  
- **Windspeed:** +1 unit → **−2.26 pp** (CI −4.95, +0.42), not significant.  
- **Hour:** Essentially zero; not significant.  
- **Weekend / Precip:** Appear as zeros with `nan` p‑values due to collinearity or lack of variation in this run; re‑encode in follow‑up.

---
<img width="1587" height="1189" alt="image" src="https://github.com/user-attachments/assets/5f0f7ba7-2f09-463b-b817-a10675b759dc" />
<img width="1589" height="1189" alt="image" src="https://github.com/user-attachments/assets/a3505c60-2776-4567-8796-7815aa8b4c31" />
<img width="1589" height="1189" alt="image" src="https://github.com/user-attachments/assets/40c22d5f-ab7d-4992-b5a4-d4e57c0fd66f" />
<img width="1172" height="960" alt="image" src="https://github.com/user-attachments/assets/646f9055-d18c-4867-97e0-016c152704c6" />
<img width="1590" height="1189" alt="image" src="https://github.com/user-attachments/assets/6b678411-a2af-4f65-b9ae-56a7385f8c32" />
<img width="690" height="590" alt="image" src="https://github.com/user-attachments/assets/98e5424b-2506-4a4f-aee4-beef786b4afd" />
<img width="1189" height="490" alt="image" src="https://github.com/user-attachments/assets/addf21ba-6bd9-4ef8-94b7-fdeef3a66a5a" />
