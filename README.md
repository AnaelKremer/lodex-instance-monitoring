# Lodex Instance Monitoring

Outils et loaders permettant de suivre l'évolution des instances Lodex à partir de snapshots mensuels.

Le projet repose sur :

* la génération d'exports mensuels avec Ezcrawl ;
* la transformation des données en JSONL ;
* l'agrégation des snapshots historiques ;
* la détection automatique des instances supprimées.

Les instances sont considérées comme :

* **Actives** lorsqu'elles apparaissent dans le dernier snapshot disponible ;
* **Supprimées** lorsqu'elles n'apparaissent plus dans le dernier snapshot.

Ce dépôt contient la documentation du workflow ainsi que les loaders EZS utilisés pour le monitoring.
