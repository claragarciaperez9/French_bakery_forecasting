# French_bakery_forecasting
Analysis and classification learning fro french bakery prices

# Les 3 datasets à utiliser (**seulement eux 3**) 
bakery_sales_top_12_cleaned (avec jour, semaine, et colonne en trop droped, mieux pour Series temp) 
ingredient_per_unit
prix_ingredient

# Global facts
- Our dataset has 234k client purchases with 7 features each. 
- January 2021 to september 2022
- **TOUT EN ANGLAIS ZEBI** 

# Objectif final :
Prévoir la quantité de chaque ingrédient qui devra être commandé EN ANALYSANT les quantitées d'ARTICLES VENDUS dans le passé

# Granularité choisie :
Par jour 


## Etape 1 reprendre EDA de Clara sur le dataset top 12 produits, pas sur les séries temps 


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
# STEP 07
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


## Étape 9 — Modèles “online / adaptatifs” (apprentissage séquentiel)

Objectif : montrer que le modèle peut s’adapter si les habitudes changent.
	1.	Régression linéaire “online” :
	•	SGDRegressor avec partial_fit
	2.	Régression pondérée (forgetting factor) :
	•	donner plus de poids aux jours récents
	3.	Variante “online” sur features identiques aux modèles régression

Livrable : backtest en mode streaming + comparaison à la version offline.

⸻

## Étape 10 — Agrégation d’experts (EWA / EG) = le gros plus “cours”
	1.	Définir tes experts :
	•	persistant, saisonnier, ARIMA, Ridge, RF, GB, online linéaire, etc.
	2.	Pour chaque date, chaque expert prédit \hat D^{(k)}_t
	3.	Mettre à jour les poids via EWA :
	•	poids \omega_{k,t} qui pénalisent les experts ayant eu une grosse erreur
	4.	Prédiction finale :
	•	\hat D_t = \sum_k \omega_{k,t}\hat D^{(k)}_t

Livrable : courbe des poids dans le temps (on voit quels experts dominent selon les périodes).

⸻

## Étape 11 — Traduire prévisions → commandes → coût
	1.	Prévoir pour chaque ingrédient \hat D[i,t] sur l’horizon (ex semaine suivante)
	2.	Convertir en quantité à commander :
	•	si tu as un stock actuel : commande = max(0, besoin - stock)
	•	sinon : commande = besoin
	3.	Calculer le coût :
	•	coût = commande × prix_fournisseur_ingrédient

Livrable : tableau final “date future / ingrédient / quantité / coût”.

⸻

## Étape 12 — Restitution & packaging
	1.	Graphs lisibles :
	•	réel vs prédit (par ingrédient majeur)
	•	coût réel vs coût prédit (si possible)
	2.	Un script reproductible :
	•	src/data.py (load + clean)
	•	src/features.py
	•	src/models.py
	•	src/backtest.py
	•	main.py (exécution complète)
	3.	Un README :
	•	comment lancer
	•	structure du projet
	•	résultats





32.6% of the articles sold are TRADITIONAL BAGUETTE, 
8,2% are CROISSANT
7% are PAIN AU CHOCOLAT
6% are COUPE
and 6% are BANETTE
In contrast,
TRADITIONAL BAGUETTE represents the 21% of the total sales (in €) followed by FORMULE SANDWICH wich represents 7.2% of the revenue even if it's the 8th article more bought, plus it's usually bought in more than one unit. BANNETTE and BAGUETTE toguether represent the 8% of the total revenue. PAIN AU CHOCOLAT and CROISSANT represent 3.5% of the revenue each. 


In general, the most bought articles are the most frquently bought. With exception: 
COMPLET is bought more frequently but in smaller quantity, 3535.0 times and in 3140 occasions
MOISSON is bought more frequently but in smaller quantity, 3362.0 times and in 3107 occasions
COOKIE is bought less frequently but in bigger quantity 3779.0 times and in 2002 occasions
ECLAIR is bought less frequently but in bigger quantity 3654.0 times and in 2006 occasions


