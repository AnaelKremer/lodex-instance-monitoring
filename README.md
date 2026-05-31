# Lodex Instance Monitoring

Outils et loaders permettant de suivre l'évolution des instances Lodex à partir de snapshots mensuels.


Le projet repose sur :

* la génération d'exports mensuels d'une machine EZmaster avec Ezcrawl
* la transformation des données via un loader spécifique
* l'agrégation des snapshots historiques vie un autre loader
* la détection automatique des instances supprimées, d'sintances créées et un template qui permet, dans Lodex, d'evoyer automatiquement des mails aux détenteurs d'instances.

Les instances sont considérées comme :

* **Actives** lorsqu'elles apparaissent dans le dernier snapshot disponible ;
* **Supprimées** lorsqu'elles n'apparaissent plus dans le dernier snapshot.

Ce dépôt contient la documentation du workflow ainsi que les loaders utilisés pour le monitoring.

## Prérequis

Ce projet s'appuie sur plusieurs outils externes.

### Environnement Linux

Les commandes présentées dans cette documentation utilisent **Bash**.

Sous Windows, il est recommandé d'utiliser **WSL** (Windows Subsystem for Linux).

### Ezcrawl

Les snapshots mensuels sont générés à l'aide d'**Ezcrawl**, un outil développé en **Rust** permettant d'extraire des informations sur les instances **Lodex**.

Exemple :

```bash
ezcrawl > fin2026-06.json
```

### EZS

Les transformations et agrégations sont réalisées avec **EZS** à l'aide de fichiers de configuration `.ini`.

Les loaders fournis dans ce dépôt nécessitent notamment les plugins :

- `basics`
- `analytics`

Les commandes suivantes doivent donc être exécutées dans un environnement disposant d'une installation fonctionnelle d'**EZS** et de ses plugins.

Exemple :

```bash
cat fin2026-06.json | ezs loaderEzCrawl.ini > fin2026-06.jsonl
```

*L'exécution des scripts en **bash** n'est pas nécessaires mais évite de charger les dumps dans **Lodex** avec le 1er loader. D'extraire le résultat, puis de le réimporter avec le 2nd loader.*

### Dépendances utilisées

| Outil | Usage |
|---------|---------|
| Bash | Exécution des commandes |
| WSL (Windows) | Environnement Linux recommandé |
| Ezcrawl | Génération des snapshots mensuels |
| EZS | Transformation et agrégation des données |
| Plugin `basics` | Fonctions de transformation |
| Plugin `analytics` | Fonctions d'agrégation |