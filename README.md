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

## Exercice 6

Écrivez le pipeline qui affichera le nom des salles ainsi qu’un tableau nommé avis_excellents qui
contiendra uniquement les avis dont la note est de 10.

```js
pipeline = [
  {
    $match: {
      avis: {
        $exists: true,
      },
    },
  },
  {
    $project: {
      _id: 0,
      nom: 1,
      avis_excellents: {
        $filter: {
          input: "$avis",
          as: "lesavis",
          cond: { $eq: ["$$lesavis.note", 10] },
        },
      },
    },
  },
  {
    $match: {
      avis_excellents: {
        $ne: [],
      },
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 7

Écrivez le pipeline qui affichera le nombre total de salles pour chaque ville. Les résultats devront être classés par ordre décroissant du nombre de salles.

```js
pipeline = [
  {
    $group: {
      _id: "$adresse.ville",
      nbSalles: {
        $sum: 1,
      },
    },
  },
  {
    $sort: {
      nbSalles: -1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 8

Créez un pipeline pour afficher, par style musical, le nombre de salles le programmant. Utilisez
sortByCount pour afficher les styles du plus populaire au moins populaire.

```js
pipeline = [
  {
    $unwind: "$styles",
  },
  {
    $sortByCount: "$styles",
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 9

Écrivez le pipeline qui renvoie les salles dont la note moyenne des avis dépasse 8. Affichez le nom de
chaque salle et sa note moyenne dans un champ moyenne.

```js
pipeline = [
  {
    $match: {
      avis: {
        $exists: true,
      },
    },
  },
  {
    $addFields: {
      moyenne: {
        $avg: "$avis.note",
      },
    },
  },
  {
    $match: {
      moyenne: {
        $gt: 8,
      },
    },
  },
  {
    $project: {
      _id: 0,
      nom: 1,
      moyenne: 1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 10

Écrivez le pipeline qui affiche uniquement les salles dont la capacité est inférieure à 200 places.
Affichez uniquement le nom et la capacité de ces salles.

```js
pipeline = [
  {
    $match: {
      capacite: {
        $lt: 200,
      },
    },
  },
  {
    $project: {
      nom: 1,
      _id: 0,
      capacite: 1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 11

Écrivez un pipeline qui ajoute un champ nommé taille avec la valeur "petite" pour les salles de moins
de 100 places, "moyenne" pour les salles entre 100 et 500 places, et "grande" pour les salles de plus
de 500 places.

```js
pipeline = [
  {
    $addFields: {
      taille: {
        $switch: {
          branches: [
            {
              case: { $lt: ["$capacite", 100] },
              then: "petite",
            },
            {
              case: { $lt: ["$capacite", 500] },
              then: "moyenne",
            },
          ],
          default: "grande",
        },
      },
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 12

Créez un pipeline pour afficher la capacité moyenne des salles de chaque ville. Utilisez $group pour
agréger par ville et calculez la capacité moyenne avec $avg.

```js
pipeline = [
  {
    $group: {
      _id: "$adresse.ville",
      capacite_moyenne: {
        $avg: "$capacite",
      },
    },
  },
  {
    $sort: {
      capacite_moyenne: -1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 13

Écrivez le pipeline qui affiche, pour chaque salle, le nom de la salle, et un tableau mauvais_avis qui
contient uniquement les avis dont la note est inférieure ou égale à 3.

```js
pipeline = [
  {
    $match: {
      avis: {
        $exists: true,
      },
    },
  },
  {
    $project: {
      _id: 0,
      nom: 1,
      mauvais_avis: {
        $filter: {
          input: "$avis",
          as: "a",
          cond: { $lte: ["$$a.note", 3] },
        },
      },
    },
  },
  {
    $match: {
      mauvais_avis: {
        $ne: [],
      },
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 14

Utilisez $bucket pour regrouper les salles en trois catégories : celles de moins de 100 places, celles
entre 100 et 500 places, et celles de plus de 500 places. Affichez le nombre de salles pour chaque
catégorie.

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
```

## Exercice 15

Écrivez le pipeline qui affiche le nom des salles et un champ avis_moyen_dernier_mois qui calcule la
note moyenne des avis publiés uniquement au cours des 30 derniers jours.

```js
pipeline = [
  {
    $match: {
      avis: {
        $exists: true,
      },
    },
  },
  {
    $addFields: {
      avis: {
        $filter: {
          input: "$avis",
          as: "a",
          cond: {
            $gte: [
              {
                $toDate: "$$a.date",
              },
              new Date(new Date() - 30 * 24 * 60 * 60 * 1000),
            ],
          },
        },
      },
    },
  },
  {
    $match: {
      avis: {
        $ne: [],
      },
    },
  },
  {
    $addFields: {
      avis_moyen_dernier_mois: {
        $avg: "$avis.note",
      },
    },
  },
  {
    $project: {
      _id: 0,
      nom: 1,
      avis_moyen_dernier_mois: 1,
    },
  },
];
db.salles.aggregate(pipeline);
```

## Exercice 16

Écrivez un pipeline qui affiche pour chaque ville le style de musique le plus populaire (c’est-à-dire celui
programmé dans le plus de salles) et le nombre total de salles programmant ce style.

```js

```
