# Hackathon CAA - Pipeline complet de modelisation du risque

Ce repository contient un pipeline data science en 6 notebooks pour predire la charge assurance (`CHARGE`) a partir des contrats, avec un workflow qui couvre:
- exploration des donnees,
- data engineering,
- traitement des extremites (EVT),
- traitement de la censure a droite,
- feature engineering avance,
- modelisation multi-approches et export des predictions.

## Sommaire

- [1. Objectif du projet](#1-objectif-du-projet)
- [2. Perimetre et fichiers](#2-perimetre-et-fichiers)
- [3. Donnees et dimensions](#3-donnees-et-dimensions)
- [4. Pipeline des notebooks](#4-pipeline-des-notebooks)
- [5. Installation et environnement](#5-installation-et-environnement)
- [6. Execution pas a pas](#6-execution-pas-a-pas)
- [7. Artefacts produits](#7-artefacts-produits)
- [8. Resultats et modeles](#8-resultats-et-modeles)
- [9. Points d'attention](#9-points-dattention)
- [10. Troubleshooting](#10-troubleshooting)
- [11. Travaux hors perimetre principal](#11-travaux-hors-perimetre-principal)

## 1. Objectif du projet

L'objectif metier est de predire le cout attendu des contrats sur le jeu test.

La cible principale est:
- `CHARGE`

Les cibles auxiliaires exploitees dans les approches de decomposition sont:
- `FREQ`
- `CM`
- `ANNEE_ASSURANCE`

Formule utilisee dans le projet:
- `CHARGE = FREQ x CM x ANNEE_ASSURANCE`

## 2. Perimetre et fichiers

Notebooks principaux (ordre logique):
- `1-Exploration_traitement_variables.ipynb`
- `2-Data_Engineering.ipynb`
- `3-Théorie_des_valeurs_extrêmes.ipynb`
- `4-Traitement_censure_droite.ipynb`
- `5-Feature_Engineering_avancé.ipynb`
- `6-Modélisation_multi-approches.ipynb`

Repertoires utilises:
- `data/` : donnees brutes
- `new_data/` : donnees transformees intermediaires
- `old_vesrion/` : versions/experiences historiques (hors pipeline principal)

## 3. Donnees et dimensions

Jeux d'entree bruts:
- `data/train_input.csv`: 383610 lignes, 374 colonnes
- `data/train_output.csv`: 383610 lignes, 5 colonnes
- `data/test_input.csv`: 95852 lignes, 374 colonnes

Jeux intermediaires produits:
- `new_data/train_engineered.csv`: 383610 lignes, 199 colonnes
- `new_data/test_engineered.csv`: 95852 lignes, 195 colonnes
- `new_data/train_advanced_fe.csv`: 383610 lignes, 250 colonnes
- `new_data/test_advanced_fe.csv`: 95852 lignes, 246 colonnes

Colonnes cibles dans `train_output.csv`:
- `ID`
- `FREQ`
- `CM`
- `ANNEE_ASSURANCE`
- `CHARGE`

## 4. Pipeline des notebooks

### Notebook 1 - Exploration

Fichier: `1-Exploration_traitement_variables.ipynb`

Objectif:
- etablir le diagnostic global des donnees avant transformation.

Contenu principal:
- distributions des cibles,
- analyse des valeurs manquantes,
- analyse des variables categorielles et numeriques,
- correlations et redondances,
- verifications de coherence train/test.

### Notebook 2 - Data Engineering

Fichier: `2-Data_Engineering.ipynb`

Objectif:
- construire un dataset propre et exploitable.

Contenu principal:
- gestion des NA (flags + traitement),
- encodage des categorielles,
- nettoyage des variables,
- construction des premieres features,
- export des jeux engineered.

Sorties:
- `new_data/train_engineered.csv`
- `new_data/test_engineered.csv`

### Notebook 3 - Theorie des valeurs extremes (EVT)

Fichier: `3-Théorie_des_valeurs_extrêmes.ipynb`

Objectif:
- estimer un seuil de sinistres graves et caracteriser la queue de distribution.

Contenu principal:
- estimateurs de Hill/Pickands/moments,
- MRLP,
- selection du seuil,
- ajustement GPD,
- synthese EVT.

Entree principale:
- `new_data/train_engineered.csv`

### Notebook 4 - Censure a droite

Fichier: `4-Traitement_censure_droite.ipynb`

Objectif:
- traiter le plafonnement des sinistres (censure a droite, plafond 500000).

Contenu principal:
- diagnostic censure,
- MLE censee,
- imputation des censes,
- modele Tobit,
- approche Kaplan-Meier.

Entree principale:
- `new_data/train_engineered.csv`

### Notebook 5 - Feature Engineering avance

Fichier: `5-Feature_Engineering_avancé.ipynb`

Objectif:
- enrichir la representation des variables et ameliorer le signal modele.

Contenu principal:
- agregations,
- encodages avances,
- interactions,
- reduction/validation.

Sorties:
- `new_data/train_advanced_fe.csv`
- `new_data/test_advanced_fe.csv`

### Notebook 6 - Modelisation multi-approches

Fichier: `6-Modélisation_multi-approches.ipynb`

Objectif:
- comparer plusieurs familles de modeles et exporter les soumissions.

Blocs de modelisation:
- bloc A: modeles decomposes actuariels,
- bloc B: modeles directs,
- bloc C: AutoML (AutoGluon),
- bloc D: ensemble et stacking,
- approche alternative Tweedie minimaliste.

Sorties:
- fichiers `submission_*.csv` (chemin d'export configurable dans le notebook).

## 5. Installation et environnement

### Prerequis

- Python 3.10+
- `pip`
- Jupyter (`jupyter lab` ou `jupyter notebook`)

### Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirement.txt
```

Le fichier de dependances est `requirement.txt` (nom singulier dans ce repo).

## 6. Execution pas a pas

Ordre recommande:

1. Executer `1-Exploration_traitement_variables.ipynb`.
2. Executer `2-Data_Engineering.ipynb`.
3. Executer `3-Théorie_des_valeurs_extrêmes.ipynb`.
4. Executer `4-Traitement_censure_droite.ipynb`.
5. Executer `5-Feature_Engineering_avancé.ipynb`.
6. Executer `6-Modélisation_multi-approches.ipynb`.

### Important pour le notebook 6

Le notebook 6 contient des chemins absolus Databricks (`/dbfs/...`).

En execution locale, adapter au minimum:
- les fichiers d'entree FE avances vers:
  - `new_data/train_advanced_fe.csv`
  - `new_data/test_advanced_fe.csv`
- le repertoire de sortie des submissions vers un chemin local, par exemple:
  - `submissions/`

Sans cette adaptation, les cellules d'I/O du notebook 6 peuvent echouer localement.

## 7. Artefacts produits

Fichiers intermediaires:
- `new_data/train_engineered.csv`
- `new_data/test_engineered.csv`
- `new_data/train_advanced_fe.csv`
- `new_data/test_advanced_fe.csv`

Fichiers finaux:
- `submission_*.csv` generes par le notebook 6.

Controle rapide des sorties (exemple):

```bash
ls -lh new_data/
ls -lh submissions/
```

## 8. Resultats et modeles

Le notebook 6 evalue plusieurs variantes (decomposees, directes, AutoML, stacking).

Un exemple de resultat present dans les sorties du notebook:
- meilleur modele annonce: `B6 - Multi-seed Tweedie (5 seeds)`
- RMSE OOF affiche: `6784.17`

Ces valeurs dependent des parametres, du split CV, de l'environnement et des donnees effectivement chargees.

## 9. Points d'attention

- La qualite de `CHARGE` depend fortement de la gestion conjointe de `FREQ` et `CM`.
- Les donnees sont desequilibrees, avec une queue lourde sur les sinistres.
- Le traitement de la censure a droite est critique pour limiter le biais sur la severite.
- Le notebook 6 peut etre couteux en temps de calcul selon les modeles actifs.

## 10. Troubleshooting

- Erreur chemin introuvable `/dbfs/...`:
  - remplacer les chemins absolus par des chemins locaux dans le notebook 6.
- Erreurs AutoGluon/CatBoost/Torch:
  - installer les extras manquants selon les besoins du bloc AutoML.
- Erreurs OpenMP (souvent macOS + LightGBM):
  - installer les dependances systeme OpenMP adaptees a votre environnement.
- Changement de versions en cours de notebook (cellule `%pip install`):
  - redemarrer le kernel puis relancer le notebook proprement.

## 11. Travaux hors perimetre principal

Le dossier `old_vesrion/` contient des essais et versions historiques.

Pour reproduire le pipeline principal actuel, utiliser uniquement les 6 notebooks a la racine du repo.
