# Monitoring des instances Lodex

## Objectif

Conserver un historique mensuel des instances Lodex afin de :

* suivre l'évolution du parc d'instances ;
* détecter automatiquement les instances supprimées ;
* produire des indicateurs de monitoring ;
* préparer les opérations de nettoyage.

---

## Fichiers utilisés

### Exports mensuels

```text
fin2026-03.json
fin2026-04.json
fin2026-05.json
...
```

### Exports transformés

```text
fin2026-03.jsonl
fin2026-04.jsonl
fin2026-05.jsonl
...
```

### Agrégations

```text
agregationPour2026-05.jsonl
agregationPour2026-06.jsonl
agregationPour2026-07.jsonl
...
```

### Loaders

```text
loaderEzCrawl.ini
loaderAggregateEzcrawlReports.ini
```

---

## Étape 1 : Générer le snapshot mensuel

Depuis le répertoire `ezcrawl` :

```bash
ezcrawl > fin2026-05.json
```

Le fichier obtenu correspond à l'état des instances observé à la fin du mois de mai 2026.

---

## Étape 2 : Transformer le JSON en JSONL

Application du loader d'enrichissement :

```bash
cat fin2026-05.json | ezs loaderEzCrawl.ini > fin2026-05.jsonl
```

Vérification éventuelle :

```bash
code fin2026-05.jsonl
```

---

## Étape 3 : Agréger les snapshots

Concaténer les snapshots puis appliquer le loader d'agrégation.

Exemple pour préparer l'import de juin 2026 :

```bash
cat fin2026-04.jsonl fin2026-05.jsonl \
| ezs loaderAggregateEzcrawlReports.ini \
> agregationPour2026-06.jsonl
```

Vérification éventuelle :

```bash
code agregationPour2026-06.jsonl
```

---

## Principe de fonctionnement

Chaque snapshot contient les instances existantes au moment de l'export.

Le loader d'agrégation :

* regroupe les données par instance ;
* conserve la version la plus récente de chaque instance ;
* calcule la date de mise à jour globale (`dateMaj`) ;
* permet d'identifier les instances supprimées.

---

## Détection des suppressions

Chaque instance possède :

* `dateImport` : dernière date à laquelle l'instance a été observée ;
* `dateMaj` : date du dernier snapshot disponible.

Règle :

```text
dateImport = dateMaj  → Active
dateImport ≠ dateMaj  → Supprimée
```

---

## Exemple

### Snapshot d'avril

```text
Instance A
Instance B
```

### Snapshot de mai

```text
Instance A
```

### Résultat après agrégation

| Instance   | dateImport | dateMaj  | Statut    |
| ---------- | ---------- | -------- | --------- |
| Instance A | mai 2026   | mai 2026 | Active    |
| Instance B | avril 2026 | mai 2026 | Supprimée |

---

## Exemple complet

```bash
ezcrawl > fin2026-05.json

cat fin2026-05.json \
| ezs loaderEzCrawl.ini \
> fin2026-05.jsonl

cat fin2026-04.jsonl fin2026-05.jsonl \
| ezs loaderAggregateEzcrawlReports.ini \
> agregationPour2026-06.jsonl
```
