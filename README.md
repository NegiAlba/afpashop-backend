<p align="center">
  <a href="https://strapi.io">
    <img src="https://strapi.io/assets/strapi-logo-dark.svg" width="318px" alt="Strapi logo" />
  </a>
</p>

<p align="center">
    <a aria-label="Magic" href="https://magic.link/">
        <img src="https://svgshare.com/i/_U9.svg" height="60px" alt="Magic.link logo" />
    </a>
    <a aria-label="Cloudinary" href="https://cloudinary.com/">
        <img src="https://svgshare.com/i/_V7.svg" height="60px" alt="Cloudinary logo logo" />
    </a>
    <a aria-label="PostgresQL" href="https://www.postgresql.org/">
        <img src="https://upload.wikimedia.org/wikipedia/commons/2/29/Postgresql_elephant.svg" height="60px" alt="PostgresQL logo" />
    </a>
</p>


# Backend réalisé avec Strapi, Magic et PostgresQL
Pour le projet e-commerce avec Next, nous utilisons un back-end réalisé avec Strapi, avec une connexion gérée par [Magic](https://magic.link/ "Lien vers le site Magic), et un paiement géré avec [Stripe](https://stripe.com/en-fr).
Le backend sera hébergé sur Heroku, et utilisera donc une base de données en PostgresQL en production.

## Dépendances du projet

Pour ce projet nous utiliserons donc le quickstarter de Strapi, et nous installerons ensuite quelques packages à l'aide d'npm (ou de yarn, c'est vous qui décidez :D ).

- strapi-provider-upload-cloudinary
- strapi-plugin-magic
- pg
- pg-connection-string


## Démarrage du back-end Strapi.

Pour créer le back-end strapi, il conviendra d'utiliser le générateur de projets strapi.
`npx create-strapi-app afpashop-backend --quickstart`

Et de se créer plusieurs collections (des tables qui vont contenir les données de notre application). Nous aurons besoin d'une table pour les produits, et une autre pour l'historique des achats.

    - Product
        - name (Text, Short text)
        - image (Media, Single media)
        - content (Rich text)
        - meta_description (Text, Long text)
        - meta_title (Text, Long text)
        - price (Number, decimal)
        - slug (UID, related to name)
    
    - Order
        - status (Enumeration,paid|unpaid)
        - total (Number,decimal)
        - checkout_session (Text, short text)
        - order_products (Relation,Product - has many - Order)
        - user (Relation, User (users-permissions - has many - Order)

Il ne faudra pas oublier de rendre les collections disponibles au public pour count, find, et findOne dans les réglages. (Settings > Users & Permissions Plugin Roles > Public > (cocher) count, findone et find. Faire de même pour le rôle Authenticated).

## Cloudinary (stockage des photos)

Nous allons héberger notre projet chez Heroku qui ne possède pas de système de conservation de fichiers permanents, par conséquent il faudra les enregistrer sur un service externe (comme Cloudinary, AWS S3 ...) pour ensuite les récupérer. Ici nous utiliserons Cloudinary.

Pour activer Cloudinary en tant que serveur d'upload, il faudra configurer le plugin de cloudinary que l'on a installé avec npm auparavant.

On créer un fichier plugins.js dans le dossier config. [Il contiendra les instructions suivantes](./config/plugins.js).

Dans un fichier .env à la racine, nous ajouterons l'ensemble de ces variables avec les informations accessibles depuis le dashboard Cloudinary.

## Authentification avec Magic

Magic étant un service externe, il faudra aussi configurer les requêtes à l'aide de son outil. Pour cela on récupèrera le fichier de permissions de base [disponible ici] (https://github.com/strapi/strapi/blob/master/packages/strapi-plugin-users-permissions/config/policies/permissions.js)

Remplacer la ligne 26 avec une nouvelle instruction qui permettra de contenir l'authentification avec Magic.

```
    try{
        await strapi.plugins['magic'].services['magic'].loginWithMagic(ctx)
    } catch (err) {
        return handleErrors(ctx, err, 'unauthorized');
    }
```

Dans le fichier ENV, rajouter une ligne MAGIC_KEY avec la Publishable API Key de votre Dashboard Magic.

Enfin, il faudra recharger votre Tableau de bord administrateur avec un `npm run build`.

## Gestion de la base de données

Pour la base de données, il conviendra d'initialiser un gestionnaire de base de données, puisque ce ne sera pas la même base en local et en remote (une fois déployé sur Heroku).

Il faudra créer un fichier database.js qui se situera au chemin config/database.js

Ajouter [les instructions suivantes](./config/database.js).

Cela vous permettra d'initialiser la connection en Postgres en remote et en SQLite en local.

Toutes les données seront isolées entre le local et le remote, comme il convient de faire d'ailleurs.

## Déploiement sur Heroku

Installer le CLI Heroku selon votre machine.
Se connecter à l'aide de votre interface de ligne de commande avec la commande `heroku login`
Créer un projet avec `heroku create`
Ajouter l'add-on PostgresQL afin de l'utiliser sur heroku avec la commande `heroku addons:create heroku-postgresql:hobby-dev`

Enfin initialiser votre repository (si ce n'est pas déja fait) et commit les changements sur votre repository avant le push de votre backend.
```
git add .
git commit -m "Commit before deploy on Heroku"
```

Enfin déployer votre projet avec la commande `git push heroku master` (ou main, en fonction du nom de votre branche principale).

**Attention votre projet ne fonctionne pas encore**

Il faudra rajouter les variables d'environnement présente dans votre ENV local afin de pouvoir utiliser le projet. 
En premier lieu, il conviendra de rajouter la variable d'environnement `NODE_ENV` et de la définir sur `production`.
Remarquez qu'il existe déja une variable d'environnement liée à la base de données, N'Y TOUCHEZ PAS. Heroku gère la connexion à la base de données que vous avez ajouté à l'aide de l'add-on ajouté précédemment.

Vous pouvez désormais configurer votre projet en remote comme vous l'avez fait en local (routes de l'API et éventuels produits)

