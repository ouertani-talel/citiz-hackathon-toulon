# Hackathon Mobilité — où ouvrir de nouvelles stations Citiz ?

Projet Epitech T-HAK-700, équipe Justice pour Hassib, en partenariat avec Citiz et Microsoft (2,5 jours).

## Le problème et notre recommandation

Dans une ville, tout le monde n'a pas accès aux mêmes moyens de transport à deux pas de chez soi. Nous mesurons, quartier par quartier, combien d'options de mobilité (bus, train, vélo, voiture partagée) un habitant peut atteindre en 5 minutes à pied. À partir de cette carte, nous recommandons à Citiz les emplacements où une nouvelle station d'autopartage toucherait le plus d'habitants pour un budget donné.

## Pourquoi Toulon, et pourquoi Marseille pour l'entraînement

- **Toulon (Métropole Toulon Provence Méditerranée) est notre ville cible.** Citiz n'y est pas encore présent, et aucune offre partagée (vélo, trottinette) n'y est publiée en open data. C'est donc un territoire où une implantation a le plus de sens.
- **Marseille sert de ville d'entraînement.** Citiz y exploite déjà 97 stations et 193 véhicules. On peut donc y observer où les stations fonctionnent, apprendre ce qui fait une bonne station, puis appliquer ce modèle à Toulon.

## Architecture

Le projet tourne sur Microsoft Fabric et suit la méthode médaillon, avec trois lakehouses :

```
Sources open data
      |
      v
 BRONZE  (lakehouse_bronze)  données brutes, téléchargées telles quelles (Files/)
      |
      v
 SILVER  (lakehouse_silver)  données nettoyées et harmonisées (tables)
      |
      v
 GOLD    (lakehouse_gold)    tables finales pour Power BI et le modèle de prédiction
```

Les notebooks sont numérotés selon leur couche : `1_bronze_*`, `2_silver_*`, `3_gold_*`, `4_modele_*`.

Pipelines :

- `pipeline_citiz_30min` : relève automatiquement la disponibilité des voitures Citiz toutes les 30 minutes, pour estimer leur usage.
- Un pipeline complet Bronze → Silver → Gold est à venir.

## Arborescence du dépôt

```
notebooks/
  1_bronze/    récupération des données brutes
  2_silver/    nettoyage
  3_gold/      tables finales, modèle de prédiction et optimisation
  4_modele/    réservé (le modèle est pour l'instant dans 3_gold)
pipelines/     exports JSON des pipelines Fabric
powerbi/       rapport Power BI (rapport_mobilite_toulon.pbix)
docs/
  sources.md   liste des sources de données
  criteres.md  nos 8 critères
slides/        présentation de la soutenance
```

## Relancer le projet dans Fabric

1. Créer un workspace Fabric.
2. Créer trois lakehouses : `lakehouse_bronze`, `lakehouse_silver`, `lakehouse_gold`.
3. Importer les notebooks du dossier `notebooks/` dans le workspace (Importer > Notebook).
4. Attacher les notebooks `1_bronze_*` et `2_silver_*` à `lakehouse_bronze` comme lakehouse par défaut. Les notebooks silver lisent le bronze par défaut et écrivent dans `lakehouse_silver` en le montant par son chemin OneLake (workspace `citiz_hackathon`) : adapter ce chemin si le workspace porte un autre nom.
5. Exécuter les notebooks bronze (`1_bronze_citizgbfs`, `1_bronze_transports`, `1_bronze_cartes`, `1_bronze_population`, `1_bronze_menages`, `1_bronze_osm`). Les trois derniers téléchargent de gros fichiers (240 à 380 Mo) au premier lancement. Les fichiers bruts apparaissent dans `Files/` du lakehouse bronze.
6. Recréer le pipeline `pipeline_citiz_30min` à partir de `pipelines/pipeline_citiz_30min.json` : une activité Notebook qui lance `1_bronze_citizgbfs`. La planification toutes les 30 minutes se règle dans Fabric (Planifier), car elle ne figure pas dans l'export JSON.
7. Exécuter les notebooks silver dans cet ordre : `2_silver_arrets`, `2_silver_carreaux`, `2_silver_acces_5min`, `2_silver_features`. Chacun réutilise les sorties du précédent.
8. Exécuter les notebooks gold : `3_gold_objectifs_1_2`, puis `3_gold_objectif_3`. Ils lisent `lakehouse_silver` et écrivent dans `lakehouse_gold`, tous deux montés par leur chemin OneLake. `3_gold_objectif_3` lit aussi le réseau piéton et les contours IRIS dans le lakehouse bronze : l'attacher à `lakehouse_bronze` comme lakehouse par défaut.
9. Les résultats du modèle sont aussi enregistrés dans l'expérience MLflow `citiz_toulon_objectif_3` du workspace.

Les clés API visibles dans certains liens (RTM, Le Vélo…) sont des clés publiques fournies par transport.data.gouv.fr. Aucune inscription n'est nécessaire.

## Sources de données

La liste complète des sources, avec leurs liens et formats, est dans [docs/sources.md](sources.md).

## Limites assumées

- **Données temps réel ponctuelles** : les flux de disponibilité (GBFS) donnent une photo à un instant donné. La collecte toutes les 30 minutes ne couvre que la durée du hackathon, pas des mois d'historique.
- **Distances à pied** : les 5 minutes à pied sont calculées par les rues du réseau piéton OpenStreetMap, à raison de 400 m. Ce seuil ne tient compte ni du relief ni des difficultés de marche, et la qualité dépend de la complétude d'OSM.
- **Accès mesuré depuis le centre du carreau** : chaque carreau de 200 m est représenté par son centre, ce qui approxime la position réelle des habitants.
- **Offre de transport sur un jour type** : le nombre de départs à l'heure de pointe (7 h – 9 h) est calculé sur un seul mardi de référence par réseau, pas sur une moyenne de l'année.
- **Trottinettes en free-floating exclues** : leurs zones de stationnement virtuelles ne sont pas comptées comme des options d'accès.
- **Absence de donnée ne veut pas dire absence de service** : à Toulon, aucune offre de vélo ou de trottinette partagés n'est publiée en open data. Cela ne prouve pas qu'il n'en existe aucune.
- **Le modèle apprend où Citiz est déjà, pas où il réussit** : la cible est la présence d'une station Citiz à 5 minutes à Marseille. Il reproduit donc les choix passés de Citiz, sans mesurer la rentabilité de chaque station.
- **Marseille n'est pas Toulon** : un modèle entraîné sur Marseille est appliqué à une ville plus petite, au relief et aux habitudes différents.
- **Coûts et CO₂ hypothétiques** : les coûts par station (Citiz 25 000 €, arrêt de bus 15 000 €, station vélo 40 000 €) sont des hypothèses de l'équipe, à remplacer par les chiffres réels. Le CO₂ évité est un ordre de grandeur tiré des études ADEME.
- **Choix glouton** : les sites sont choisis un par un, en prenant à chaque fois celui qui ajoute le plus d'habitants. Cette méthode est simple et rapide, mais ne garantit pas la meilleure combinaison possible.

## Avancement

Fait (Bronze) :

- `1_bronze_citizgbfs` : flux GBFS Citiz Marseille (stations, véhicules, statut), collectés toutes les 30 minutes.
- `1_bronze_transports` : horaires GTFS Mistral (Toulon), RTM (Marseille), ZOU (trains régionaux) ; flux GBFS Le Vélo, Voi, Lime, Pony (Marseille).
- `1_bronze_cartes` : contours IRIS 2023 (IGN), filtrés sur Marseille (393 IRIS) et TPM (176 IRIS), projetés en Lambert-93 (EPSG:2154).
- `1_bronze_population` : population par carreau de 200 m (INSEE Filosofi 2021), filtrée sur TPM et Marseille.
- `1_bronze_menages` : motorisation des ménages par IRIS (INSEE recensement Logement 2021) : ménages sans voiture, avec 0 ou 1 voiture.
- `1_bronze_osm` : extrait OpenStreetMap PACA (Geofabrik) découpé sur TPM et Marseille (avec 1 km de marge) : réseau piéton, parkings, lieux d'activité (commerces, bureaux, écoles, hôpitaux…) et offres de partage déjà cartographiées (autopartage, vélos).

Fait (Silver) :

- `2_silver_arrets` : réunit dans une seule table tous les points de mobilité de Toulon et Marseille. Arrêts de transport en commun (avec le mode et le nombre de départs à l'heure de pointe sur un mardi type), stations de vélos et de trottinettes, stations Citiz avec leur nombre de voitures. Les véhicules en libre accès sont rangés dans une table à part.
- `2_silver_carreaux` : rattache chaque carreau de population à son IRIS et y ajoute la part de ménages sans voiture, la densité, le revenu moyen et la part des 65 ans et plus.
- `2_silver_acces_5min` : regroupe les quais d'un même arrêt en une seule option, puis calcule par le réseau piéton OSM quelles options chaque carreau atteint en 5 minutes à pied (400 m).
- `2_silver_features` : construit, par carreau, les indicateurs du futur modèle : population, motorisation, revenu, transports et vélos à 5 minutes, stations et voitures Citiz à 5 minutes, activités et parkings à moins de 400 m, distance à la gare la plus proche.

Tables produites dans `lakehouse_silver` : `arrets`, `vehicules_libres`, `carreaux`, `options`, `acces_5min` et la table de features.

Fait (Gold) :

- `3_gold_objectifs_1_2` :
  - Objectif 1 : pour chaque carreau, nombre d'options de mobilité à 5 minutes à pied et options pour 1 000 habitants (méthode 2SFCA, qui tient compte du nombre de personnes qui partagent la même option). Les carreaux sont rangés en 6 classes, de « aucune option » à « très bonne ». Les carreaux de moins de 11 habitants sont signalés pour ne pas être affichés seuls (confidentialité).
  - Objectif 2 : pour chaque arrêt ou station, habitants et ménages sans voiture desservis à 5 minutes, et habitants pour qui c'est la seule option de ce type.
  - Tables `carte_acces` et `options_desservies` dans `lakehouse_gold`.
- `3_gold_objectif_3` :
  - Modèle : apprend sur Marseille quels carreaux ont une station Citiz à 5 minutes, à partir de 10 indicateurs (densité, motorisation, revenu, transports, activités, parkings, distance à la gare…). Trois modèles sont comparés avec une validation par blocs de 1 km, pour éviter que des carreaux voisins faussent le test. Le meilleur (gradient boosting) donne un score Citiz à chaque carreau de Toulon.
  - Optimisation : propose des emplacements pour trois types de sites (station Citiz, arrêt de bus, station vélo), choisis un par un selon les habitants nouvellement desservis, puis classés par habitants gagnés pour 1 000 €. Chaque site indique sa commune, son quartier, son coût cumulé et, pour Citiz, le CO₂ évité estimé.
  - Tables `scores_toulon` et `sites_recommandes` dans `lakehouse_gold`, et expérience MLflow `citiz_toulon_objectif_3`.

Fait (Power BI) :

- `powerbi/rapport_mobilite_toulon.pbix` : rapport construit sur les tables de `lakehouse_gold`, pour présenter la carte d'accès, la desserte et les sites recommandés.

Premiers résultats :

- Habitants sans aucune option à 5 minutes à pied : 23,5 % à Toulon, 16,7 % à Marseille.
- Validation du modèle sur Marseille : AUC de 0,965 pour le gradient boosting, contre 0,872 avec la densité seule. Le premier facteur est le nombre d'activités (commerces, bureaux, équipements) à moins de 400 m, devant le revenu moyen.
- Premiers sites recommandés à Toulon : une station Citiz à Dutasta-Mayol, puis des stations vélo à Saint-Louis et Saint-Roch.

À venir :

- Emplois par IRIS (INSEE recensement).
- Remplacer les hypothèses de coût par les chiffres réels de Citiz et de la métropole.
- Pipeline complet Bronze → Silver → Gold.

## Équipe

- Claude Capito — chef de groupe
- Talel Ouertani
- James Junior Pamara
- Zakaria Abdelkhaled
- Colin Parent

## Branches

- `main` : versions stables et livrables.
- `dev` : travail en cours, intégré avant de passer sur `main`.
