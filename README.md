# # TP2 MongoDB

## Connection à compass

- Lancer l'UI

```bash
mongodb-compass
```

- Se connecter à la base de données
- Créer une database `tp2` puis une collection `salle`
- Importer le fichier `salles.json` et confirmer

> La pipeline se situe dans l'onglet `Aggregations`, puis sélectionner le champ `text` au lieu de `stages`

## Exercice 1

Écrivez le pipeline qui affichera dans un champ nommé ville le nom de celles abritant une salle de
plus de 50 personnes ainsi qu’un booléen nommé grande qui sera positionné à la valeur « vrai »
lorsque la salle dépasse une capacité de 1 000 personnes. Voici le squelette du code à utiliser dans
le shell :

```js
pipeline = [
  {
    $match: {
      capacite: { $gte: 50 },
    },
  },
  {
    $project: {
      _id: 0,
      ville: "$adresse.ville",
      grande: { $gte: ["$capacite", 1000] },
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 2

Écrivez le pipeline qui affichera dans un champ nommé apres_extension la capacité d’une salle
augmentée de 100 places, dans un champ nommé avant_extension sa capacité originelle, ainsi
que son nom.

```js
pipeline = [
  {
    $addFields: {
      apres_extension: {
        $add: ["$capacite", 100],
      },
    },
  },
  {
    $project: {
      _id: 0,
      nom: 1,
      avant_extension: "$capacite",
      apres_extension: 1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 3

Écrivez le pipeline qui affichera, par numéro de département, la capacité totale des salles y résidant.
Pour obtenir ce numéro, il vous faudra utiliser l’opérateur $substrBytes dont la syntaxe est la
suivante :

```js
{$substrBytes: [ < chaîne de caractères >, < indice de départ >, < longueur > ]}
```

```js
pipeline = [
  {
    $addFields: {
      departement: {
        $substrBytes: ["$adresse.codePostal", 0, 2],
      },
    },
  },
  {
    $group: {
      _id: "$departement",
      capacite_total: {
        $sum: "$capacite",
      },
    },
  },
  {
    $project: {
      _id: 0,
      capacite_total: 1,
      departement: "$_id",
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 4

Écrivez le pipeline qui affichera, pour chaque style musical, le nombre de salles le programmant. Ces
styles seront classés par ordre alphabétique.

```js
pipeline = [
  {
    $unwind: "$styles",
  },
  {
    $group: {
      _id: "$styles",
      nombre_salle: {
        $sum: 1,
      },
    },
  },
  {
    $project: {
      _id: 0,
      style: "$_id",
      nombre_salle: 1,
    },
  },
  {
    $sort: {
      style: 1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 5

À l’aide des buckets, comptez les salles en fonction de leur capacité :

- celles de 100 à 500 places
- celles de 500 à 5000 places

```js
pipeline = [
  {
    $bucket: {
      groupBy: "$capacite",
      boundaries: [100, 500, 5000],
      default: "Autre",
      output: { nombre_de_salles: { $sum: 1 } },
    },
  },
];
db.salles.aggregate(pipeline);
```
