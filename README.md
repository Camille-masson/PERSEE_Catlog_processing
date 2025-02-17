---
title: "Catalog Processing - README"
author: '[MASSON Camille / Script écrit par : PERRON Rémy]'
date: 'Dernière mise à jour : [14/02/2025]'
output:
  html_document:
    toc: true
    toc_float: true
    number_sections: true
    theme: readable
    css: styles.css
  pdf_document:
    toc: true
    number_sections: true
    latex_engine: xelatex
    extra_dependencies: ["georgia"]
fontsize: 12pt
mainfont: Georgia
geometry: margin=1in
---

<style>
  body {
    font-family: Georgia, serif;
    text-align: justify;
  }
  h1, h2, h3, h4, h5, h6 {
    font-family: Georgia, serif;
  }
</style>

## Présentation

Ce script réalisé par PERRON Rémy durant sa thèse permet d’extraire et d’analyser les données issues des colliers GPS Catlog afin de caractériser les déplacements selon trois comportements :

- Repos  
- Déplacement  
- Pâturage  

Il permet également de calculer les taux de chargement sur les alpages étudiés. Il a été adapté pour permettre un partage de cette méthode aux différents utilisateurs dans le cadre du projet PERSEE.

L’implémentation repose sur R et RStudio, avec une gestion collaborative via GitHub.

---

## Installation et Configuration du Dépôt

### Prérequis

Avant de commencer, assurez-vous d’avoir installé le logiciel Git :

- [Git](https://git-scm.com/) | Assurez-vous également de l'avoir correctement configuré avec votre IDE (RStudio). Vous pouvez consulter les tutoriels suivants si nécessaire :  
  - [Tuto_1](https://youtu.be/QLFc9gw_Hfs?si=W2lMnlM_NHd5S7ba)  
  - [Tuto_2](https://youtu.be/bUoN85QvC10?si=O33JTkJ3NFHiF1zp)




### Clonage du Dépôt

Le projet fonctionne sous forme d’un dépôt GitHub. Pour le cloner localement :

```
# Étapes d’installation
1. Créer un compte GitHub (si ce n’est pas déjà fait)
2. Ouvrir RStudio
3. Aller dans Fichier → Nouveau projet → Version Control → Git
4. Entrer l’URL du dépôt GitHub et sélectionner un répertoire local pour l’enregistrer :

git clone : https://github.com/Camille-masson/Catlog_processing.git
```

---

## Organisation des Fichiers

Le projet suit une organisation bien structurée :

```
📂 Catlog_processing/ # Répertoire principal du projet
│-- 📂 Functions/     # Dossier contenant les fonctions utilisées, partagé via Git
│-- 📂 raster/        # Fichiers NDVI nécessaires aux calculs, format .tif
│-- 📂 data/          # Données d’entrée, avec un exemple de jeu d’entraînement
│   │-- 📂 Colliers_AAAA_brutes/(AAAA = Année d’étude, 9999 pour le jeu de démonstration)
│   │   │-- 9999_infos_alpages.csv  (Infos sur les alpages étudiés)
│   │   │-- 9999_tailles_troupeaux.csv (Taille des troupeaux au cours de la saison)
│   │   │-- 9999_colliers_poses.csv (Détails des colliers et paramètres)
│   │   │-- 📂 Alpage_demo/  (Données GPS des individus étudiés)
│   │   │   │-- C00_000000.csv  (Exemple de fichier GPS d’un collier)
│   │   │   │-- C01_000000.csv  (Exemple de fichier GPS d’un collier)
│   │   │   │-- C02_000000.csv  (Exemple de fichier GPS d’un collier)
│-- 📂 outputs/       # Dossier où seront enregistrés les fichiers de sortie
│-- config.R          # Script définissant les chemins et le chargement des packages
│-- script_analysis.R # Script principal exécutant les différentes étapes de traitement
```

Tous les fichiers seront donc déjà créés lors du partage du projet Git ! Cependant, notamment pour le dossier `data`, il faudra organiser correctement les fichiers en suivant le modèle explicité ci-dessus et en s’appuyant sur la structure du jeu de données d’entraînement fourni. Voir la partie 4 pour plus détail sur la forme et la strcture des données. 



---

## Description des données d’entrée

### Jeu de données de démonstration
Ce jeu de données est deja strcturé et paratgé de manière correcte via le git. Jeu de données dont l’année d’échantillonnage est fictive, fixée à 9999, et dont l’alpage fictif est établi sous le nom *Alpage_demo*. Ce jeu de données d’entraînement présente un échantillon de points GPS de 3 colliers sur une période d’échantillonnage restreinte à 1 mois. Il permet à la fois d’illustrer et de comprendre les différentes tables d’entrée, mais également de faciliter l’interprétation du script principal.

### Données GPS

Chaque collier GPS extrait via le logiciel Catlog génère un fichier `.csv` encodé en UTF-8 contenant les colonnes suivantes :

| **Colonne**           | **Description**                                        |
|-----------------------|--------------------------------------------------------|
| Date                 | Date d’enregistrement *(format : `dd/mm/yyyy`)*           |
| Time                 | Heure d’enregistrement *(format : `hh:mm:ss`)*           |
| Latitude             | Latitude GPS                                          |
| Longitude            | Longitude GPS                                         |
| Altitude             | Altitude en mètres                                    |
| Satellites           | Nombre de satellites utilisés                         |
| HDOP                 | Précision horizontale du GPS                          |
| PDOP                 | Précision tridimensionnelle du GPS                    |
| Temperature [C]      | Température mesurée en degrés Celsius                 |
| Speed [km/h]        | Vitesse en km/h                                       |
| TTFF                 | Temps nécessaire pour le premier signal GPS           |
| SNR AVG              | Rapport signal/bruit moyen des satellites utilisés    |
| SNR MAX              | Rapport signal/bruit maximum parmi les satellites     |

**Remarque** : Pour les utilisateurs n’ayant pas de colliers Catlog, seules les colonnes Date, Time, Latitude, Longitude, Altitude sont indispensables.


---

### Tables de Métadonnées

#### Informations des alpages étudiés : `AAAA_infos_alpages.csv`

Ce tableau regroupe les informations principales liées à l’alpage. Dans le jeu de données de démonstration, un seul alpage est détaillé, mais si d’autres alpages figurent dans votre jeu de données, il suffit d’ajouter de nouvelles lignes en spécifiant un nom de référence pour chaque alpage.

| **Variable**              | **Description**                                                    |
|---------------------------|------------------------------------------------------------------|
| alpage                    | Nom de l’alpage étudié                                          |
| nom_alpage_determinant    | Nom utilisé pour identifier l’alpage                            |
| date_pose                 | Date de pose des colliers *(format : `dd/mm/yyyy hh:mm:ss`)*      |
| date_retrait              | Date de retrait des colliers *(format : `dd/mm/yyyy hh:mm:ss`)*   |
| proportion_jour_allume    | Proportion de la journée où les colliers sont actifs *(1 = 24h/24)* |
| taille_troupeau           | Taille du troupeau associé à l’alpage                          |
| nom1_UP                   | Nom de l’unité de pâturage principale                          |
| medcrit                   | Valeur médiane critique de l’alpage                            |
| meancrit                  | Valeur moyenne critique de l’alpage                            |
| spikesp                   | Nombre de pics spécifiques                                     |
| spikecos                  | Indice de corrélation des pics                                |


#### Évolution de la taille du troupeau : `AAAA_tailles_troupeaux.csv`

Ce tableau contient l’évolution de la taille du troupeau au cours de la saison d’alpage. La taille du troupeau peut être fixe (cas principal, où elle ne change pas), ou variable en fonction du temps, notamment pour des contraintes techniques ou des mesures de gestion (exemple : alpage du Viso). Dans ce cas, il est important de noter les variations de la taille du troupeau en fonction du temps.

| **Variable**              | **Description**                                                    |
|---------------------------|------------------------------------------------------------------|
| alpage                    | Nom de l’alpage étudié                                          |
| date_debut_periode        | Date de début de la période *(format : `dd/mm/yyyy`)*            |
| taille_totale_troupeau    | Taille totale du troupeau à cette date                          |


#### Informations individuelles sur les colliers et les individus : `AAAA_colliers_poses.csv`

Ce tableau contient les informations relatives aux individus sur lesquels les colliers ont été posés, telles que l’espèce, la période d’échantillonnage, la proportion de jour allumé, ainsi que les dates de pose et de retrait pour chaque individu.

| **Variable**              | **Description**                                                   |
|---------------------------|-----------------------------------------------------------------|
| Collier                   | Identifiant du collier                                         |
| Bat                       | Référence de la brebis portant le collier                     |
| Programmation             | Type de programmation du collier                              |
| Alpage                    | Nom de l’alpage où l’individu se trouve                      |
| Espèce                    | Espèce de l’individu                                         |
| Race                      | Race de l’individu                                           |
| Éleveur                   | Identifiant de l’éleveur                                      |
| Âge                       | Âge de l’individu en mois                                    |
| Période échantillonnage   | Période d’échantillonnage en fraction de la journée          |
| Proportion jour allumé    | Durée pendant laquelle le collier est actif                  |
| Date pose                 | Date de pose du collier *(format : `dd/mm/yyyy hh:mm:ss`)*     |
| Date retrait              | Date de retrait du collier *(format : `dd/mm/yyyy hh:mm:ss`)*  |



## Description du code

### config.R

Le script `config.R` permet de configurer automatiquement l’environnement de travail du projet. Il définit :

- Les chemins des dossiers principaux
- Le chargement des bibliothèques
- Le chargement automatique des fonctions
- Les paramètres globaux

*Remarque* : Il est essentiel d’exécuter (`source`) ce script pour garantir le bon fonctionnement du projet. Ce script est automatiquement appelé dans `script_analysis.R` (Section `0. LIBRARIES AND CONSTANTS`).

---

### script_analysis.R

Le script `script_analysis.R` est le script principal permettant de générer toutes les données de sortie (caractérisation des comportements, calcul du taux de chargement, etc.) à partir des données GPS.

Ce script s’articule en cinq parties décrites ci-dessous.



---

**Partie 1 : SIMPLIFICATION DATA EN GPKG**

**Objectif :** 

Cette première étape permet de simplifier les données GPS brutes et de les transformer en un fichier GPKG (GeoPackage). L'objectif est de visualiser dans QGIS la date exacte de pose et retrait des colliers GPS.

**Données d’entrée :** 

- Données brutes GPS des colliers : `data/Colliers_9999_brutes/`

**Données de sortie :**

- Fichier GPKG généré dans `outputs/GPS_simple_GPKG/`
- Nom du fichier : `Donnees_brutes_9999_Alpage_demo_simplifiees.gpkg`
- Ce fichier contiendra les données simplifiées par alpage

**Etapes de traitement QGIS pour identifier la date de pose et de retrait des colliers :**

1. Exécuter la section de code dans `script_analysis.R`.  
   Cela va générer le fichier GPKG dans `outputs/GPS_simple_GPKG/`.

2. Importer la couche GPKG dans QGIS.  
   - Ouvrir QGIS.  
   - Aller dans "Ajouter une couche" → "Ajouter une couche vecteur".  
   - Sélectionner le fichier `Donnees_brutes_9999_Alpage_demo_simplifiees.gpkg`.

3. Créer une colonne "Datetime" pour l'analyse temporelle.  
   - Ouvrir "Calculatrice de champ".  
   - Ajouter un nouveau champ :  
     - Nom : `Datetime`  
     - Type : `Date et heure`  
     - Expression :  
       `to_datetime(date)` 
     - Valider et appliquer.

4. Appliquer un style coloré par ID de collier.  
   - Aller dans Propriétés de la couche → Symbologie.  
   - Choisir "Catégorisé".  
   - Sélectionner le champ `ID`.  
   - Cliquer sur "Classer" pour attribuer une couleur unique à chaque collier.

5. Activer le contrôle temporel dynamique.  
   - Aller dans Propriétés de la couche → Onglet "Temporel".  
   - Cocher la case "Activer le contrôle temporel dynamique".  
   - Dans "Configuration", choisir :  
     - "Champ unique avec date et heure".  
     - Sélectionner le champ `Datetime`.  
   - Valider.

6. Utiliser l'animation temporelle pour visualiser les trajectoires GPS.  
   - Dans la fenêtre principale, cliquer sur l’icône d’horloge pour ouvrir le Panneau de contrôle temporel.  
   - Régler la "Plage d’animation" avec :  
     - Date de début et date de fin correspondant à la saison d’étude.  
     - Pas de temps : 24 heures.

7. Explorer les trajectoires GPS pour identifier les dates de pose et retrait des colliers.  
   - Jouer l’animation pour voir quand chaque collier a cessé d’émettre.  
   - Cocher/décocher les colliers dans la légende pour isoler leur mouvement.




---

**Partie 2 : BJONERAAS FILTER CALIBRATION **

**Objectif :**

Ce script permet d’analyser et filtrer les données GPS en utilisant la méthode de Bjorneraas, pour éliminer les erreurs de position et améliorer la qualité des trajectoires. Il offre une visualisation des données brutes et filtrées, et permet de tester plusieurs ensembles de paramètres avant d’adopter un filtrage optimal.

**Principe du filtre de Bjorneraas :**

Bjorneraas et al. (2010) ont développé une méthode de filtrage des erreurs GPS en écologie du mouvement, basée sur :

- Un filtre médian (medcrit) : détecte les points aberrants sur la base des distances médianes entre points.
- Un filtre de moyenne (meancrit) : supprime les points très éloignés de la moyenne des distances parcourues.
- Un seuil de vitesse (spikesp) : élimine les points où la vitesse dépasse un seuil défini.
- Un critère de changement brutal de direction (spikecos) : supprime les virages anormaux.

Ces filtres permettent d’éliminer les erreurs dues à des sauts GPS ou des réflexions du signal.


**Données d'entrés : **

- Données brutes GPS des colliers : `data/Colliers_9999_brutes/`
- Fichier d’information sur l’alpage : `data/Colliers_9999_brutes/9999_infos_alpages.csv`
  

**Données de sortie : **

- Fichier PDF de sortie : `outputs/Filtre_de_Bjorneraas/Filtering_calibration_YEAR_Alpage_demo.pdf`
  - Ce fichier contiendra des visualisations des trajectoires GPS brutes, des résultats des filtres appliqués, et des erreurs détectées (R1 et R2).
  - Chaque graphique montre les trajectoires avec des codes de couleurs pour les points valides et les erreurs détectées.

---

**Partie 3 : FILTERING CATLOG DATA **

**Objectif :**
Cette partie permet de filtrés les données brutes des gps en appliquant le filtre de Bjorneraas. 


**Données d'entrés : **

- Données brutes GPS des colliers : `data/Colliers_9999_brutes/`
- Fichier d’information individuel sur les colliers : `data/Colliers_9999_brutes/9999_colliers_poses.csv`




**Données de sortie : **

- Données GPS filtrés : `outputs/Filtre_de_Bjorneraas/Catlog_9999_filtered_Alpage_demo.rds`
- Fichier csv : `outputs/Filtre_de_Bjorneraas/9999_filtering_Alpage_demo.csv`
  - Ce fichier contiendra le nombre de point filtrés par colliers du aux erreurs détectées (R1 et R2).
  - Ainsi que le nombre de point total tout collier confondues filtré
 

---


**Partie 4 : HMM FITTING **

**Objectif :**
Cette partie permet l'analyse des trajéctoire GPS des brebis en utilisant le modèle caché de Markov (HMM- Hidden Markov Model) afins de caractérisés trois type de comportement : déplacement, paturage, repos.
Il s'appuie sur les trajéctoires filtrées par le filtre de Bjorneraas effectué dans la partie 3. 


Cette partie permet de filtrés les données brutes des gps en appliquant le filtre de Bjorneraas. 


**Données d'entrés : **

- Données GPS filtrés : `outputs/Filtre_de_Bjorneraas/Catlog_9999_filtered_Alpage_demo.rds`
- Fichier d’information individuel sur les colliers : `data/Colliers_9999_brutes/9999_colliers_poses.csv`




**Données de sortie : **

- Trajectoires GPS catégorisées : `outputs/HMM_comportement/Catlog_9999_Alpage_demo_viterbi.rds`
- Rapport pdf : `outputs/HMM_comportement/individual_trajectories/C00.pdf`
  - Génère un pdf par collier
  - Avec les résultat individuel des ajustements du modèle de Markov caché
 

---



