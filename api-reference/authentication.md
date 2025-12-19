# Authentification

## Vue d'ensemble

L'API utilise des tokens Bearer pour l'authentification. Chaque requête doit inclure un token valide dans le header `Authorization`.

## Obtenir un token

### 1. Créer un compte API

Créez un compte sur le dashboard : https://dashboard.example.com

### 2. Générer une clé API

Depuis le dashboard, générez une nouvelle clé API :

```bash
curl -X POST https://api.example.com/v1/auth/keys \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Production Key",
    "scopes": ["read", "write"]
  }'
```

Réponse :

```json
{
  "success": true,
  "data": {
    "id": "key_abc123",
    "name": "Production Key",
    "key": "sk_live_1234567890abcdef",
    "scopes": ["read", "write"],
    "createdAt": "2025-01-15T10:30:00Z"
  }
}
```

**Important** : Conservez votre clé en sécurité. Elle ne sera affichée qu'une seule fois.

## Utiliser le token

Incluez le token dans le header `Authorization` de chaque requête :

```http
GET /v1/users HTTP/1.1
Host: api.example.com
Authorization: Bearer sk_live_1234567890abcdef
```

### Exemple avec cURL

```bash
curl https://api.example.com/v1/users \
  -H "Authorization: Bearer sk_live_1234567890abcdef"
```

### Exemple avec JavaScript

```javascript
const response = await fetch('https://api.example.com/v1/users', {
  headers: {
    'Authorization': 'Bearer sk_live_1234567890abcdef',
    'Content-Type': 'application/json'
  }
});

const data = await response.json();
```

### Exemple avec Python

```python
import requests

headers = {
    'Authorization': 'Bearer sk_live_1234567890abcdef',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/v1/users', headers=headers)
data = response.json()
```

## Types de clés

### Clés de production

Format : `sk_live_...`

Utilisez ces clés dans votre environnement de production.

### Clés de test

Format : `sk_test_...`

Utilisez ces clés pour le développement et les tests. Elles sont limitées à 100 requêtes par jour.

## Scopes (portées)

Les clés API peuvent avoir différents scopes :

| Scope | Description |
|-------|-------------|
| `read` | Lecture seule |
| `write` | Création et modification |
| `delete` | Suppression de ressources |
| `admin` | Accès administrateur complet |

Exemple de requête avec scopes limités :

```bash
curl -X POST https://api.example.com/v1/auth/keys \
  -H "Authorization: Bearer MASTER_KEY" \
  -d '{
    "name": "Read Only Key",
    "scopes": ["read"]
  }'
```

## OAuth 2.0

Pour les applications qui accèdent à l'API au nom des utilisateurs, utilisez OAuth 2.0.

### Flow Authorization Code

#### 1. Rediriger l'utilisateur

```
https://api.example.com/oauth/authorize?
  client_id=YOUR_CLIENT_ID&
  redirect_uri=https://yourapp.com/callback&
  response_type=code&
  scope=read write
```

#### 2. Échanger le code contre un token

```bash
curl -X POST https://api.example.com/oauth/token \
  -H "Content-Type: application/json" \
  -d '{
    "grant_type": "authorization_code",
    "code": "AUTHORIZATION_CODE",
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET",
    "redirect_uri": "https://yourapp.com/callback"
  }'
```

Réponse :

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "def50200abc...",
  "scope": "read write"
}
```

#### 3. Utiliser le token

```http
GET /v1/users/me HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

#### 4. Rafraîchir le token

```bash
curl -X POST https://api.example.com/oauth/token \
  -H "Content-Type: application/json" \
  -d '{
    "grant_type": "refresh_token",
    "refresh_token": "def50200abc...",
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET"
  }'
```

## JWT (JSON Web Tokens)

Pour les intégrations serveur-à-serveur, vous pouvez générer vos propres JWT.

### Génération du JWT

```javascript
const jwt = require('jsonwebtoken');

const token = jwt.sign(
  {
    sub: 'user_123',
    iss: 'your-app',
    aud: 'api.example.com',
    exp: Math.floor(Date.now() / 1000) + (60 * 60), // 1 heure
    scope: ['read', 'write']
  },
  YOUR_SECRET_KEY,
  { algorithm: 'HS256' }
);
```

### Utilisation

```http
GET /v1/users HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

## Sécurité

### Bonnes pratiques

✅ **À faire** :
- Stockez les clés dans des variables d'environnement
- Utilisez HTTPS pour toutes les requêtes
- Limitez les scopes aux besoins minimaux
- Renouvelez régulièrement les clés
- Révoquez immédiatement les clés compromises

❌ **À éviter** :
- Ne commitez jamais les clés dans Git
- N'exposez pas les clés côté client
- N'utilisez pas la même clé partout
- Ne partagez pas les clés par email

### Rotation des clés

Renouvelez vos clés régulièrement :

```bash
# Créer une nouvelle clé
curl -X POST https://api.example.com/v1/auth/keys \
  -H "Authorization: Bearer OLD_KEY" \
  -d '{"name": "New Key", "scopes": ["read", "write"]}'

# Révoquer l'ancienne clé
curl -X DELETE https://api.example.com/v1/auth/keys/OLD_KEY_ID \
  -H "Authorization: Bearer NEW_KEY"
```

### Révocation

Pour révoquer une clé :

```bash
curl -X DELETE https://api.example.com/v1/auth/keys/key_abc123 \
  -H "Authorization: Bearer MASTER_KEY"
```

## Erreurs d'authentification

| Code | Message | Description |
|------|---------|-------------|
| 401 | `INVALID_TOKEN` | Token invalide ou expiré |
| 401 | `MISSING_TOKEN` | Token manquant |
| 403 | `INSUFFICIENT_SCOPE` | Scope insuffisant |
| 403 | `REVOKED_TOKEN` | Token révoqué |

Exemple de réponse d'erreur :

```json
{
  "success": false,
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Le token fourni est invalide ou expiré",
    "details": {
      "expiredAt": "2025-01-15T09:30:00Z"
    }
  }
}
```

## Limitations

- **Clés de test** : 100 requêtes/jour
- **Clés de production** : 10,000 requêtes/heure
- **OAuth tokens** : 1,000 requêtes/heure par utilisateur
- **Durée de vie JWT** : Maximum 24 heures

## Prochaines étapes

- Consultez les [endpoints disponibles](endpoints.md)
- Comprenez les [codes d'erreur](error-codes.md)
- Explorez les exemples d'intégration
