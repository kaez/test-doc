# Questions fréquentes (FAQ)

## Général

### Qu'est-ce que cette application ?

Cette application est une plateforme de démonstration conçue pour tester et illustrer les capacités de GitBook. Elle offre un ensemble complet de fonctionnalités pour la gestion de données, l'intégration d'API et bien plus encore.

### Est-ce gratuit ?

Cette documentation de test est fournie à titre d'exemple uniquement. Pour les informations sur les tarifs de l'application réelle, consultez notre page de tarification.

### Quelles sont les prérequis système ?

- **Node.js** : Version 16 ou supérieure
- **npm** : Version 7 ou supérieure
- **Système d'exploitation** : Linux, macOS, ou Windows
- **Mémoire** : Minimum 2 GB RAM
- **Espace disque** : 500 MB disponible

## Installation et configuration

### Comment installer l'application ?

```bash
npm install -g mon-application
```

Pour plus de détails, consultez le [guide d'installation](getting-started/installation.md).

### L'installation échoue avec une erreur de permissions

Sous Linux/macOS, utilisez :

```bash
sudo npm install -g mon-application
```

Ou configurez npm pour installer sans sudo :

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Comment mettre à jour vers la dernière version ?

```bash
npm update -g mon-application
```

Vérifiez la version installée :

```bash
mon-application --version
```

### Où se trouve le fichier de configuration ?

Par défaut, le fichier de configuration se trouve à :

- **Linux/macOS** : `~/.config/mon-application/config.json`
- **Windows** : `%APPDATA%\mon-application\config.json`

Vous pouvez spécifier un chemin personnalisé :

```bash
mon-application --config /path/to/config.json
```

## Utilisation

### Comment créer mon premier projet ?

```bash
mkdir mon-projet
cd mon-projet
mon-application init
```

Consultez le guide [Premier projet](getting-started/first-project.md) pour plus de détails.

### Puis-je utiliser l'application avec TypeScript ?

Oui ! L'application supporte TypeScript nativement. Créez un projet TypeScript :

```bash
mon-application init --typescript
```

### Comment migrer depuis la version 0.x ?

Consultez notre [guide de migration](changelog.md#migration-depuis-0x) pour les instructions détaillées.

### L'application est lente, que faire ?

Plusieurs optimisations possibles :

1. **Activer le cache** :
```json
{
  "cache": {
    "enabled": true,
    "ttl": 3600
  }
}
```

2. **Augmenter les resources** :
```json
{
  "performance": {
    "workers": 4,
    "maxMemory": "2GB"
  }
}
```

3. Consultez le guide [Performance](guides/performance.md)

## API

### Comment obtenir une clé API ?

1. Créez un compte sur https://dashboard.example.com
2. Allez dans "Settings" > "API Keys"
3. Cliquez sur "Generate New Key"
4. Copiez et sauvegardez votre clé

Voir [Authentification](api-reference/authentication.md) pour plus de détails.

### Quelle est la limite de taux de l'API ?

- **Clés de test** : 100 requêtes/jour
- **Clés de production** : 10,000 requêtes/heure
- **OAuth tokens** : 1,000 requêtes/heure

Les headers de réponse indiquent vos limites :

```http
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9543
X-RateLimit-Reset: 1642252800
```

### Comment gérer l'expiration du token ?

Implémentez un système de refresh automatique :

```javascript
async function apiCall(url, options = {}) {
  try {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Authorization': `Bearer ${getToken()}`,
        ...options.headers
      }
    });

    if (response.status === 401) {
      // Token expiré, rafraîchir
      await refreshToken();
      // Réessayer
      return apiCall(url, options);
    }

    return await response.json();
  } catch (error) {
    console.error('API call failed:', error);
    throw error;
  }
}
```

### L'API retourne une erreur 500, que faire ?

1. Vérifiez le status de l'API : https://status.example.com
2. Consultez votre `requestId` dans la réponse
3. Contactez le support avec le `requestId`

### Puis-je utiliser l'API en local ?

Oui, installez la version locale :

```bash
npm install @example/api-server
mon-application serve --port 3000
```

## Erreurs courantes

### "Module not found"

Réinstallez les dépendances :

```bash
rm -rf node_modules package-lock.json
npm install
```

### "Port already in use"

Changez le port :

```bash
mon-application start --port 3001
```

Ou trouvez et arrêtez le processus utilisant le port :

```bash
# Linux/macOS
lsof -i :3000
kill -9 <PID>

# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

### "Database connection failed"

Vérifiez vos paramètres de connexion :

```json
{
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "myapp",
    "user": "dbuser",
    "password": "dbpass"
  }
}
```

Assurez-vous que la base de données est démarrée :

```bash
# PostgreSQL
sudo service postgresql start

# MongoDB
sudo service mongod start
```

### "Permission denied"

Sous Linux/macOS, vérifiez les permissions du répertoire :

```bash
sudo chown -R $USER:$USER ~/.config/mon-application
chmod -R 755 ~/.config/mon-application
```

## Sécurité

### Où stocker mes clés API en toute sécurité ?

Utilisez des variables d'environnement :

```bash
# .env
API_KEY=sk_live_1234567890abcdef
DATABASE_URL=postgres://user:pass@localhost/db

# .gitignore
.env
```

Chargez-les dans votre application :

```javascript
require('dotenv').config();

const apiKey = process.env.API_KEY;
```

**Ne commitez JAMAIS les clés dans Git !**

### Comment activer HTTPS en développement ?

Générez un certificat auto-signé :

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

Utilisez-le :

```bash
mon-application start --https --key key.pem --cert cert.pem
```

### Mon compte a été compromis, que faire ?

1. **Révoquez immédiatement** toutes vos clés API
2. **Changez votre mot de passe**
3. **Activez l'authentification 2FA**
4. **Contactez le support** : security@example.com
5. **Vérifiez les logs** d'activité suspecte

## Performance

### Comment améliorer les temps de réponse ?

1. **Activez le cache** (voir ci-dessus)
2. **Utilisez la pagination** pour les listes
3. **Optimisez vos requêtes** de base de données
4. **Activez la compression** :

```json
{
  "compression": {
    "enabled": true,
    "level": 6
  }
}
```

Consultez le guide [Performance](guides/performance.md).

### L'utilisation mémoire est élevée

Ajustez la configuration :

```json
{
  "performance": {
    "maxMemory": "1GB",
    "gc": {
      "interval": 60000
    }
  }
}
```

Ou utilisez les options Node.js :

```bash
node --max-old-space-size=1024 app.js
```

## Développement

### Comment contribuer au projet ?

1. Fork le repository
2. Créez une branche : `git checkout -b feature/ma-fonctionnalite`
3. Committez vos changes : `git commit -am 'Ajout fonctionnalité'`
4. Pushez : `git push origin feature/ma-fonctionnalite`
5. Créez une Pull Request

### Comment exécuter les tests ?

```bash
# Tous les tests
npm test

# Tests unitaires uniquement
npm run test:unit

# Tests d'intégration
npm run test:integration

# Avec couverture
npm run test:coverage
```

### Comment déboguer l'application ?

Activez le mode debug :

```bash
DEBUG=* mon-application start
```

Ou utilisez le débogueur Node.js :

```bash
node --inspect app.js
```

Puis connectez Chrome DevTools à `chrome://inspect`.

## Support

### Comment obtenir de l'aide ?

- **Documentation** : https://docs.example.com
- **Forum communautaire** : https://community.example.com
- **GitHub Issues** : https://github.com/example/app/issues
- **Email support** : support@example.com
- **Chat** : https://chat.example.com

### Les temps de réponse du support

- **Email** : 24-48 heures
- **Chat** : En temps réel (heures de bureau)
- **Forum** : Variable (communauté)

### Où signaler un bug de sécurité ?

**Ne créez PAS d'issue publique.**

Envoyez un email à : security@example.com

Nous répondrons sous 48 heures et travaillerons avec vous pour résoudre le problème.

## Ressources

### Où trouver des exemples de code ?

- [Tutoriels](tutorials/basic-tutorial.md)
- [Cas d'usage](tutorials/use-cases.md)
- [GitHub Examples](https://github.com/example/examples)

### Y a-t-il une communauté ?

Oui ! Rejoignez-nous :

- **Discord** : https://discord.gg/example
- **Twitter** : @example_app
- **Reddit** : r/example_app
- **Stack Overflow** : Tag `example-app`

### Où suivre les mises à jour ?

- **Blog** : https://blog.example.com
- **Newsletter** : https://example.com/newsletter
- **Twitter** : @example_app
- **Changelog** : [Voir le changelog](changelog.md)

---

**Votre question n'est pas ici ?**

Consultez la [documentation complète](README.md) ou [contactez le support](#support).
