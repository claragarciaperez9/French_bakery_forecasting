# French_bakery_forecasting
Analysis and classification learning fro french bakery prices

# Global facts
- Our dataset has 234k client purchases with 7 features each. 
- January 2021 to september 2022
- Final objective: Forecast the quantity of each ingredient that will need to be ordered by analyzing the quantities of items sold in the past.
- Chosen granularity: Daily.

## Step 0 — Dataset Reduction to Top 12 Articles (≈75% of Total Sales)

In this notebook, we load the original transaction dataset (`data_tmp/bakery_sales.csv`) and build a reduced version focused on the **12 best-selling articles** that together represent roughly **75% of total units sold**.

After checking that there are no missing/aberrant values, we clean the data by removing **refund-related transactions** (negative `Quantity`) along with their corresponding purchase lines (using `ticket_number` logic), and we also exclude the articles **"COUPE"** and **"FORMULE SANDWICH"**. We then compute total units sold per article, remove the least-selling products (bottom ~25% of cumulative sales), and keep the remaining “top” set, resulting in a dataset going from **146 → 12 articles** and **207,068 → 143,702 rows** (units sold **331,332 → 251,675**).

The final reduced dataset is exported to **`data_tmp/bakery_sales_top12.csv`** for use in the next notebooks.

## Step 1 — EDA on the Top 12 Articles (Popularity, Pricing, and Temporal Patterns)

In this notebook, we perform an exploratory analysis of the reduced dataset (`data_tmp/bakery_sales_top12.csv`). 

After basic cleaning and feature engineering (lowercasing column names, converting `unit_price` from strings to numeric, building a proper datetime from `date` + `time`, and creating `year/month/day_of_week/hour/is_weekend`), we compute **revenue** as `quantity × unit_price` and export a cleaned file to `data_tmp/bakery_sales_top12_cleaned.csv`.  

We then analyze the **popularity** of products using three complementary views: (1) total **quantity sold** per article, (2) **purchase frequency** (number of lines/tickets per article), and (3) total **revenue** per article — each with percentage contributions and barplots for the top 12 items, followed by a short interpretation (e.g., TRADITIONAL BAGUETTE dominates both volume and revenue). 

Finally, we check **price variability** over time via min/max unit prices per article, and we summarize **temporal sales patterns** by plotting revenue by **hour**, **day of week**, and **month** (highlighting typical peaks such as the morning rush, a strong Sunday effect, and a large July spike), alongside a few key aggregate numbers.


## Step 2 — Building Product Time Series (Daily Demand & Revenue)

In this notebook, we transform the raw transaction-level dataset (`bakery_sales_top12.csv`) into daily time series per product. 

After basic cleaning (dropping the useless index column, parsing `date`, converting `Quantity` to numeric, and converting `unit_price` from strings like `"1,05 €"` to floats), we aggregate sales **by day and by article**. We compute :
- (1) the **daily quantity sold**
- (2) the **daily revenue (turnover / CA)** as `Quantity × unit_price`.

**The outputs** are two wide-format DataFrames ready for the next notebooks: **`qty_ts`** (rows = dates, columns = the 12 products, values = daily quantities) and **`rev_ts`** (same structure, values = daily revenues). Future notebooks should import `qty_ts` and/or `rev_ts` as the main product-level time series inputs.

## Step 3 — Time Series EDA (12 products)

In this step, we performed a **time-series EDA** on the **12 product-level daily demand series** built in Step 2. 
The notebook includes: 
- (i) raw time-series plots
- (ii) **weekday seasonality** (mean demand by day of week)
- (iii) **rolling statistics** (7d/30d rolling means and rolling volatility)
- (iv) **ACF/PACF** diagnostics, and (v) **STL decomposition** (trend/seasonal/residual) to separate weekly seasonality from longer-term movements.

A **product-by-product interpretation** is provided inside the notebook, along with comments on volatility and spikes/outliers.

### **Main conclusion across the 12 products:** 
Demand is strongly **weekly seasonal** (weekends higher, Sunday usually the peak), with clear ACF/PACF signals at **lags 7, 14, 21, …**. Most products also show **non-stationarity** (level shifts/regime changes over time) and **time-varying volatility**, with occasional **spikes/outliers** (especially for pastries/desserts).  

### **Implications for forecasting:** 
Use the **seasonal naïve baseline** \(\hat{y}_t = y_{t-7}\) as a strong benchmark, prefer models that handle **weekly seasonality (m=7)** and **level changes** (e.g., SARIMA / STL+residual modeling), and keep an **outlier monitoring step**. Because demand evolves, adaptive approaches (rolling windows / forgetting factors) may help.

All generated figures are saved in **`reports/figures/`**.


## Step 4 — Evaluation Protocol

This step defines a rigorous and leakage-free evaluation framework for daily product demand forecasting.

Starting from transaction-level bakery sales data, daily demand time series were built for the 12 most sold products. A strict temporal split was applied to respect the chronological nature of the data:

- (i) Train: January 2021 → March 2022
- (ii) Validation: April 2022 → June 2022
- (iii) Test: July 2022 → September 2022

Forecasting performance is evaluated using an expanding window backtesting protocol, which reflects real operational conditions: at each forecast origin, models are trained on all past observations and used to predict future demand.

Two forecasting horizons are considered:

- (i) H = 1 day ahead (short-term operational decisions)
- (ii) H = 7 days ahead (weekly ingredient ordering)

Model accuracy is measured using MAE and RMSE, computed per product and aggregated globally. Results highlight strong weekly seasonality, significant non-stationarity over time, and large differences in forecast difficulty across products. Weekly forecasts are consistently harder than next-day forecasts, reinforcing the need for seasonality-aware models.


## Step 5 — Baseline Forecasting Models

This step establishes strong baseline models that serve as reference points for evaluating more advanced forecasting approaches.

Three baseline methods were implemented:

- (i) Naive (persistence): assumes future demand equals the most recent observation

- (ii) Seasonal naive: uses demand from the same weekday one week earlier

- (iii) 7-day moving average: averages demand over the last seven days

All baselines were evaluated using the same expanding-window backtesting protocol defined in Step 4, for both forecasting horizons (H = 1 and H = 7), on validation and test sets.

Results show that all baselines significantly outperform a dummy reference model. For next-day forecasting (H = 1), differences between baselines are small, indicating strong short-term autocorrelation in demand. For weekly forecasting (H = 7), the seasonal naive baseline clearly outperforms the others, confirming the dominant role of weekly seasonality in bakery sales.

These results define a strong performance floor: any advanced forecasting model must outperform the seasonal naive baseline, particularly for weekly forecasts, to justify its added complexity.

⸻

## Étape 6 — Modèles “time series classiques”

### Step 1: Baseline Model Implementation SARIMAX and Holt-Winters (Triple Exponential Smoothing).

SARIMAX was chosen for its ability to handle non-stationary data and complex seasonal patterns. I initially applied a parsimonious $(1,1,1)(1,1,1)_7$ configuration. 
Holt-Winters was utilized as it effectively decomposes data into level, trend, and seasonality. 
Both models were configured with a seasonal period of 7 days. This decision was directly informed by our Exploratory Data Analysis (EDA), which revealed a dominant weekly cycle where sales peaks consistently occurred on specific days (notably weekends).

### Step 2: Performance Evaluation vs. Baseline
The initial results were very encouraging. Both SARIMAX and Holt-Winters achieved significantly lower MAE (Mean Absolute Error) and RMSE (Root Mean Squared Error) scores compared to a Zero-baseline. This confirmed that the models had successfully "learned" the underlying temporal dynamics of the bakery, providing genuine predictive value for inventory management compared to a purely reactive approach.

### Step 3: Hyperparameter Optimization
To refine the forecasts further, I transitioned from a "one-size-fits-all" approach to a data-driven optimization for each specific product:For SARIMAX, I implemented the auto_arima algorithm, which explores multiple $(p, d, q)$ combinations to minimize the Akaike Information Criterion (AIC). For Holt-Winters, I optimized the seasonal component by testing both additive and multiplicative frameworks to see how fluctuations scaled with sales volume.

### Step 4: Prédiction and Analysis of Results and Model Limitations
Despite the increased complexity, the results showed that the optimized models performed slightly worse than our initial theoretical baseline. This is likely due to overfitting: the automated search tuned the parameters too closely to the "noise" of the training data, which hindered their ability to generalize to the test period.

Furthermore, we observed that in both cases, the forecasts appear to "repeat" a standard week indefinitely. This phenomenon occurs because these models rely solely on historical patterns. Since the weekly cycle is the most dominant signal in our data, and we lack external variables (such as weather, local events, or school holidays), the models statistically converge on a constant seasonal cycle as the most probable forecast.

2 csv files were created : prediction_expert_complet containing the tre train val and test predictions of both SARIMAX and Holt-Winters and 
prédiction_expert_avec_futur which contains the 7 days predictions in addition to the other predictions.
⸻
# STEP 07 Online expert aggregation
This part implements online expert aggregation.

We combine multiple expert predictions $f_{t,k}$ into an aggregate $\hat{y}_t = \sum_{k=1}^K w_{t,k} f_{t,k}$.
Weights are updated via 
- Exponentially Weighted Average (EWA): $w_{t,k} \propto \exp(-\eta L_{t-1,k})$.
- Exponentiated Gradient (EG): $w_{t+1,k} \propto w_{t,k} \exp(-\eta \nabla \ell(\hat{y}_t, y_t))$.
  
The learning rate $\eta$ is optimized using a grid search over the validation period.
Adding a Moving Average ($MA_{J-7}$) expert significantly improved the overall accuracy.
Results:
- Products with high volume, like "Traditional Baguette", naturally exhibit the largest $RMSE$.
- EWA weights effectively "track" the best-performing expert as the sales regime changes.
- The "Cookie" product favored a high $\eta=0.5$, indicating a need for rapid weight shifts.
- Aggregation outperfomrs individual models through diversification.
- Weight evolution plots show the algorithms successfully identifying the superior experts.

All generated figures are saved in **`reports/figures/`**.

# STEP 08 Predict next ingredient order
This notebook converts aggregated sales forecasts into ingredient needs, then produces a simple “next order” table.

We start from the **EG aggregated forecast** (`eg_prediction`) and build a `final_forecasts` dataframe with `(Date, product, real_sale, final_forecast)`.

Using the recipe table **`ingredients_per_unite.csv`** (quantities per unit product), we compute for each row (day × product):
- **Real ingredient consumption**: `real_<ing> = real_sale × qty_per_unit(product, ing)`
- **Predicted ingredient consumption**: `predict_<ing> = final_forecast × qty_per_unit(product, ing)`

We then **aggregate by day** (sum over all products) to obtain daily totals per ingredient.

Evaluation:
- Compute **RMSE** and **MAE** (global over all ingredients) on the **test period** `2022-07-01 → 2022-09-30`, ignoring NaN/Inf values.

Final output:
- A table `order_df` giving the **quantity to order per ingredient** as the **sum of predicted quantities over the last 7 available days**.

