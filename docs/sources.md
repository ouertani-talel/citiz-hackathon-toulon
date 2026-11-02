# Sources de données

Toutes les données utilisées sont publiques (open data).

## Autopartage

| Donnée | Producteur | Format | Lien |
|---|---|---|---|
| Stations et véhicules Citiz Marseille | Citiz | GBFS 3.0 | https://backend.citiz.fr/public/provider/6/gbfs/3.0/gbfs.json |

## Transports en commun

| Donnée | Producteur | Format | Lien |
|---|---|---|---|
| Réseau Mistral (Toulon) | Métropole Toulon Provence Méditerranée | GTFS | https://transport.data.gouv.fr/datasets/reseau-de-transport-urbain-de-la-metropole-toulon-provence-mediterranee |
| Réseau RTM (Marseille) | Métropole Aix-Marseille-Provence | GTFS | https://transport.data.gouv.fr/datasets/reseaux-de-transports-en-commun-de-la-metropole-daix-marseille-provence-et-des-bouches-du-rhone |
| Trains régionaux ZOU | Région Sud | GTFS | https://transport.data.gouv.fr/datasets/zou |

## Vélos et trottinettes partagés (Marseille)

| Donnée | Producteur | Format | Lien |
|---|---|---|---|
| LeVélo | Métropole Aix-Marseille-Provence | GBFS 2.2 | https://transport.data.gouv.fr/datasets/velos-a-assistance-electrique-en-libre-service-levelo-sur-marseille |
| Voi | Voi | GBFS | https://transport.data.gouv.fr/datasets/voi-gbfs-et-netex-marseille |
| Lime | Lime | GBFS | https://transport.data.gouv.fr/datasets/gbfs-marseille |
| Pony | Pony | GBFS | https://transport.data.gouv.fr/datasets/velos-pony-a-marseille-gbfs-netex |

Aucune offre de vélos ou trottinettes partagés n'est publiée en open data pour Toulon.

## Territoire et population

| Donnée | Producteur | Format | Lien |
|---|---|---|---|
| Population par carreau de 200 m (Filosofi 2021) | INSEE | Shapefile | https://www.insee.fr/fr/statistiques/8735162 |
| Motorisation des ménages par IRIS (recensement 2021) | INSEE, base IC Logement | CSV | https://www.insee.fr/fr/statistiques/fichier/8268838/base-ic-logement-2021_csv.zip |
| Contours IRIS 2023 | IGN / INSEE | Shapefile | https://data.geopf.fr/telechargement/download/CONTOURS-IRIS/CONTOURS-IRIS_3-0__SHP__FRA_2023-01-01/CONTOURS-IRIS_3-0__SHP__FRA_2023-01-01.7z |
| Communes et contours (Métropole TPM, Marseille) | API Découpage administratif (geo.api.gouv.fr) | GeoJSON | https://geo.api.gouv.fr |

## Réseau piéton et points d'intérêt

| Donnée | Producteur | Format | Lien |
|---|---|---|---|
| OpenStreetMap PACA (rues et chemins piétons, parkings, commerces et équipements, autopartage et vélos déjà cartographiés) | Contributeurs OpenStreetMap, extrait Geofabrik | PBF | https://download.geofabrik.de/europe/france/provence-alpes-cote-d-azur-latest.osm.pbf |

Données OpenStreetMap © contributeurs OpenStreetMap, sous licence ODbL.

## Hypothèses de coût et d'impact (objectif 3)

| Donnée | Producteur | Utilisation |
|---|---|---|
| Kilomètres évités par voiture partagée (10 000 à 19 000 km/an) | ADEME, enquête nationale autopartage 2022 | Estimation du CO₂ évité par station Citiz |
| Émissions d'une voiture particulière (≈ 0,2 kg CO₂/km, ordre de grandeur) | ADEME, Base Empreinte (à vérifier) | Estimation du CO₂ évité |
| Coûts d'installation (Citiz 25 000 €, arrêt de bus 15 000 €, station vélo 40 000 €) | Hypothèses de l'équipe | Optimisation sous budget, à remplacer par les chiffres de Citiz et de la métropole |

## Notes

- Les clés API présentes dans certains liens (RTM, LeVélo) sont des clés publiques fournies par transport.data.gouv.fr.
- Les données GBFS sont des instantanés en temps réel. La disponibilité Citiz est collectée toutes les 30 minutes pour estimer l'usage des stations.
- Dans les carreaux de moins de 11 ménages, l'INSEE estime les valeurs pour protéger la vie privée (colonne `i_est_200`). Les résultats y sont donc moins précis.