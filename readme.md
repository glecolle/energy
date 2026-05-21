# Energy

## Comment gérer le tarif de l'électricité ? -> avoir une ligne avec les tarifs quand ils évoluent

donnée globales:
    - surface m²
    - surface vitrage pour chauffage solaire
    - prix kWh HP/HC
colonnes :
- date JJ/MM/AAAA
- total_hc
- total hp
    -> conso mois, conso jour, % hc
- chauffage_hc
- chauffage_hp
    -> conso mois, conso jour, % hc
- EV
    -> conso mois, conso jour
- ecs
    -> conso mois, conso jour
    -> divers conso mois, conso jour, % hc
- bois_poids
    -> conso mois, conso jour
    -> estimation besoin chauffage ?
    -> estimation chauffage solaire + ratio chauffage total?
    -> total chauffage (elec+bois)

bilan annuel:
- total kWh
- coût total €
- kWhef/m²
- DPE f ?
- chauffage élec
    - total
    - kWh/m²
    - ratio par rapport à l'énergie totale
    - coût
    - ratio HC
- chauffage bois
    - total
    - kWh/m²
    - coût
- ECS : idem chauffage
- Divers : idem chauffage
- bilan kg CO2



TODO
    + refonte complète du moteur de calcul pour le rendre plus générique, notamment sur les HP/HC, il doit founir des valeurs prêtes à l'emploi pour les graphiques et donner plus de contrôle à la configuration. Il faut éviter le codage d'informations structurantes en dur. La configuration doit avoir un maximum de liberté quand à la production des indicateurs, mêmes indirects.
    + on laisse le maximum de choix à la configuration pour créer des séries de données qui peuvent ensuite être affichées dans les tableaux et les graphes
    + ajouter une fonction de non régression de la préparation des données (injection de données, pas de variables globale), il faut bien valider tous les modes de calculs
    + calculer les coefficients des prévisions sur la base de la dernière année complète
    + calculer les valeurs mensuelles et annuelles pour tous les échantillons
    + déplacer les contrôles de validation du format des modèles de valeurs dans une fonction dédiée
    + définir les tables et graphiques à présenter (colonnes, affichage et tooltip), largeur et hauteur
    + définir graphes à afficher (séries), les couleurs/styles sont déjà présents dans le modèle, pour assurer la cohérence et éviter les répétitions
    + fond de cellule avec un gradient ?
    + calculer le prix avec un nouveau type de variable "bill" + champ from qui doit être ventilé sur les records précédents ne contenant pas target
    - ajouter les prix des factures d'électricité réelles
    - permettre de choisir l'année ou la plage de référence pour le calcul de la régression ? nécessite un recalcul de la page
    - bilan CO2

