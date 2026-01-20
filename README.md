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


## Étape 4 — Définir le protocole d’évaluation (très important)
	1.	Split temporel :
	•	Train : début → date T0
	•	Validation : T0 → T1
	•	Test : T1 → fin
	2.	Backtesting (rolling/expanding window) :
	•	tu ré-entraînes (ou mets à jour) et tu prédis la fenêtre suivante
	3.	Métriques :
	•	MAE (très lisible)
	•	RMSE (punit les gros écarts)
	•	MAPE (attention si quantités parfois 0)
	•	Option business : erreur sur coût total semaine suivante

Livrable : fonctions backtest() + tableau de scores.


## Étape 5 — Baselines (obligatoires)
	1.	Naïf persistant : \hat D_t = D_{t-1}
	2.	Saisonnier : \hat D_t = D_{t-7}
	3.	Moyenne mobile : moyenne des 7 derniers jours

Livrable : baseline report (scores + graph prévision vs réel).

⸻

## Étape 6 — Modèles “time series classiques”

Sur quelques séries “importantes” (top ingrédients ou top produits) :
	1.	AR / ARMA / ARIMA / SARIMA (si saisonnalité hebdo)
	2.	Lissage exponentiel / Holt-Winters (si tu l’implémentes)

Tu peux faire :
	•	soit un modèle par produit puis conversion.

Livrable : comparaison vs baselines.

⸻



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


