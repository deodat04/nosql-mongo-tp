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
- Score inférieur id 
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
- Récupérer uniquement les restaurants qui n'ont pas de score 
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
- 