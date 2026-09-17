# Données - Classification du risque de malnutrition infantile (Togo)

Ce dépôt contient un sous-ensemble du Togo MICS6, utilisé comme base de données pour un
projet de classification du risque de malnutrition infantile.

## Sommaire

- [Source](#source)
- [Fichiers retenus](#fichiers-retenus)
- [Identifiants communs](#identifiants-communs)
- [Variables retenues](#variables-retenues)
- [Décodage des principales variables catégorielles](#décodage-des-principales-variables-catégorielles)
- [Valeurs manquantes](#valeurs-manquantes)
- [Comment charger les données](#comment-charger-les-données)
- [Conditions d'utilisation](#conditions-dutilisation)
- [Remarque](#remarque)

## Emplacement dans le projet

```
malnutrition/
├── notebooks/
│   └── preparation_donnees.ipynb
└── data/                 <- ce README
    ├── originaux/         <- fichiers .sav bruts, jamais modifiés
    └── convertis/         <- CSV et Parquet générés par le notebook
```

Ce README documente le contenu de `data/`. Les fichiers `.sav` d'origine vont
dans `data/originaux/` ; le notebook de préparation (`notebooks/`) lit ce
dossier et écrit ses résultats dans `data/convertis/`, sans jamais modifier
`data/originaux/`.

## Source

**Togo MICS6** (Multiple Indicator Cluster Survey), Institut National de la Statistique et
des Études Économiques et Démographiques (INSEED-Togo), en partenariat avec l'UNICEF.
Enquête nationale représentative, collecte de terrain autour de 2019-2020.

Distribution originale : plusieurs fichiers SPSS (`.sav`), un par unité d'enquête
(ménage, membres du ménage, moustiquaires, femmes, historique des naissances, hommes,
enfants de moins de 5 ans, enfants de 5-17 ans).

Ce dépôt ne retient que trois de ces fichiers.

## Fichiers retenus

| Fichier   | Unité d'analyse                  | Nb. lignes | Description |
|-----------|-----------------------------------|-----------:|-------------|
| `hh.sav`  | Ménage                            | 8 404      | Caractéristiques du foyer : localisation, eau, assainissement, richesse |
| `ch.sav`  | Enfant de moins de 5 ans          | 5 030      | Table centrale : caractéristiques de l'enfant et mesures anthropométriques |
| `wm.sav`  | Femme de 15 à 49 ans              | 7 657      | Informations sur la mère |

Les fichiers `hl.sav` (membres du ménage), `bh.sav` (historique des naissances), `mn.sav`,
`tn.sav`, `fs.sav` et `fg.sav` ne sont pas utilisés dans cette version du projet.

## Identifiants communs

- `HH1` : numéro de grappe (cluster)
- `HH2` : numéro de ménage
- `LN` : numéro de ligne d'un individu à l'intérieur du ménage

Ces trois identifiants permettent de relier une ligne d'un fichier à une ligne d'un autre
fichier (même ménage, même individu).

## Variables retenues

### `hh.sav` - Ménage

| Code | Signification |
|------|----------------|
| `HH6` | Milieu de résidence |
| `HH7` | Région administrative |
| `WS1` | Source principale d'eau de boisson |
| `WS11` | Type de toilettes / installation sanitaire |
| `windex5` | Quintile de bien-être du ménage (indice de richesse) |
| `HH48` | Nombre de membres du ménage |

### `ch.sav` - Enfant de moins de 5 ans

| Code | Signification |
|------|----------------|
| `HL4` | Sexe de l'enfant |
| `CAGE` | Âge de l'enfant, en mois |
| `melevel` | Niveau d'instruction de la mère |
| `BD3` | L'enfant est encore allaité |
| `CA1` | Épisode de diarrhée dans les deux dernières semaines |
| `CA14` | Épisode de fièvre dans les deux dernières semaines |
| `HAZ2`, `WAZ2`, `WHZ2` | Indices anthropométriques standardisés (référence OMS 2006) - non utilisés comme question, seulement pour construire la cible à l'entraînement |
| `HAZFLAG`, `WAZFLAG`, `WHZFLAG`, `FLAG` | Indicateurs de qualité de la mesure anthropométrique |

`BD2` (l'enfant a déjà été allaité) a été retiré : 93% de "Oui" parmi les
répondants, variable trop peu discriminante. `BD3` (allaitement en cours) est
conservé seul, mieux réparti.

**Questionnaire de terrain retenu : 13 questions**, hors identifiants et hors
indices anthropométriques (`HH6`, `HH7`, `WS1`, `WS11`, `windex5`, `HH48`,
`HL4`, `CAGE`, `melevel`, `BD3`, `CA1`, `CA14`, `CM11`).

### `wm.sav` - Mère

| Code | Signification |
|------|----------------|
| `CM11` | Nombre total de naissances vivantes de la mère |

## Décodage des principales variables catégorielles

**HH6 - Milieu**
`1` = Urbain · `2` = Rural

**HH7 - Région**
`1` = Maritime · `2` = Plateaux · `3` = Centrale · `4` = Kara · `5` = Savanes ·
`6` = Lomé Commune · `7` = Golfe Urbain

**melevel - Niveau d'instruction de la mère**
`0` = Aucun / Préscolaire · `1` = Primaire · `2` = Secondaire et plus · `9` = Manquant

**windex5 - Quintile de bien-être**
`1` = Le plus pauvre → `5` = Le plus riche · `0` = code résiduel non documenté par
l'INSEED (aucun libellé officiel), à traiter comme valeur manquante

**HL4 - Sexe**
`1` = Masculin · `2` = Féminin

**BD2 / BD3 / CA1 / CA14**
`1` = Oui · `2` = Non · `8`/`9` = Ne sait pas / Non réponse

**WS1 (extrait) - Source d'eau**
`11-14` = Robinet · `21` = Puits à pompe/forage · `31/41` = Puits ou source protégés ·
`32/42/81` = Puits, source ou eau de surface non protégés · `91/92` = Eau conditionnée

**WS11 (extrait) - Toilettes**
`11-14/21/22/31` = Installations améliorées · `23/41/51` = Installations non améliorées ·
`95` = Pas de toilettes / nature

## Valeurs manquantes

Les enquêtes MICS utilisent des codes numériques réservés pour signaler l'absence de
réponse, distincts des modalités réelles :

| Code | Signification |
|------|----------------|
| `7` | Incohérent |
| `8` | Ne sait pas (NSP) |
| `9` | Non réponse |

Ces codes apparaissent selon la variable et ne doivent pas être confondus avec une
modalité valide (par exemple, `9` sur `CA1` ne signifie pas "Non", mais "on ne sait pas").

## Comment charger les données

Depuis `notebooks/` (ou tout script situé au même niveau) :

```python
import pandas as pd
from pathlib import Path

DOSSIER_ORIGINAUX = Path("..") / "data" / "originaux"

# nécessite : pip install pyreadstat
ch = pd.read_spss(DOSSIER_ORIGINAUX / "ch.sav", convert_categoricals=False)
hh = pd.read_spss(DOSSIER_ORIGINAUX / "hh.sav", convert_categoricals=False)
wm = pd.read_spss(DOSSIER_ORIGINAUX / "wm.sav", convert_categoricals=False)

# jointure enfant -> ménage
df = ch.merge(hh, on=["HH1", "HH2"], how="left")

# jointure enfant -> mère (via le numéro de ligne de la mère, AN5)
df = df.merge(
    wm[["HH1", "HH2", "LN", "CM11"]],
    left_on=["HH1", "HH2", "AN5"],
    right_on=["HH1", "HH2", "LN"],
    how="left",
)
```

## Conditions d'utilisation

Les données sont distribuées gratuitement pour un usage de recherche légitime, à
condition qu'une description des objectifs du projet soit communiquée à l'INSEED-Togo
préalablement, et que toute publication basée sur ces données leur soit transmise
(contacts indiqués dans le fichier `Read_me_Togo_MICS6.txt` fourni avec les données).

## Remarque

Ce document décrit le contenu et la structure des données. Le détail de la méthode
(construction de la variable cible, choix des variables explicatives et exclusions,
traitement des valeurs manquantes et des jointures) fait l'objet d'une note
méthodologique séparée.
