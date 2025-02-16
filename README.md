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



---

## Description du code 


 
