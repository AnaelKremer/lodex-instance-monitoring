# Monitoring des instances Lodex

## Objectif

Conserver un historique mensuel des instances Lodex afin de :

* suivre l'évolution du parc d'instances
* détecter automatiquement les instances supprimées
* produire des indicateurs de monitoring
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

## Étape 2 : Transformation des données

Application du loader :

```bash
cat fin2026-05.json | ezs loaderEzCrawl.ini > fin2026-05.jsonl
```

### Loader EzCrawl

```INI
append = pack
label = json-lines

# load some plugins to activate some statements
[use]
plugin = basics

[unpack]
; Définit un dictionnaire permettant d’associer chaque gestionnaire d’instance à un service.
[env]
path =dictionary
value = fix({"A. Kremer" : "APIL", \
             "J. Mourot" : "APIL", \
             "P. Ranoarisoa" : "APIL", \
             "C. Gay" : "APIL", \
             "E. Giret" : "APIL", \
             "S. Barreaux" : "ISTEX" ,\
             "E. Caule" : "ISTEX" ,\
             "M. Huguin" : "ISTEX" ,\
             "P. Viot" : "ISTEX" ,\
             "P. Cuxac" : "TDM",\
             "N. Thouvenin" : "DAFIS"\
})

[exchange]
value = get("instance")
; Dans le dump Ezcrawl, les informations utiles sont contenues dans le champ instance.

[remove]
test = get("technicalName").thru(name => name !== "instance-globale-161010")
; On supprime tout ce qui ne concerne pas l'instance globale utilisée

[exchange]
value = get("tenant")
; Remplace l’objet courant par le contenu du champ `tenant`, qui contient les informations principales de l’instance.

[assign]
path = service
value = get("author").thru(status => env("dictionary")[status] ?? "inconnu")
; Associe l’instance à un service à partir du champ `author`.

path = datePourGraph
value = get("createdAt").slice(0, 7).join("")
; Extrait l’année et le mois de création de l’instance au format `AAAA-MM`.

path = published
value = get("published").thru(bool => bool === true ? "Oui" : "Non")
; Indique si l'instance est publiée.

path = hasData
value = get("dataset").thru(v => v === 0 ? "Non" : "Oui")
; Indique si l’instance contient des données.

path = mailToContact
value = get("description").split(";").last().trim().thru(v => _.includes(v,"@") ? v : "Pas de mail")
; Récupère une adresse mail de contact à partir de la description.
; Le loader prend la dernière partie de la description après le caractère `;`.
; Si aucune adresse mail n’est détectée, la valeur devient `Pas de mail`.

path = dateImport
value = fix(new Date()) \
  .thru(d => new Intl.DateTimeFormat('fr-FR', { \
    year: 'numeric', \
    month: 'long', \
    day: 'numeric', \
  }).format(d))
; Cette date permet de savoir à quel moment les données ont été récupérées.

path = instanceUrl
value = fix(`https://lodex.istex.fr/instance/${self.name}`)
; Construit automatiquement l’URL publique de l’instance.

path = verbalizedCreatedDate
value = get("createdAt").thru(d => new Intl.DateTimeFormat('fr-FR', { \
    year: 'numeric', \
    month: 'long' \
  }).format(new Date(d)))
; Produit une date de création lisible

[assign]
path = fieldsForTemplate
value = fix(self.service, self.instanceUrl, self.verbalizedCreatedDate, self.mailToContact, self.published, self.hasData)
; Regroupe plusieurs champs dans un tableau afin de faciliter leur réutilisation dans un template EJS
; permettant d'envoyer un mail aux détenteteurs d'instances inutilisées.

[remove]
test=fix(self.name,self.author).join("|").isEqual("default|Root")
; On supprimme l'instance `default` de la liste des instancews.

[exchange]
value = omit(["username","password","totalSize"])
; Supprime les champs non nécessaires ou sensibles avant import dans Lodex.

# Ensures that each object contains an identification key (required by lodex)
[swing]
test = pick(['URI', 'uri']).pickBy(_.identity).isEmpty()
[swing/identify]

# Ignore objects with duplicate URI
[dedupe]
ignore = true

# Prevent keys form containing dot path notation (which is forbidden by nodejs mongoDB driver)
[OBJFlatten]
separator = fix('.')
reverse = true
```

---

## Étape 3 : Agréger tous les snapshots

Une fois le snapshot mensuel transformé en JSONL, on reconstruit l’agrégation complète à partir de tous les snapshots disponibles.

```bash
cat fin*.jsonl | ezs loaderAggregateEzcrawlReports.ini > agregationPour2026-06.jsonl
```

### Loader AggregateEzcrawlReports

```INI
append = pack
label = json-import

# load some plugins to activate some statements
[use]
plugin = basics
plugin = analytics

; since lodex@12.23
[unpack]

; On fait l'agrégation depuis les url des instances
[replace]
path = id
value = get("publicUrl")

path = value
value = self()

[aggregate]

[assign]
path = latest
value = get("value").maxBy(obj => \
  new Date( \
    obj.dateImport.split(' ')[2], \
    { \
      janvier:0, \
      février:1, \
      mars:2, \
      avril:3, \
      mai:4, \
      juin:5, \
      juillet:6, \
      août:7, \
      septembre:8, \
      octobre:9, \
      novembre:10, \
      décembre:11 \
    }[obj.dateImport.split(' ')[1].toLowerCase()], \
    obj.dateImport.split(' ')[0] \
  ) \
).omit(["uri"])
; Sélectionne, pour chaque instance, la version la plus récente à partir du champ dateImport.

[exchange]
value = get("latest")
; Remplace l’objet courant par sa version la plus récente connue.

# Ensures that each object contains an identification key (required by lodex)
[swing]
test = pick(['URI', 'uri']).pickBy(_.identity).isEmpty()
[swing/identify]

# Ignore objects with duplicate URI
[dedupe]
ignore = true

# Prevent keys from containing dot path notation (which is forbidden by nodejs mongoDB driver)
[OBJFlatten]
separator = fix('.')
reverse = true
safe = true

# Uncomment to see each data sent to the database
#[debug]

# Add contextual metadata related to the import
[assign]
path = dateMiseAJour
value = fix(new Date()) \
  .thru(d => new Intl.DateTimeFormat('fr-FR', { \
    year: 'numeric', \
    month: 'long', \
    day: 'numeric', \
  }).format(d))

[assign]
path = statut
value = get("dateMiseAJour").thru(date => date === self.dateImport ? "Ouverte" : "Supprimée")
```

---

## Principe de fonctionnement

Chaque snapshot contient les instances existantes au moment de l'export.

Le loader d'agrégation :

* regroupe les données par instance
* conserve la version la plus récente de chaque instance
* calcule la date de mise à jour globale (`dateMaj`)
* permet d'identifier les instances supprimées.

---

## Détection des suppressions

Chaque instance possède :

* `dateImport` : dernière date à laquelle l'instance a été observée
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
```

```bash
cat fin2026-05.json | ezs loaderEzCrawl.ini > fin2026-05.jsonl
```

```bash
cat fin2026-04.jsonl fin2026-05.jsonl | ezs loaderAggregateEzcrawlReports.ini > agregationPour2026-06.jsonl
```
