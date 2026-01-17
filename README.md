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

## Étape 2 — Construire les séries temporelles (le cœur du projet)

 2A. Séries “demandes produits”
	1.	Agréger les ventes en quantité par produit et par pas de temps :
	•	ex : Q[p, t] = somme des quantités vendues du produit p le jour t
	2.	Optionnel : agréger aussi le CA :
	•	Revenue[p, t] = somme(quantité × prix_unitaire)
  Livrable : deux jeux de séries temporelles :
	•	12 séries produits

## Étape 3 — EDA time series & features (cours “TimeSeries”)
	1.	Visualiser :
	•	tendance / saisonnalités (jour de semaine, mois, vacances si dispo)
	•	pics (week-end, fêtes)
	2.	Diagnostics :
	•	ACF / PACF sur quelques séries (produits et ingrédients)
	3.	Features calendaires :
	•	jour de semaine, week-end, mois

	4.	Features “lags” :
	•	D_{t-1}, D_{t-7}, moyenne mobile 7 jours, etc.

Livrable : notebook EDA + liste de features retenues.

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


