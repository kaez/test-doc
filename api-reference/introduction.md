# Introduction à l'API

## Vue d'ensemble

Bienvenue dans la documentation de référence de l'API. Cette section détaille toutes les endpoints, méthodes et paramètres disponibles.

## URL de base

```
Production: https://api.example.com/v1
Staging:    https://staging-api.example.com/v1
Dev:        http://localhost:3000/v1
```

## Versioning

L'API utilise le versioning sémantique (SemVer). La version actuelle est `v1`.

Les versions sont spécifiées dans l'URL :
```
https://api.example.com/v1/resource
```

## Formats de données

### Requêtes

L'API accepte les données au format JSON. Assurez-vous d'inclure le header :

```http
Content-Type: application/json
```

Exemple de requête :

```http
POST /v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "name": "Jean Dupont",
  "email": "jean@example.com"
}
```

### Réponses

Toutes les réponses sont retournées au format JSON avec le header :

```http
Content-Type: application/json; charset=utf-8
```

#### Réponse de succès

```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "name": "Jean Dupont",
    "email": "jean@example.com",
    "createdAt": "2025-01-15T10:30:00Z"
  },
  "meta": {
    "timestamp": "2025-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

#### Réponse d'erreur

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email invalide",
    "details": {
      "field": "email",
      "value": "invalid-email"
    }
  },
  "meta": {
    "timestamp": "2025-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

## Codes de statut HTTP

L'API utilise les codes de statut HTTP standard :

| Code | Signification | Description |
|------|---------------|-------------|
| 200 | OK | Requête réussie |
| 201 | Created | Ressource créée avec succès |
| 204 | No Content | Requête réussie sans contenu de réponse |
| 400 | Bad Request | Requête invalide |
| 401 | Unauthorized | Authentification requise |
| 403 | Forbidden | Accès interdit |
| 404 | Not Found | Ressource non trouvée |
| 422 | Unprocessable Entity | Validation échouée |
| 429 | Too Many Requests | Limite de taux dépassée |
| 500 | Internal Server Error | Erreur serveur |
| 503 | Service Unavailable | Service temporairement indisponible |

## Pagination

Les endpoints qui retournent des listes supportent la pagination :

```http
GET /v1/users?page=2&limit=50
```

Paramètres :
- `page` : Numéro de page (défaut : 1)
- `limit` : Nombre d'éléments par page (défaut : 20, max : 100)

Réponse :

```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 50,
    "total": 250,
    "totalPages": 5,
    "hasNext": true,
    "hasPrevious": true
  }
}
```

## Filtrage

Vous pouvez filtrer les résultats avec des paramètres de query :

```http
GET /v1/users?status=active&role=admin&createdAfter=2025-01-01
```

## Tri

Utilisez le paramètre `sort` pour trier les résultats :

```http
GET /v1/users?sort=createdAt:desc,name:asc
```

Format : `champ:ordre` où ordre est `asc` ou `desc`.

## Recherche

Utilisez le paramètre `search` pour rechercher :

```http
GET /v1/users?search=jean
```

La recherche s'effectue sur les champs principaux de la ressource.

## Inclusion de relations

Utilisez le paramètre `include` pour inclure des ressources liées :

```http
GET /v1/users/123?include=posts,comments
```

## Sélection de champs

Limitez les champs retournés avec le paramètre `fields` :

```http
GET /v1/users?fields=id,name,email
```

## Rate Limiting

L'API limite le nombre de requêtes :

- **Limite** : 1000 requêtes par heure
- **Headers** :
  - `X-RateLimit-Limit` : Limite totale
  - `X-RateLimit-Remaining` : Requêtes restantes
  - `X-RateLimit-Reset` : Timestamp de réinitialisation

Exemple de headers :

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 873
X-RateLimit-Reset: 1642252800
```

## CORS

L'API supporte CORS pour les requêtes cross-origin.

Headers CORS retournés :

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, PATCH
Access-Control-Allow-Headers: Content-Type, Authorization
```

## Webhooks

L'API peut envoyer des notifications webhook pour certains événements.

Consultez la section Webhooks pour plus de détails.

## SDK et bibliothèques

Des SDK sont disponibles pour faciliter l'intégration :

- **JavaScript/TypeScript** : `npm install @example/api-client`
- **Python** : `pip install example-api`
- **Ruby** : `gem install example-api`
- **Go** : `go get github.com/example/api-go`

## Prochaines étapes

- Configurez l'[authentification](authentication.md)
- Explorez les [endpoints](endpoints.md)
- Consultez les [codes d'erreur](error-codes.md)

## Support

Pour toute question :
- Email : api-support@example.com
- Documentation : https://docs.example.com
- Status : https://status.example.com
