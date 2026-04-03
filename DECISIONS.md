
# DECISIONS.md : Documentation des décisions de nettoyage et justifications


## 1. Importation & Exploration

* **Observation** : Identification de types mixtes (nombres et textes mélangés), de colonnes entièrement vides (comme `taux_interet`) et de formats de dates incohérents.
* **Justification** : Cette phase permet de cartographier les anomalies avant le traitement pour éviter les erreurs de calcul (`TypeErrors`) lors des phases d'analyse ou de modélisation.

## 2. Nettoyage des données (Data Cleaning)

* **Suppression des doublons (`transaction_id`)** :
  * **Justification** : Garantir l'unicité de chaque opération. Cela évite de compter plusieurs fois la même transaction, ce qui fausserait les calculs de flux financiers et le volume réel des activités.
* **Normalisation du format des dates** :
  * **Justification** : Convertir les dates en objets `datetime` pour permettre les analyses temporelles (Time-series) et assurer un tri chronologique correct.
* **Correction des montants (`montant` & `solde_avant`)** :
  * **Justification** : Python utilise le point `.` comme séparateur décimal. La conversion des chaînes de caractères en nombres flottants (`float`) est indispensable pour effectuer des agrégations mathématiques.
* **Uniformisation de la casse et suppression des espaces (Trim)** :
  * **Justification** : Pour le moteur de traitement, "EUR" et "eur" sont deux catégories distinctes. L'harmonisation assure l'intégrité des regroupements (`groupby`) et des jointures.
* **Traitement des valeurs manquantes** :
  * **Médiane** pour `score_credit_client` : Contrairement à la moyenne, la médiane n'est pas influencée par les valeurs extrêmes (outliers).
  * **Mode** pour les données catégorielles (`agence`, `segment_client`) : On remplace le vide par la valeur la plus fréquente pour maintenir la cohérence statistique.

## 3. Détection des Valeurs Aberrantes (Outliers)

* **Application du score IQR / Z-score** :
  * **Justification** : Isoler les montants disproportionnés qui pourraient provenir d'erreurs système ou de tentatives de fraude.
* **Création d'un flag `is_anomaly`** :
  * **Justification** : Au lieu de supprimer les données arbitrairement, le marquage permet de conserver l'historique complet tout en permettant d'exclure ces lignes lors de l'analyse finale.

## 4. Ingénierie des caractéristiques (Feature Engineering)

* **Extraction temporelle (Année, Mois, Trimestre, Jour)** :
  * **Justification** : Permettre d'analyser la saisonnalité et d'identifier les tendances de consommation selon les périodes (ex: pics d'activité en fin de mois).
* **Vérification du taux de change (`montant_eur_verifie`)** :
  * **Justification** : Étape de contrôle qualité pour valider la cohérence entre le montant brut, la devise et le montant converti en Euros présent dans le dataset.
* **Segmentation du risque (`categorie_risque`)** :
  * **Justification** : Transformer un score numérique brut en catégories métier exploitables (Low, Medium, High) pour faciliter la prise de décision.
* **Calcul du taux de rejet par agence** :
  * **Justification** : Identifier les agences présentant un taux d'échec anormalement élevé, ce qui peut révéler des problèmes techniques locaux ou des besoins d'audit.
