# TP NOSQL 

## Log into mongo shell
`docker exec -it mongodb mongosh -u root -p @password`

## Use database
`use testdb`

## Insert into database
`db.users.insertOne({ name: "Alice", age: 30, city: "Paris" })`

`db.cours.insertOne({
  cours: "Programmation JavaScript",
  chapitres: [
    "Introduction",
    "Variables et Types",
    "Fonctions",
    "Objets"
  ],
  auteur: {
    nom: "Dupont",
    prenom: "Jean"
  }
})
`

## Find command
`db.users.find()
`
`db.users.find({ name: "Alice" })
`
`db.restaurants.find().limit(5)
`
## Log into mongo compass
- Open mongo compass app in your desktop
- Add connection `mongodb://root:example@localhost:27017`

## Lunch mongo with volum mouth
`
docker run -d \
--name mongodb \
-p 27017:27017 \
-v ~/Documents/CESI/tp-nosql/datas:/workspace \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=root \
mongo:latest
`

## Import du fichier
`
docker exec -it mongodb mongoimport \
  --username root \
  --password root \
  --authenticationDatabase admin \
  --db new_york \
  --collection restaurants \
  --file /workspace/restaurants.json \
`

## Quelques commandes pour le filtrage
- Filtrage sur les restaurants dont le quartier est Brooklyn
    `db.restaurants.find({ borough: "Brooklyn" })`
- Compter le nombre de résultats
    `db.restaurants.find().count()`
- Restaurant situé à la 5th avenue
    `db.restaurants.find({ "address.street": "5 Avenue" }).limit(10)`
- Restaurant dont le name contient Pizza avec recherche insensible à la casse
    `db.restaurants.find( { name: { $regex: "pizza", $options: "i" } } ).limit(10)`
- Ajouter une projection pour ne retourner que le champ name
    `db.restaurants.find({ name: /pizza/i },{ name: 1 })`
- Sans afficher _id
    `db.restaurants.find({ name: /pizza/i },{ name: 1, _id: 0 })`
    `db.restaurants.find( { name: { $regex: "pizza", $options: "i" } }, {name: 1, _id: 0} )`
- Filtrer un sous ensembre de restaurant
    `db.restaurants.find({ borough: "Brooklyn" },{ "grades.score": 1 })`
- Projeter le champ grades.score et sans _id
    `db.restaurants.find( { borough: "Brooklyn" }, { "grades.score": 1, _id: 0 } )`
- Garder les restaurants de Manhanttan dont le score est inférieur à 10
    `db.restaurants.find( { borough: { $regex: "Manhattan", $options: "i" }, grades: { $elemMatch: { score: { $lt: 10 } } } })`
- Score < 10 sans _id 
    `db.restaurants.find(
  {
  borough: /Manhattan/i,
  grades: { $elemMatch: { score: { $lt: 10 } } }
  },
  {
  name: 1,
  borough: 1,
  "grades.score": 1,
  _id: 0
  }
  )
`
- Récupérer uniquement les restaurants qui n'ont pas de score >= 10 
    `db.restaurants.find(
  {
    grades: {
      $not: {
        $elemMatch: { score: { $gte: 10 } }
      }
    }
  }
)
`
- Restaurants qui ont un grande C et un score < 40
    `db.restaurants.find({
  grades: {
    $elemMatch: {
      grade: "C",
      score: { $lt: 40 }
    }
  }
})
`
- Valeurs distinctes
    `db.restaurants.distinct("borough")`
- Trier des valeurs
    `db.restaurants.distinct("borough").sort()`
    `db.restaurants.distinct("borough").sort().reverse()`
- Aggregation avec aggregate() pipeline | Restaurant (nom + quartier) dont la dernière inspection a le grade C
    `db.restaurants.aggregate([
  {
    $project: {
      name: 1,
      borough: 1,
      lastInspection: { $arrayElemAt: ["$grades", 0] }
    }
  },
  {
    $match: {
      "lastInspection.grade": "C"
    }
  }
])
`
- Tri de la requete précedente par name croissant
    `db.restaurants.aggregate([
  {
    $project: {
      name: 1,
      borough: 1,
      lastInspection: { $arrayElemAt: ["$grades", 0] }
    }
  },
  {
    $match: {
      "lastInspection.grade": "C"
    }
  },
  {
    $sort: { name: 1 }
  }
])
`
- Tri par name decroissant
    `db.restaurants.aggregate([
  {
    $project: {
      name: 1,
      borough: 1,
      lastInspection: { $arrayElemAt: ["$grades", 0] }
    }
  },
  {
    $match: {
      "lastInspection.grade": "C"
    }
  },
  {
    $sort: { name: -1 }
  }
])`
- Compter le total et ajouter un compteur total
    `db.restaurants.aggregate([
  {
    $project: {
      name: 1,
      borough: 1,
      lastInspection: { $arrayElemAt: ["$grades", 0] }
    }
  },
  {
    $match: {
      "lastInspection.grade": "C"
    }
  },
  {
    $group: {
      _id: null,        
      total: { $sum: 1 }
    }
  }
])
`
- Compter par borough (quartier) le nombre de restaurants concernés puis trier en decroissant
    `db.restaurants.aggregate([
  {
    $project: {
      name: 1,
      borough: 1,
      lastInspection: { $arrayElemAt: ["$grades", 0] }
    }
  },
  {
    $match: {
      "lastInspection.grade": "C"
    }
  },
  {
    $group: {
      _id: "$borough",
      total: { $sum: 1 }
    }
  },
  {
    $sort: { total: -1 }
  }
])
`
- Moyenne des scores par quartier puis trier le score par ordre decroissant
    `db.restaurants.aggregate([
  {
    $unwind: "$grades"
  },
  {
    $group: {
      _id: "$borough",
      avgScore: { $avg: "$grades.score" }
    }
  },
  {
    $sort: { avgScore: -1 }
  }
])
`
- Mise à jour d'un document 
  - Trouver le document `db.restaurants.findOne({ name: "Riviera Caterer" })`
  - Changer le nom `db.restaurants.updateOne(
    { _id: ObjectId("699718746944f0a4c8cd1448") },
    { $set: { name: "Riviera Caterer Updated" } }
    )`
  - Verifier `db.restaurants.find(
  { _id: ObjectId("699718746944f0a4c8cd1448") }
)
`
- Suppression d'un document
  - `db.restaurants.deleteOne(
    { _id: ObjectId("699718746944f0a4c8cd1448") }
    )`
  - `db.restaurants.find(
    { _id: ObjectId("699718746944f0a4c8cd1448") }
    )`


- **Indexation 2dsphere et requêtes géographiques**
  - Création d'un index 2dsphere sur le champ 
  
    `db.cities.createIndex({ location: "2dsphere" })`
  - Récupération des coordonées de Paris, Lyon, Bordeaux
  
    `db.cities.find( { name: "Paris" }, { name: 1, location: 1, _id: 0 } )`
    `db.cities.find( { name: "Lyon" }, { name: 1, location: 1, _id: 0 } )`
    `db.cities.find( { name: "Bordeaux" }, { name: 1, location: 1, _id: 0 } )`
  - Villes autour de Paris dans un rayon de 100km
  
    `db.cities.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [ 2.3522, 48.8566 ] // Paris
      },
      $maxDistance: 100000 // 100 km en mètres
    }
  }
})
`
  - Somme des populations de cette zone
  
    `db.cities.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [ 2.3522, 48.8566 ] },
      distanceField: "dist",
      maxDistance: 100000,
      spherical: true
    }
  },
  {
    $group: {
      _id: null,
      totalPopulation: { $sum: "$population" }
    }
  }
])
`
  - Villes comprises dans le triangle Paris-Lyon-Bordeaux
  
    `db.cities.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [ 2.3522, 48.8566 ],   // Paris
          [ 4.8357, 45.7640 ],   // Lyon
          [ -0.5792, 44.8378 ],  // Bordeaux
          [ 2.3522, 48.8566 ]    // Retour au point de départ
        ]]
      }
    }
  }
})
`
  - Vérification de l'existance des villes hors France dans le résultat
  
    `db.cities.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [ 2.3522, 48.8566 ],
          [ 4.8357, 45.7640 ],
          [ -0.5792, 44.8378 ],
          [ 2.3522, 48.8566 ]
        ]]
      }
    }
  },
  country: { $ne: "France" }
})
`

## Quelques rappels
Rappel des opérateurs à mobiliser

• $match : filtrer

• $project : choisir / transformer les champs

• $sort : trier

• $group : agréger (compter, moyenne, somme…)

• $unwind : “déplier” une liste en plusieurs documents