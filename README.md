# Analyse des Trajets Taxi et Exploration du Ride-Sharing à New York

## 1. Introduction
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

Le projet a été développé en **Apache Spark avec Scala**, permettant un traitement efficace de gros volumes de données.

---

## 2. Méthodologie et Pipeline Spark

### 2.1 Prétraitement des données
1. Conversion des dates et heures en timestamps standard (`timestamp_ntz`).  
2. Filtrage des trajets invalides :  
   - `trip_distance` > 0  
   - `fare_amount` ≥ 0  
   - `trip_duration` calculée en minutes à partir des timestamps.  
3. Création de nouvelles colonnes pour enrichir les analyses :  
   - `trip_type` : court (<5 km) / long (≥5 km)  
   - `average_speed` : vitesse moyenne en km/h  
   - `tip_percentage` : pourcentage de pourboire par rapport au total  
   - `hour_of_day`, `day_of_week` pour les analyses temporelles.  

### 2.2 Phase exploratoire
- Statistiques descriptives : nombre de trajets par type, mode de paiement et zone.  
- Analyse temporelle : nombre de trajets par mois et par heure.  

### 2.3 Ride-Sharing (Phase 5)
- Identification des trajets courts partageables :  
  - Même zone de départ et d’arrivée (`PULocationID`, `DOLocationID`).  
  - Départs proches dans le temps (≤10 minutes).  
- Calcul des économies potentielles :  
  - **Coût partagé** = moitié du montant total pour deux clients.  
  - **Temps économisé** : gain moyen sur la durée de trajet.  

### 2.4 Détection d’anomalies
- Utilisation des quantiles pour identifier les trajets atypiques :  
  - `trip_duration` min : 0.0167 min  
  - `trip_duration` max : 5438.8 min  
- Filtrage des outliers pour analyses fiables.  

### 2.5 Extensions
- Feature engineering avancé : `average_speed`, `tip_percentage`, fréquence de trajets par zone.  
- Catégorisation des trajets par distance, tarif ou durée.  
- Segmentation par clustering KMeans pour détecter les hotspots de demande.  
- Visualisations avancées : heatmaps, graphiques temporels, dashboards interactifs.

---

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

### 3.5 Extensions et analyses avancées

1. **Feature Engineering :**  
   - `average_speed` = distance / durée  
   - `tip_percentage` = tip / total_amount  
   - Encodage temporel : `hour_of_day`, `day_of_week`, périodes de pointe  

2. **Segmentation et clustering KMeans :**  
   - Détection des hotspots de départ et d’arrivée  
   - Analyse de la demande par zones et horaires

3. **Visualisations avancées :**  
   - Heatmaps des zones les plus fréquentées  
   - Graphiques temporels pour la demande et les revenus  
   - Dashboards interactifs avec Plotly ou Folium

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

## 5. Conclusion

- Le projet offre une **analyse complète des trajets de taxis** et met en évidence les avantages du covoiturage.  
- Les extensions proposées enrichissent les données pour des applications prédictives et décisionnelles.  
- Les analyses et visualisations peuvent guider la **gestion opérationnelle des taxis** et améliorer l’expérience client.

---

**Auteur :** Khaoula Boughattas  
**Date :** 31 Décembre 2025  
