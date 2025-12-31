# Analyse des Trajets Taxi et Exploration du Ride-Sharing à New York

## Description du projet
Ce projet consiste à analyser un jeu de données de trajets en taxi afin de répondre à diverses problématiques autour du comportement des passagers, des modes de paiement, et du covoiturage (ride-sharing). L’objectif est de produire des insights exploitables pour optimiser les trajets, identifier des opportunités de covoiturage et effectuer des analyses avancées.

Ce projet vise à analyser les données de trajets de taxis à New York en mettant l’accent sur les aspects suivants :  

- Compréhension des types de trajets : courts (<5 km) vs longs (≥5 km).  
- Analyse des modes de paiement et leur distribution.  
- Identification des opportunités de **covoiturage (ride-sharing)** pour des trajets courts proches dans le temps et l’espace.  
- Détection des anomalies et outliers pour fiabiliser les analyses.  
- Propositions d’extensions pour valoriser davantage les données, telles que l’analyse horaire, le clustering et la prédiction.

Les données contiennent des informations détaillées sur chaque trajet :  
- Dates et heures de prise en charge (`tpep_pickup_datetime`) et de dépose (`tpep_dropoff_datetime`).  
- Localisations de départ (`PULocationID`) et d’arrivée (`DOLocationID`).  
- Distance (`trip_distance`) et durée (`trip_duration`) du trajet.  
- Montant total payé (`total_amount`) et mode de paiement (`payment_type`).  
- Autres frais annexes (`tip_amount`, `tolls_amount`, etc.).  
Le projet est réalisé en **Scala Spark**, avec des visualisations générées en **Python** (notebook séparé) pour une exploration graphique riche.

---

## Structure du projet

taxi_project/
│
├─ notebooks/
│ ├─ analysis.scala.ipynb # Notebook Scala avec toutes les transformations et analyses
│ └─ visualization_python.ipynb # Notebook Python pour visualisations graphiques
│
├─ data/
│ ├─ taxi_data.csv 
│ ├─ yellow_tripdata_2025-11.parquet 
│
├─ output/
│ ├─ payment_by_trip.csv # Résultats des paiements par type de trajet
│ ├─ payment_over_time.csv # Paiements au fil du temps
│ ├─ amount_by_payment.csv # Montants moyens et médians par mode de paiement
│ └─ ride_sharing_analysis.csv # Résultats du covoiturage
│
├─ README.md # Ce fichier
└─ docker-compose.yaml 

---

## Phases et analyses réalisées

### **Phase 1 : Prétraitement des données**
- Nettoyage des valeurs manquantes et invalides (trip_distance > 0, fare_amount ≥ 0, tpep_pickup_datetime non nul).
- Création des colonnes utiles :
  - `trip_duration` (en minutes)
  - `average_speed` (km/h)
  - `trip_type` (`short_trip` si < 10 km, `long_trip` sinon)

---

### **Phase 2 : Analyse exploratoire**
- Statistiques descriptives des trajets : distance, durée, vitesse moyenne.
- Histogrammes et boxplots pour détecter des valeurs aberrantes.
- Quantiles calculés pour identifier les trajets très courts ou très longs.

---

### **Phase 3 : Segmentation des trajets**
- Catégorisation par distance, durée et tarif.
- Distribution par type de trajet (`short_trip` vs `long_trip`).

---

### **Phase 4 : Analyse des modes de paiement**
**Questions traitées :**
1. Comment les passagers paient-ils les trajets courts et longs ?
2. L’utilisation des modes de paiement évolue-t-elle dans le temps ?
3. Y a-t-il une relation entre le montant total et le mode de paiement ?

**Analyses réalisées :**
- Comptage des trajets par type de paiement et type de trajet (`paymentByTripTypeDF`).
- Comptage des paiements par mois (`paymentOverTimeDF`).
- Calcul de la moyenne et médiane du `total_amount` par mode de paiement (`amountByPaymentDF`).

**Visualisations générées :**
- Bar plot : nombre de trajets par type de paiement (courts vs longs)
- Line plot : évolution des paiements par mois
- Box plot : distribution du montant total par mode de paiement

**Interprétations :**
- Les trajets courts sont majoritairement payés par carte de crédit (`payment_type = 1`) alors que les trajets longs montrent une plus grande diversité.
- Le paiement par carte reste dominant sur la période analysée.
- Les montants moyens varient selon le mode de paiement, avec certains modes plus coûteux en moyenne.

---

### **Phase 5 : Exploration du covoiturage (Ride-Sharing)**
**Questions traitées :**
1. Peut-on regrouper des trajets courts ayant des départs proches dans le temps et l’espace ?
2. Quelle économie de temps ou d’argent cela pourrait-elle générer ?

**Analyses réalisées :**
- Fenêtre temporelle pour calculer l’écart entre trajets (`lag` sur `pickup_ts` par PULocationID et DOLocationID).
- Détection des trajets partageables (écart ≤ 10 minutes).
- Calcul des économies potentielles :
  - `shared_cost = total_amount / 2`
  - `money_saved = total_amount - shared_cost`
- Calcul du temps gagné si les trajets étaient combinés.

**Résultats clés :**
- **Taux de trajets courts partageables : 49.08 %**
- **Économie moyenne par trajet : 10.22$**
- **Temps moyen gagné : 12.85 minutes**
- Identification des zones avec le plus fort potentiel de covoiturage (`topRideSharingZonesDF`).

**Interprétations :**
- Presque la moitié des trajets courts peuvent être combinés pour un covoiturage.
- Les économies sont significatives tant en temps qu’en argent.
- Les zones les plus fréquentées sont les candidats prioritaires pour un service de covoiturage.

---

### **Extension : Détection d’anomalies **
- Détection des trajets anormalement longs ou courts (`durationQuantiles`).
- Calcul de la vitesse moyenne (`average_speed = trip_distance / trip_duration * 60`).
- Création de nouvelles features pour analyses prédictives :
  - `tip_percentage`
  - `jour_de_la_semaine`
  - `heure_de_la_journee`
  - `fréquence_de_trajets_par_zone`
- Utilisation des quantiles pour identifier les trajets atypiques :  
  - `trip_duration` min : 0.0167 min  
  - `trip_duration` max : 5438.8 min  
- Filtrage des outliers pour analyses fiables.

## 3. Analyses principales et résultats

### 3.1 Répartition des trajets par type et mode de paiement

| trip_type  | payment_type | count     |
|-----------|-------------|----------|
| long_trip | 0           | 108,186  |
| long_trip | 1           | 390,627  |
| long_trip | 2           | 52,286   |
| long_trip | 3           | 1,748    |
| long_trip | 4           | 7,320    |
| short_trip | 0          | 516,130  |
| short_trip | 1          | 2,238,432|
| short_trip | 2          | 295,284  |
| short_trip | 3          | 9,463    |
| short_trip | 4          | 26,061   |

**Interprétation :**  
- Les trajets courts sont majoritairement payés par carte (payment_type 1).  
- Les trajets longs présentent une distribution plus équilibrée entre espèces et cartes.  
- Cette répartition peut guider la mise en place de promotions et la sélection de trajets pour le covoiturage.

---

### 3.2 Évolution des paiements dans le temps

| year_month | payment_type | count    |
|------------|-------------|---------|
| 2025-11    | 1           | 2,629,039 |
| 2025-11    | 2           | 347,566   |
| 2025-11    | 0           | 624,316   |

**Interprétation :**  
- Le mode 1 (carte) domine largement, confirmant la tendance vers les paiements électroniques.  
- Les pics de paiement peuvent être utilisés pour optimiser le covoiturage et la répartition des taxis.

---

### 3.3 Ride-Sharing et économies

- **Trajets courts partageables** : 49.08 %  
- **Économie moyenne par trajet** : 10,22 $  
- **Temps moyen économisé** : 12,85 minutes

**Exemple :**

| total_amount | shared_cost | money_saved |
|--------------|------------|-------------|
| 22.95        | 11.475     | 11.475      |
| 24.78        | 12.39      | 12.39       |

**Interprétation :**  
- Le covoiturage permet presque de **diviser par deux le coût** pour les clients.  
- Le gain en temps est significatif pour les trajets fréquents dans les mêmes zones.  
- Les zones les plus pertinentes pour le covoiturage sont celles avec une forte densité de trajets (`PULocationID` et `DOLocationID` les plus fréquents).

---

### 3.4 Détection d’anomalies

- Trajets très courts ou très longs :  
  - Durée min : 0.0167 min  
  - Durée max : 5438.8 min  

**Interprétation :**  
- Les anomalies sont rares mais importantes pour le nettoyage des données.  
- Elles peuvent provenir d’erreurs de saisie ou de trajets exceptionnels.  

---

## 4. Recommandations

1. **Covoiturage :**  
   - Déploiement pour les trajets courts à forte densité.  
   - Réduction de coût et de temps pour les clients.  

2. **Optimisation des trajets :**  
   - Surveiller les anomalies pour améliorer la fiabilité des analyses et des tarifs.  

3. **Analyse prédictive :**  
   - Prédiction de la demande et allocation dynamique de taxis selon zones et heures.  

4. **Visualisations et dashboards :**  
   - Suivi en quasi temps réel de la demande et des revenus.  
   - Identification rapide des zones critiques pour le covoiturage.

---

## Instructions d’exécution

### Scala Notebook
1. Ouvrir `analysis.scala.ipynb` dans Databricks ou tout notebook Spark compatible.
2. Charger le dataset :  
```scala
val taxiDF = spark.read.option("header", true).csv("data/yellow_tripdata_2025-11.parquet.parquet")


Exécuter toutes les cellules pour reproduire les analyses et calculs.

Exporter les résultats souhaités en CSV pour les visualisations Python.

Python Notebook (Visualisations)

Ouvrir visualization_python.ipynb.

Installer les dépendances si nécessaire :

pip install pandas matplotlib seaborn plotly


Charger les CSV exportés depuis Scala.

Exécuter les cellules pour générer les graphiques.

Technologies et bibliothèques utilisées

Scala Spark : traitement et transformation des données

Python (Pandas, Matplotlib, Seaborn, Plotly) : visualisations et graphiques

Databricks Notebook : exécution des notebooks Scala et Python

CSV/Parquet : stockage intermédiaire des résultats

Livrables

analysis.scala.ipynb : notebook Scala complet

visualization_python.ipynb : notebook Python avec toutes les visualisations

README.md : ce fichier détaillé

CSV de sortie pour les analyses et visualisations

Conclusion

Ce projet fournit une analyse complète des trajets de taxi, identifie les modes de paiement dominants, explore le potentiel de covoiturage et met en évidence des opportunités d’optimisation de temps et de coûts. Les extensions permettent d’appliquer un feature engineering pour des analyses prédictives ou un suivi en quasi temps réel.

---

**Auteur :** Khaoula Boughattas  
**Date :** 31 Décembre 2025  
