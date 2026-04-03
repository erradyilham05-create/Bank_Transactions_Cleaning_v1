# Rapport de Nettoyage et de Préparation des Données

## 1. Introduction
Ce rapport documente le processus de nettoyage (Data Cleaning) et d'enrichissement (Feature Engineering) appliqué au jeu de données des transactions bancaires. L'objectif est de garantir la fiabilité des données pour une analyse financière précise et la détection d'éventuelles anomalies.

## 2. Synthèse de l'Exploration Initiale
* **Volume :** 2060 transactions et 16 colonnes.
* **Qualité des données :**
    * Présence de types mixtes dans les colonnes `agence` et `segment_client`.
    * La colonne `taux_interet` était vide à 100%.
    * Taux de valeurs manquantes significatif sur le `score_credit_client` (8.1%).

## 3. Méthodologie de Nettoyage (Data Cleaning)

### A. Intégrité et Unicité
* **Suppression des doublons :** Les transactions basées sur un `transaction_id` identique ont été supprimées pour éviter toute surestimation des volumes financiers.
* **Normalisation textuelle :** Suppression des espaces parasites (`trim`) et uniformisation de la casse (ex: `PREMIUM` → `Premium`).

### B. Rectification des Types et Formats
* **Dates :** Conversion de la colonne `date_transaction` au format standard `AAAA-MM-JJ HH:MM:SS` pour permettre les analyses temporelles.
* **Montants :** Remplacement des virgules par des points et suppression des suffixes textuels (" EUR") dans la colonne `solde_avant`, suivis d'une conversion en type `float`.

### C. Traitement des Données Manquantes
* **Variables Numériques :** Imputation par la **médiane** pour le `score_credit_client` afin de ne pas subir l'influence des valeurs extrêmes.
* **Variables Catégorielles :** Imputation par le **mode** pour les colonnes `agence` et `segment_client`.
* **Suppression :** La colonne `taux_interet` a été définitivement supprimée car inexploitable.

## 4. Analyse des Valeurs Aberrantes (Outliers)
Nous avons utilisé la méthode **IQR (Interquartile Range)** sur la colonne `montant` :
* **Action :** Les lignes détectées comme aberrantes n'ont pas été supprimées mais marquées via une colonne `is_anomaly`. 
* **Objectif :** Permettre un audit spécifique sur ces transactions (risques de fraude ou erreurs de saisie) sans perdre d'information historique.

## 5. Enrichissement des Données (Feature Engineering)
Pour améliorer la puissance d'analyse, les variables suivantes ont été créées :
* **Variables Temporelles :** Extraction de l'année, du mois, du trimestre et du jour de la semaine.
* **Vérification de Change :** Création de `montant_eur_verifie` pour s'assurer de la cohérence avec les taux de change fournis.
* **Segmentation métier :**
    * `categorie_risque` : Classification automatique (Low, Medium, High) basée sur le score de crédit.
    * `taux_rejet_agence` : Calcul de la proportion de transactions rejetées par point de vente pour identifier des anomalies opérationnelles locales.

## 6. Conclusion
Le dataset est désormais **propre, normalisé et enrichi**. Il est prêt pour des étapes de visualisation (Dashboard Streamlit) ou pour l'entraînement de modèles de détection de fraude. La cohérence des montants et la gestion rigoureuse des valeurs manquantes assurent la robustesse des futures conclusions analytiques.
