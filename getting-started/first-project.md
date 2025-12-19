# Premier projet

## Créer votre premier projet

Maintenant que l'installation et la configuration sont terminées, créons votre premier projet !

## Initialisation du projet

Créez un nouveau répertoire et initialisez votre projet :

```bash
mkdir mon-premier-projet
cd mon-premier-projet
mon-application create
```

## Structure du projet

Après l'initialisation, votre projet aura cette structure :

```
mon-premier-projet/
├── src/
│   ├── index.js
│   ├── components/
│   └── utils/
├── public/
│   └── assets/
├── tests/
│   └── index.test.js
├── config.json
├── package.json
└── README.md
```

## Éditer le fichier principal

Ouvrez `src/index.js` et ajoutez ce code :

```javascript
const { Application } = require('mon-application');

const app = new Application({
  name: 'Mon Premier Projet',
  version: '1.0.0'
});

app.on('ready', () => {
  console.log('Application prête !');
  app.run();
});

app.on('error', (err) => {
  console.error('Erreur:', err);
});

app.initialize();
```

## Lancer le projet

Pour démarrer votre application :

```bash
npm start
```

Vous devriez voir :

```
✓ Application initialisée
✓ Configuration chargée
✓ Application prête !
→ Serveur démarré sur http://localhost:3000
```

## Votre première fonctionnalité

Ajoutons une route simple. Créez `src/routes.js` :

```javascript
module.exports = {
  routes: [
    {
      path: '/',
      handler: (req, res) => {
        res.json({ message: 'Bonjour le monde !' });
      }
    },
    {
      path: '/api/status',
      handler: (req, res) => {
        res.json({
          status: 'ok',
          timestamp: Date.now()
        });
      }
    }
  ]
};
```

## Tester votre application

Ouvrez votre navigateur et accédez à `http://localhost:3000`. Vous devriez voir :

```json
{
  "message": "Bonjour le monde !"
}
```

## Mode développement

Pour le développement avec rechargement automatique :

```bash
npm run dev
```

## Prochaines étapes

Félicitations ! Vous avez créé votre premier projet.

Pour aller plus loin :
- Consultez les [tutoriels](../tutorials/basic-tutorial.md)
- Explorez la [référence API](../api-reference/introduction.md)
- Découvrez les [bonnes pratiques](../guides/best-practices.md)
