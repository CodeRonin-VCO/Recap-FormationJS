# Récap MongoDB/Mongoose

## Créer une collection
```
db.createCollection("name of the table/collection");
```
## Changer de DB
```
use('demo2');
db.createCollection('personne');
```
## Supprimer une DB
```
use('demo2');
db.personne.drop();
```
## Insertion d'un schéma
```
db.runCommand({
    collMod: 'produits', 
    validator: {
        $jsonSchema: produitSchema
    }
});
```
## Insertion des données
```
db.collection.insertOne(doc);
insertMany([doc1, doc2]);
```
## Chercher dans la db
### Extraction simple : Tous les documents qui matchent (20premiers)
```
db.movies.find({"filtres"}, {"sélection"});
db.movies.findOne(); // un seul document
```
### Permet de changer l'ordre des éléments (projection, filtre, etc.).
```
db.movies.aggregate([ // step1 // step2 // step3 //... ]);
```
## __Vérification__
### Est égal à
  {$eq: //...}
### Est égal à une RegEx
  { title: /forrest gump/i}
### Comparateurs
  $lt(<), $lte (<=), $gt (>), $gte (>=)
### Entre
  {$gt: 1990, $lt: 2000}
### Soit...soit...
  {$in: [1990, 2000, 2010]}
### Et
  $and: [
            { year: 1994 },
            { title: /^a/i }
        ]
### Ou
  $or: [
            { year: 1994 },
            { title: /^a/i }
        ]
### Expressions dynamiques
```
$expr: { $gt: [ {$size: "$actors"}, 5] }
```
### Ventiler avec aggregate
```
$unwind:
```
### Grouper avec aggregate
```
$group:
```
### Infos utiles:
```
// *MongoDB peut aussi rechercher directement les informations dans les tableaux 
// * genre:'Action' strictement
// * genre: ['Action', 'Adventure'] strictement ces termes
// * $and: [ { genre: 'Action' }, {genre: 'Adventure'}] contient (non strict)
```
## __Méthodes__
```
db.movies.find({"filtres"}, {"sélection"})
  .toArray() → Tous les éléments
  .sort() → trier
  .skip() → passer x éléments
  .limit() → limiter les résultats à x éléments
  .count() → compter le nombre d'éléments
```
## Connexion avec Mangoose
Exemple:
```
// == Importer la BD
import Mongoose from "Mongoose";

// == Connection à la BD de mongo
Mongoose.connect('mongodb://localhost/demo');

// == Définir le schéma
const MoviesSchema = Mongoose.Schema({
    title: String,
    year: Number,
    score: Number,
    genre: [String],
    actor: [String]
});

// == Créer un modèle
const Movies = new Mongoose.model('movies', MoviesSchema);

// == Faire des requêtes
// const movies = await Movies.find({ year:1994 }, { title: 1 });
const moviesByGenre = await Movies.aggregate([
    {
        $unwind: '$genre'
    },
    {
        $group: { _id: '$genre', titles: {$push: '$title'}}
    }
])

console.log(moviesByGenre.map(g => ({ genre: g._id, count: g.titles.length })));
```

# Exemple Schéma
```
{
    "type": "object",
    "required": ["nomProduit", "prix", "reference", "categorie", "tags", "commentaires"],
    "properties": {
        "nomProduit": { "type": "string", "minLength": 3, "maxLength": 100 },
        "prix": { "type": "number", "multipleOf": 0.01, "exclusiveMinimum": 0, "maximum": 9999 },
        "reference": { "type": "string", "minLength": 10, "maxLength": 10, "pattern": "^\\d{10}$" },
        "categorie": { "enum": ["Boisson", "Viande", "Légumes"] },
        "tags": { "type": "array", "items": {"type": "string", "maxLength": 255}, "maxItems": 10, "uniqueItems": true },
        "commentaires": { 
            "type": "array", 
            "prefixItems": [
                { "type": "string" },
                { "type": "string" },
                { "type": "string" }
            ]
        }
    }
}
```
# Exemple insertion de données
```
db.produits.insertOne({
    "nomProduit": "Coca Cola",
    "prix": 1.20,
    "reference": "1234567891",
    "categorie": "Boisson",
    "tags": ["Soda", "Sucre", "Caféine"],
    "commentaires": [
        { "auteur": "Khun LY", "date": "1982-01-01", "message": "TROP de sucre"},
        { "auteur": "M. Jordan", "date": "1982-01-02", "message": "Le coca c'est la vie"}
    ]
});
```








