# Critères d'implantation d'une station Citiz

Nous avons d'abord défini 8 critères à partir de notre connaissance de l'autopartage (hypothèses),
classés en trois niveaux. Nous les avons ensuite confrontés aux données de Marseille, où Citiz
exploite 97 stations : on compare les carreaux de 200 m situés à moins de 5 min d'une station Citiz
aux autres carreaux (notebooks `2_silver_features` et `3_gold_objectif_3`).

- **Critère « Quartier »** : sert à choisir la zone, il est calculé partout et entre dans le modèle.
- **Critère « Emplacement »** : sert à placer précisément la station dans la zone retenue.

| Niveau | Signification |
|---|---|
| 🔴 Critique | Sans ce critère, la station ne marchera pas |
| 🟠 Important | Fait la différence entre une station moyenne et une bonne station |
| 🟢 Secondaire | À optimiser une fois le reste validé |

## Les 8 critères

1. **Taux de motorisation faible** — 🔴 Critique · Quartier
   - Pourquoi : les clients de l'autopartage sont surtout des ménages sans voiture, ou qui veulent revendre leur 2ᵉ voiture.
   - Mesure : part des ménages sans voiture par IRIS (INSEE, recensement 2021), reportée sur les carreaux.
   - Marseille : **confirmé**. 42 % de ménages sans voiture autour des stations Citiz, contre 20 % ailleurs (×2). C'est l'une des variables les plus importantes du modèle.

2. **Densité de population suffisante** — 🔴 Critique · Quartier
   - Pourquoi : il faut assez d'habitants à 5 min à pied pour que la voiture soit utilisée.
   - Mesure : habitants par carreau de 200 m (INSEE Filosofi 2021).
   - Marseille : **confirmé, mais insuffisant seul**. 16 874 hab/km² autour des stations, contre 5 197 ailleurs (×3). Pourtant, la densité seule ne retrouve que 51 % des zones Citiz, contre 80 % pour notre modèle complet.

3. **Proximité des transports collectifs** — 🔴 Critique · Quartier
   - Pourquoi : l'autopartage complète les transports en commun ; sans eux, les ménages gardent leur voiture.
   - Mesure : nombre d'options de transport à moins de 400 m par les vraies rues (GTFS Mistral, RTM, ZOU!, SNCF + réseau piéton OSM).
   - Marseille : **confirmé**. 4,7 options à 5 min autour des stations, contre 1,6 ailleurs. Ce n'est cependant pas l'une des premières variables du modèle.

4. **Mixité habitat / emploi** — 🟠 Important · Quartier
   - Pourquoi : les résidents utilisent la voiture le soir et le week-end, les salariés en journée, donc la voiture tourne toute la journée.
   - Mesure : commerces, bureaux et lieux d'activité à moins de 400 m (OpenStreetMap).
   - Marseille : **plus important que prévu**. On compte 93 commerces ou bureaux à 400 m autour des stations, contre 14 ailleurs (×6,5). C'est le **1er facteur** du modèle : ce critère mérite de passer en Critique.

5. **Station accessible (à pied et en voiture)** — 🟠 Important · Emplacement
   - Pourquoi : une station difficile d'accès est peu utilisée, même dans un bon quartier.
   - Mesure : distance à pied par le réseau piéton OSM (400 m) ; accès routier.
   - Dans le projet : l'accessibilité à pied est intégrée au calcul des 5 min. L'accès routier sera vérifié sur les sites finalistes.

6. **Stationnement simple** — 🟠 Important · Emplacement
   - Pourquoi : si rendre la voiture est compliqué, le client ne revient pas (centre ancien de Toulon, relief, saison touristique).
   - Mesure : parkings OpenStreetMap (2 487 parkings sur la Métropole de Toulon).
   - Dans le projet : **appliqué comme filtre**. Un carreau n'est candidat pour Citiz que s'il a un parking à moins de 400 m.

7. **Station visible** — 🟢 Secondaire · Emplacement
   - Pourquoi : aide à faire connaître le service, mais compte moins que l'accès.
   - Mesure : axe passant ou lieu fréquenté à proximité (Street View, repérage terrain).
   - Dans le projet : non automatisé, vérifié à la main sur les sites recommandés.

8. **Pas d'arbre juste au-dessus** — 🟢 Secondaire · Emplacement
   - Pourquoi : sève, fientes et feuilles salissent les voitures (entretien, image). Facile à corriger en décalant de quelques places.
   - Mesure : arbres OpenStreetMap (`natural=tree`) ou données « arbres d'alignement » de la ville.
   - Dans le projet : non automatisé, vérifié à la main sur les sites recommandés.

## Ce que les données ont ajouté

Le modèle entraîné sur Marseille fait ressortir **deux facteurs que nous n'avions pas listés** :

- **Le revenu** des habitants (INSEE Filosofi) ;
- **La part de logements collectifs** : 94 % autour des stations Citiz, contre 53 % ailleurs.

## Conclusion

- Nos hypothèses sont **globalement confirmées** : les zones Citiz de Marseille sont denses, peu motorisées et bien desservies.
- **La mixité habitat / emploi est sous-estimée** dans notre classement : c'est le 1er facteur du modèle.
- **Aucun critère ne suffit seul** : la densité seule retrouve 51 % des zones Citiz, la combinaison de tous les critères en retrouve 80 %.
- **Limite** : le modèle apprend où Citiz *est implanté*, pas où il est *le plus utilisé* (pas de données de réservation).