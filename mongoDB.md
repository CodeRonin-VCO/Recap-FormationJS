# Récap MongoDB/Mongoose

## Créer une nouvelle base de données dans Docker
```
docker run --name donkey-mongodb -d -p 27017:27017 mongo
```

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
  .toArray()  → Tous les éléments
  .sort()     → trier
  .skip()     → passer x éléments
  .limit()    → limiter les résultats à x éléments
  .count()    → compter le nombre d'éléments
```
## Connexion avec Mangoose
### Connection locale à mongodb
```
mongodb://localhost:27017
```

### Connection distance à mongodb
```
mongodb://user:password@domain:27017
```

# Exemple de mise en place d'un DB avec mongoose:
## Dans un fichier config/init-db.js
```
import mongoose from 'mongoose';
import models from '../models/index.js';
import fs from "fs";
import path from 'path';
import { fileURLToPath } from 'url';


// ==== Get env data ====
const { DB_DATABASE, DB_USER, DB_PASSWORD, DB_SERVER, DB_PORT } = process.env;


const uri = `mongodb://${DB_USER ? DB_USER + ':' + DB_PASSWORD + '@' : ''}${DB_SERVER}:${DB_PORT}/${DB_DATABASE}`;

// ==== Init mangoose ====
const connectDB = async () => {
    try {
        mongoose.connect(uri)
        console.log(`MongoDB connected`);

    } catch (error) {
        console.log(`Error mongoDB`, error.message);
        process.exit(1);
    }
};

// ==== Synchronize ====
const initDB = async () => {
    try {
        await connectDB();

        const {User, Post} = models;

        // Load fake data (json)
        const __filename = fileURLToPath(import.meta.url);
        const __dirname = path.dirname(__filename);

        const usersPath = path.join(__dirname, "./../data/users.json");
        const postsPath = path.join(__dirname, "./../data/posts.json");

        // Read json files
        const usersRaw = fs.readFileSync(usersPath, "utf-8");
        const postsRaw = fs.readFileSync(postsPath, "utf-8");

        // Parse json files
        const usersData = JSON.parse(usersRaw);
        const postsData = JSON.parse(postsRaw);

        // Insert users if none exists
        const userCount = await User.countDocuments();
        if (userCount === 0) {
            const insertedUsers = await User.insertMany(usersData.users);
            console.log(`Users inserted:`, insertedUsers.map(u => u.email));
        };

        // Insert posts if none exists
        const postCount = await Post.countDocuments();
        if (postCount === 0) {
            const insertedPosts = await Post.insertMany(postsData.posts);
            console.log(`Posts inserted:`, insertedPosts.map(p => p.id));
        }

        console.log(`Database initialization complete`);

        
    } catch (error) {
        console.error("Database initialization error :", error.message);
        process.exit(1);
    };
};

export {
    connectDB,
    initDB
}
```
## Dans le dossier models (schéma)
### index.js
```
// Import db model DB here
import { Post } from "./posts.model.js";
import { User } from "./user.model.js";

export default {
    User,
    Post
};

```
### posts.model.js
```
import mongoose from 'mongoose';

const postSchema = new mongoose.Schema({
    author: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    content: { type: String, required: true },
    images: [{ type: String }],
    likes: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
    comments: [{
        author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        text: String,
        date: { type: Date, default: Date.now }
    }],
    tags: [{ type: String }]
}, { timestamps: true });

export const Post = mongoose.model('Post', postSchema);
```
### users.model.js
```
import mongoose from "mongoose";

const userSchema = new mongoose.Schema({
    firstname: { type: String, required: true },
    lastname: { type: String, required: true },
    email: {
        type: String,
        required: true,
        unique: true,
        validate: {
            validator: (v) => /^\S+@\S+\.\S+$/.test(v),
            message: props => `${props.value} n'est pas un email valide !`
        }
    },
    password: { type: String, required: true },
    avatar: { type: String }, // URL ou nom de fichier
    bio: { type: String },
    friends: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }]
}, { timestamps: true });


export const User = mongoose.model("User", userSchema);
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








