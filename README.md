# Hackathon CAA - Prédiction de la charge assurance

Ce repository contient un pipeline data science complet pour prédire `CHARGE` à partir de données contrat/sinistre.

Relation métier utilisée dans l'ensemble du projet :

`CHARGE = FREQ x CM x ANNEE_ASSURANCE`

## Objectif

- prédire la charge attendue sur le jeu test ;
- traiter explicitement les difficultés actuarielles du dataset (queue lourde, censure à droite, déséquilibre) ;
- comparer plusieurs familles de modèles puis générer des fichiers de soumission.

## Données

| Fichier | Rôle | Lignes | Colonnes |
|---|---|---:|---:|
| `data/train_input.csv` | Variables explicatives (train) | 383610 | 374 |
| `data/train_output.csv` | Cibles train (`FREQ`, `CM`, `CHARGE`, `ANNEE_ASSURANCE`) | 383610 | 5 |
| `data/test_input.csv` | Variables explicatives (test) | 95852 | 374 |
| `new_data/train_engineered.csv` | Dataset intermédiaire après data engineering | 383610 | 199 |
| `new_data/test_engineered.csv` | Dataset intermédiaire test | 95852 | 195 |
| `new_data/train_advanced_fe.csv` | Dataset final enrichi pour modélisation | 383610 | 250 |
| `new_data/test_advanced_fe.csv` | Dataset final enrichi test | 95852 | 246 |

## Résumé de tous les notebooks

### Pipeline principal (racine)

**`1-Exploration_traitement_variables.ipynb`**  
Objectif : établir un diagnostic complet des données avant toute transformation.  
Ce notebook couvre l'exploration des cibles, l'analyse des valeurs manquantes, la lecture des variables catégorielles et numériques, les corrélations, la cohérence entre train et test, ainsi que l'identification du plafond à `500000`.  
Entrées principales : `data/train_input.csv`, `data/train_output.csv`, `data/test_input.csv`.  
Sortie : analyse EDA (pas d'export structurant).

**`2-Data_Engineering.ipynb`**  
Objectif : construire un dataset propre, cohérent et exploitable pour la modélisation.  
Le notebook met en place les flags de NA, les encodages, le nettoyage des variables (constantes, corrélations élevées), l'alignement train/test, les premières features métier et les corrections de censure.  
Entrées principales : `data/train_input.csv`, `data/train_output.csv`, `data/test_input.csv`.  
Sorties : `new_data/train_engineered.csv` et `new_data/test_engineered.csv`.

**`3-Théorie_des_valeurs_extrêmes.ipynb`**  
Objectif : caractériser la queue de distribution des sinistres et fixer un seuil de gravité robuste.  
Le notebook applique les outils EVT (Hill, Pickands, moments, mean residual life, sélection de seuil, ajustement GPD avec variantes censurées) pour documenter le comportement des extrêmes.  
Entrée principale : `new_data/train_engineered.csv`.  
Sortie : diagnostic EVT et paramétrage des sinistres graves (sans export CSV dédié).

**`4-Traitement_censure_droite.ipynb`**  
Objectif : corriger le biais induit par la censure à droite (plafond de sinistre).  
Le notebook compare plusieurs approches : maximum de vraisemblance censuré (lognormale/gamma), imputation ponctuelle et multiple, modèle Tobit et méthode Kaplan-Meier.  
Entrée principale : `new_data/train_engineered.csv`.  
Sortie : méthodes de correction de censure et éléments de comparaison.

**`5-Feature_Engineering_avancé.ipynb`**  
Objectif : enrichir la représentation des données pour améliorer la performance des modèles.  
Le notebook ajoute des agrégations métier, des encodages avancés (dont target encoding K-fold), des interactions, puis applique des étapes de réduction/validation (corrélations, PCA, adversarial validation).  
Entrées principales : `new_data/train_engineered.csv`, `new_data/test_engineered.csv`.  
Sorties : `new_data/train_advanced_fe.csv` et `new_data/test_advanced_fe.csv`.

**`6-Modélisation_multi-approches.ipynb`**  
Objectif : entraîner, comparer et combiner plusieurs familles de modèles pour produire les soumissions finales.  
Le notebook est organisé en blocs : modèles décomposés actuariels, modèles directs (Tweedie/Poisson/Hurdle), AutoML (AutoGluon), puis stacking et ensembles pondérés à partir des prédictions OOF.  
Entrées principales : datasets `train_advanced_fe` et `test_advanced_fe` (chemins Databricks présents dans le notebook).  
Sortie : fichiers de soumission `submission_*.csv`.

## Exécution recommandée

Ordre de reproduction du pipeline principal :

1. `1-Exploration_traitement_variables.ipynb`
2. `2-Data_Engineering.ipynb`
3. `3-Théorie_des_valeurs_extrêmes.ipynb`
4. `4-Traitement_censure_droite.ipynb`
5. `5-Feature_Engineering_avancé.ipynb`
6. `6-Modélisation_multi-approches.ipynb`

## Installation

Prérequis :

- Python 3.10+
- `pip`
- Jupyter (`jupyter lab` ou `jupyter notebook`)

Installation locale :

```bash
python -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirement.txt
```
