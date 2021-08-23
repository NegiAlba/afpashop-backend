# Backend Strapi

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

## Déploiement sur Heroku