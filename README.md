# French_bakery_forecasting
Analysis and classification learning fro french bakery prices

# IDEAS 
- Maybe create a "type" column to make analysis in terms of sandwiches, vienoisseries, pains etc and not article by article
- éliminer ou regrouper les articles qui n'apparaissent qu'une fois? Potentiellement ils ont arrÊté de se vendre???
- Créer une variable temps (semaine, jour, heure de la journée etc) pour analyses temporelles

# Global facts
- Our dataset has 234k client purchases with 7 features each. 
- January 2021 to september 2022

Articles that sold only 1 time ---> Maybe eliminate them???
119	SACHET DE VIENNOISERIE	1.0
105	PLATPREPARE6,00	1.0
22	CAKE	1.0
143	TROIS CHOCOLAT	1.0
91	PAIN NOIR	1.0
3	ARTICLE 295	1.0
112	REDUCTION SUCREES 24	1.0
45	DOUCEUR D HIVER	1.0
100	PLAT 6.50E	0.0

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


