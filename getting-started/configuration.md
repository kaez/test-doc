# Configuration

## Configuration initiale

Après l'installation, vous devez configurer l'application. Créez un fichier de configuration :

```bash
mon-application init
```

Cette commande crée un fichier `config.json` dans votre répertoire courant.

## Structure du fichier de configuration

Voici un exemple de fichier de configuration :

```json
{
  "version": "1.0.0",
  "settings": {
    "language": "fr",
    "theme": "dark",
    "autoSave": true,
    "notifications": {
      "enabled": true,
      "sound": false
    }
  },
  "api": {
    "endpoint": "https://api.example.com",
    "timeout": 5000,
    "retries": 3
  }
}
```

## Options de configuration

### Paramètres généraux

| Option | Type | Défaut | Description |
|--------|------|--------|-------------|
| `language` | string | "en" | Langue de l'interface |
| `theme` | string | "light" | Thème de l'application |
| `autoSave` | boolean | false | Sauvegarde automatique |

### Paramètres API

| Option | Type | Défaut | Description |
|--------|------|--------|-------------|
| `endpoint` | string | - | URL de l'API |
| `timeout` | number | 3000 | Timeout en ms |
| `retries` | number | 1 | Nombre de tentatives |

## Variables d'environnement

Vous pouvez également utiliser des variables d'environnement :

```bash
export APP_API_KEY="votre-clé-api"
export APP_DEBUG="true"
export APP_LOG_LEVEL="info"
```

## Configuration avancée

Pour des configurations plus avancées, consultez le fichier `config.advanced.json` :

```json
{
  "cache": {
    "enabled": true,
    "ttl": 3600,
    "storage": "redis",
    "redis": {
      "host": "localhost",
      "port": 6379
    }
  },
  "logging": {
    "level": "info",
    "format": "json",
    "output": "file",
    "file": "./logs/app.log"
  }
}
```

## Prochaines étapes

Maintenant que votre application est configurée, créez votre [premier projet](first-project.md).
