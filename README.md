# Recap-FormationJS : react, next.js, node.js et express.js

## Installation

### React
```npm create vite@latest```
### Next.js
```npm create next-app@latest```
### Node.js
```npm init```
### Express.js
```npm init```
```npm i express```

## Dépendances

### React
- Jotai
- React-Router
- (Sass)
- ...

### Next.js
- 

### Node.js
- nodemon (dev)
- nanoid

### Express.js
- nodemon (dev)
- Router
- morgan("tiny") (Middleware init server)
- jsonwebtoken (Gestion des tokens)
- argon2 (Hash mdp)
- sequelize (connexion DB SQL) ou mangoose (connnexion DB non relationnel)
- uuid (generation d'id)
- multer

## Package.json → scripts
```
"scripts": {
"test": "echo \"Error: no test specified\" && exit 1",
"dev": "nodemon --env-file=.env src/app.js",
"start": "node --env-file-if-exists=.env src/app.js",
"init-db": "node --env-file=.env src/init-db.js"
}
```
