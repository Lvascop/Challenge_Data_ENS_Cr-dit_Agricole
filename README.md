# Challenge_Data_ENS_Credit_Agricole

# Hackathon CAA - Pipeline complet de modélisation du risque

Ce projet contient un pipeline en 6 notebooks, de l'exploration initiale des variables jusqu'a la modelisation finale des cibles assurance.

## Structure generale

Le workflow suit la logique suivante:

1. Exploration et diagnostic des variables brutes.
2. Data engineering et nettoyage.
3. Traitement des valeurs extremes (EVT).
4. Traitement de la censure a droite (plafond).
5. Feature engineering avance.
6. Modelisation et prediction finale.

## Description des notebooks

## 1) `1-Exploration_traitement_variables.ipynb`

### Objectif
Explorer les variables d'entree (numeriques et categorielles), comprendre la qualite des donnees et identifier les premiers signaux utiles pour les cibles.

### Contenu principal
- Chargement et fusion des jeux train/test et des sorties train.
- Profilage des colonnes: types, cardinalite, valeurs manquantes.
- Analyse des distributions et asymetries.
- Selection initiale des variables via Mutual Information et tests statistiques.
- Visualisations exploratoires (histogrammes, boxplots, heatmaps).

### Sorties attendues
- Tableau de synthese des variables.
- Liste de variables potentiellement informatives.
- Diagnostics de manquants par blocs metier.

## 2) `2-Data_Engineering.ipynb`

### Objectif
Construire une base propre et exploitable pour l'apprentissage en traitant les NA, les variables peu informatives et l'encodage.

### Contenu principal
- Strategie de gestion des valeurs manquantes (imputations + flags de presence).
- Suppression des colonnes trop vides et quasi constantes.
- Encodage des variables categorielles.
- Detection et reduction des redondances (fortes correlations).
- Controle de coherence entre train et test.

### Sorties attendues
- `new_data/train_engineered.csv`
- `new_data/test_engineered.csv`
- Journal des colonnes conservees/supprimees.

## 3) `3-Theorie_des_valeurs_extremes.ipynb`

### Objectif
Modeliser la queue de distribution des sinistres severes avec la theorie des valeurs extremes (EVT).

### Contenu principal
- Statistiques descriptives de severite.
- Choix de seuil (Mean Residual Life, analyses de stabilite).
- Ajustement de lois de type Generalized Pareto.
- Validation d'ajustement (graphiques diagnostiques, KS test).

### Sorties attendues
- Parametres de loi de queue.
- Seuil de severite retenu.
- Graphiques CDF/PDF/Q-Q pour validation.

## 4) `4-Traitement_censure_droite.ipynb`

### Objectif
Corriger le biais induit par la censure a droite (plafond des montants) et estimer des charges non observees.

### Contenu principal
- Identification des observations censurees.
- Comparaison censure vs non censure.
- Approches de survie (Kaplan-Meier) et modeles parametriques.
- Estimation par maximum de vraisemblance.
- Imputation des montants censes au-dela du plafond.

### Sorties attendues
- Serie de montants corriges.
- Comparaison de modeles (qualite d'ajustement).
- Variables de cout corrigees pour la suite du pipeline.

## 5) `5-Feature_Engineering_avance.ipynb`

### Objectif
Enrichir la representation des donnees pour augmenter la performance predictive.

### Contenu principal
- Creation d'interactions et transformations non lineaires.
- Reduction de dimension (PCA selon blocs).
- Selection de variables par importances (RF/permutation).
- Adversarial validation pour detecter un drift train/test.

### Sorties attendues
- Features enrichies et selectionnees.
- Classements d'importance des variables.
- Diagnostics de robustesse des nouvelles features.

## 6) `6-Modelisation.ipynb`

### Objectif
Entrainer et evaluer les modeles finaux pour les cibles FREQ, CM, CHARGE et IS_PLAFONNE.

### Contenu principal
- Pipelines de modelisation par cible.
- Validation croisee et tuning d'hyperparametres (Optuna).
- Modeles de gradient boosting et approches ensemble.
- Evaluation via metriques regression/classification.

### Sorties attendues
- Predicteurs finaux entraines.
- Scores de validation.
- Fichiers de predictions test.

## Donnees du projet

- `data/train_input.csv`: variables d'entree train.
- `data/train_output.csv`: cibles train.
- `data/test_input.csv`: variables d'entree test.
- `new_data/`: jeux transformes produits pendant le pipeline.

## Installation

1. Creer un environnement Python 3.10+.
2. Installer les dependances:

```bash
pip install -r requirement.txt
```

## Notes de compatibilite

- Si un notebook utilise `np.trapz`, preferer `scipy.integrate.trapezoid` pour rester compatible avec les versions recentes de l'ecosysteme scientifique.
- Sur macOS, certaines stacks (LightGBM, OpenMP) peuvent necessiter des dependances systeme supplementaires.

