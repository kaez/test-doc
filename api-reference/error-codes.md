# Codes d'erreur

## Format des erreurs

Toutes les erreurs suivent ce format standardisé :

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Description de l'erreur",
    "details": {
      "field": "champ concerné",
      "value": "valeur invalide"
    }
  },
  "meta": {
    "timestamp": "2025-01-15T10:30:00Z",
    "requestId": "req_abc123",
    "documentation": "https://docs.example.com/errors/ERROR_CODE"
  }
}
```

## Codes HTTP

### 4xx - Erreurs client

#### 400 Bad Request

La requête est mal formée ou contient des paramètres invalides.

| Code | Message | Résolution |
|------|---------|------------|
| `INVALID_REQUEST` | Requête invalide | Vérifiez la structure de votre requête |
| `MISSING_PARAMETER` | Paramètre requis manquant | Ajoutez le paramètre manquant |
| `INVALID_PARAMETER` | Paramètre invalide | Corrigez la valeur du paramètre |
| `INVALID_JSON` | JSON mal formé | Vérifiez la syntaxe de votre JSON |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "MISSING_PARAMETER",
    "message": "Le paramètre 'email' est requis",
    "details": {
      "parameter": "email",
      "location": "body"
    }
  }
}
```

#### 401 Unauthorized

Problème d'authentification.

| Code | Message | Résolution |
|------|---------|------------|
| `MISSING_TOKEN` | Token d'authentification manquant | Ajoutez le header Authorization |
| `INVALID_TOKEN` | Token invalide ou expiré | Générez un nouveau token |
| `EXPIRED_TOKEN` | Token expiré | Rafraîchissez votre token |
| `REVOKED_TOKEN` | Token révoqué | Générez un nouveau token |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "EXPIRED_TOKEN",
    "message": "Votre token d'authentification a expiré",
    "details": {
      "expiredAt": "2025-01-15T09:30:00Z",
      "tokenId": "tok_abc123"
    }
  }
}
```

#### 403 Forbidden

Accès interdit à la ressource.

| Code | Message | Résolution |
|------|---------|------------|
| `INSUFFICIENT_SCOPE` | Scopes insuffisants | Demandez les scopes nécessaires |
| `FORBIDDEN` | Accès interdit | Vérifiez vos permissions |
| `ACCOUNT_SUSPENDED` | Compte suspendu | Contactez le support |
| `IP_BLOCKED` | Adresse IP bloquée | Utilisez une IP autorisée |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_SCOPE",
    "message": "Scope 'write' requis pour cette opération",
    "details": {
      "requiredScopes": ["write"],
      "currentScopes": ["read"]
    }
  }
}
```

#### 404 Not Found

Ressource introuvable.

| Code | Message | Résolution |
|------|---------|------------|
| `NOT_FOUND` | Ressource non trouvée | Vérifiez l'ID de la ressource |
| `ENDPOINT_NOT_FOUND` | Endpoint inexistant | Vérifiez l'URL de l'endpoint |
| `USER_NOT_FOUND` | Utilisateur non trouvé | Vérifiez l'ID utilisateur |
| `POST_NOT_FOUND` | Publication non trouvée | Vérifiez l'ID de la publication |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "L'utilisateur avec l'ID 'user_123' n'existe pas",
    "details": {
      "resourceType": "user",
      "resourceId": "user_123"
    }
  }
}
```

#### 422 Unprocessable Entity

Erreur de validation.

| Code | Message | Résolution |
|------|---------|------------|
| `VALIDATION_ERROR` | Erreur de validation | Corrigez les champs invalides |
| `INVALID_EMAIL` | Email invalide | Fournissez un email valide |
| `PASSWORD_TOO_WEAK` | Mot de passe trop faible | Utilisez un mot de passe plus fort |
| `DUPLICATE_ENTRY` | Entrée dupliquée | La ressource existe déjà |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Erreurs de validation",
    "details": {
      "errors": [
        {
          "field": "email",
          "message": "Format d'email invalide",
          "value": "invalid-email"
        },
        {
          "field": "age",
          "message": "Doit être supérieur à 18",
          "value": 15
        }
      ]
    }
  }
}
```

#### 429 Too Many Requests

Limite de taux dépassée.

| Code | Message | Résolution |
|------|---------|------------|
| `RATE_LIMIT_EXCEEDED` | Limite de taux dépassée | Attendez avant de réessayer |
| `QUOTA_EXCEEDED` | Quota dépassé | Augmentez votre quota |
| `CONCURRENT_LIMIT` | Trop de requêtes simultanées | Réduisez la concurrence |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Limite de 1000 requêtes/heure dépassée",
    "details": {
      "limit": 1000,
      "remaining": 0,
      "resetAt": "2025-01-15T11:00:00Z",
      "retryAfter": 1800
    }
  }
}
```

### 5xx - Erreurs serveur

#### 500 Internal Server Error

Erreur interne du serveur.

| Code | Message | Résolution |
|------|---------|------------|
| `INTERNAL_ERROR` | Erreur serveur interne | Réessayez plus tard |
| `DATABASE_ERROR` | Erreur de base de données | Contactez le support |
| `UNKNOWN_ERROR` | Erreur inconnue | Contactez le support |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Une erreur interne s'est produite",
    "details": {
      "requestId": "req_abc123",
      "timestamp": "2025-01-15T10:30:00Z"
    }
  },
  "meta": {
    "support": "Contactez api-support@example.com avec le requestId"
  }
}
```

#### 503 Service Unavailable

Service temporairement indisponible.

| Code | Message | Résolution |
|------|---------|------------|
| `SERVICE_UNAVAILABLE` | Service indisponible | Réessayez plus tard |
| `MAINTENANCE_MODE` | Mode maintenance | Consultez status.example.com |
| `UPSTREAM_TIMEOUT` | Timeout du service en amont | Réessayez avec un timeout plus long |

**Exemple** :

```json
{
  "success": false,
  "error": {
    "code": "MAINTENANCE_MODE",
    "message": "Service en maintenance programmée",
    "details": {
      "startedAt": "2025-01-15T10:00:00Z",
      "estimatedEnd": "2025-01-15T12:00:00Z",
      "status": "https://status.example.com"
    }
  }
}
```

## Erreurs métier

### Utilisateurs

| Code | Message | HTTP |
|------|---------|------|
| `USER_ALREADY_EXISTS` | Email déjà utilisé | 422 |
| `INVALID_CREDENTIALS` | Identifiants invalides | 401 |
| `ACCOUNT_NOT_VERIFIED` | Compte non vérifié | 403 |
| `ACCOUNT_LOCKED` | Compte verrouillé | 403 |

### Publications

| Code | Message | HTTP |
|------|---------|------|
| `POST_ALREADY_PUBLISHED` | Publication déjà publiée | 422 |
| `CANNOT_DELETE_PUBLISHED` | Impossible de supprimer une publication publiée | 422 |
| `SLUG_ALREADY_EXISTS` | Slug déjà utilisé | 422 |

### Fichiers

| Code | Message | HTTP |
|------|---------|------|
| `FILE_TOO_LARGE` | Fichier trop volumineux | 413 |
| `INVALID_FILE_TYPE` | Type de fichier non supporté | 422 |
| `STORAGE_QUOTA_EXCEEDED` | Quota de stockage dépassé | 422 |
| `VIRUS_DETECTED` | Virus détecté dans le fichier | 422 |

## Gestion des erreurs

### Retry automatique

Pour certaines erreurs, un retry automatique est recommandé :

```javascript
async function retryRequest(fn, maxAttempts = 3) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      const shouldRetry = [
        'INTERNAL_ERROR',
        'SERVICE_UNAVAILABLE',
        'UPSTREAM_TIMEOUT'
      ].includes(error.code);

      if (!shouldRetry || attempt === maxAttempts) {
        throw error;
      }

      // Backoff exponentiel
      const delay = Math.pow(2, attempt) * 1000;
      await sleep(delay);
    }
  }
}
```

### Gestion côté client

```javascript
try {
  const response = await fetch('https://api.example.com/v1/users', {
    headers: { 'Authorization': `Bearer ${token}` }
  });

  if (!response.ok) {
    const error = await response.json();

    switch (error.error.code) {
      case 'EXPIRED_TOKEN':
        // Rafraîchir le token
        await refreshToken();
        return retry();

      case 'RATE_LIMIT_EXCEEDED':
        // Attendre avant de réessayer
        const retryAfter = error.error.details.retryAfter;
        await sleep(retryAfter * 1000);
        return retry();

      case 'VALIDATION_ERROR':
        // Afficher les erreurs de validation
        displayValidationErrors(error.error.details.errors);
        break;

      default:
        // Erreur générique
        console.error('Erreur API:', error.error.message);
    }
  }

  return await response.json();
} catch (error) {
  console.error('Erreur réseau:', error);
}
```

## Logging et debugging

Utilisez le `requestId` pour le support :

```bash
curl https://api.example.com/v1/users/123 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-Request-ID: my-custom-request-id"
```

Le `requestId` sera retourné dans la réponse et les logs serveur.

## Status page

Consultez https://status.example.com pour :
- État des services en temps réel
- Incidents en cours
- Maintenances programmées
- Historique des incidents

## Support

En cas d'erreur persistante :

1. Consultez cette documentation
2. Vérifiez https://status.example.com
3. Contactez le support avec :
   - Le code d'erreur
   - Le requestId
   - La date/heure de l'erreur
   - Les étapes pour reproduire

Email : api-support@example.com

## Prochaines étapes

- Retournez aux [endpoints](endpoints.md)
- Consultez l'[authentification](authentication.md)
- Explorez les exemples d'intégration
