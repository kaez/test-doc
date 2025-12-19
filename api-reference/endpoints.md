# Endpoints

## Ressources disponibles

L'API expose les ressources suivantes :

- [Users](#users) - Gestion des utilisateurs
- [Posts](#posts) - Gestion des publications
- [Comments](#comments) - Gestion des commentaires
- [Files](#files) - Gestion des fichiers
- [Analytics](#analytics) - Données analytiques

---

## Users

### Lister les utilisateurs

```http
GET /v1/users
```

**Paramètres de query** :

| Paramètre | Type | Description |
|-----------|------|-------------|
| `page` | integer | Numéro de page (défaut: 1) |
| `limit` | integer | Éléments par page (défaut: 20) |
| `status` | string | Filtrer par statut (active, inactive) |
| `role` | string | Filtrer par rôle (admin, user, guest) |
| `search` | string | Rechercher par nom ou email |

**Exemple de requête** :

```bash
curl https://api.example.com/v1/users?status=active&limit=50 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Réponse** :

```json
{
  "success": true,
  "data": [
    {
      "id": "user_123",
      "name": "Jean Dupont",
      "email": "jean@example.com",
      "role": "user",
      "status": "active",
      "createdAt": "2025-01-10T10:30:00Z",
      "updatedAt": "2025-01-15T14:20:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 125,
    "totalPages": 3
  }
}
```

### Obtenir un utilisateur

```http
GET /v1/users/:id
```

**Paramètres d'URL** :

| Paramètre | Type | Description |
|-----------|------|-------------|
| `id` | string | ID de l'utilisateur |

**Exemple** :

```bash
curl https://api.example.com/v1/users/user_123 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Réponse** :

```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "name": "Jean Dupont",
    "email": "jean@example.com",
    "role": "user",
    "status": "active",
    "profile": {
      "bio": "Développeur passionné",
      "location": "Paris, France",
      "website": "https://example.com"
    },
    "createdAt": "2025-01-10T10:30:00Z",
    "updatedAt": "2025-01-15T14:20:00Z"
  }
}
```

### Créer un utilisateur

```http
POST /v1/users
```

**Body** :

```json
{
  "name": "Marie Martin",
  "email": "marie@example.com",
  "password": "SecurePassword123!",
  "role": "user"
}
```

**Exemple** :

```bash
curl -X POST https://api.example.com/v1/users \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Marie Martin",
    "email": "marie@example.com",
    "password": "SecurePassword123!",
    "role": "user"
  }'
```

**Réponse** (201 Created) :

```json
{
  "success": true,
  "data": {
    "id": "user_456",
    "name": "Marie Martin",
    "email": "marie@example.com",
    "role": "user",
    "status": "active",
    "createdAt": "2025-01-15T15:00:00Z"
  }
}
```

### Mettre à jour un utilisateur

```http
PATCH /v1/users/:id
```

**Body** :

```json
{
  "name": "Marie Dupont",
  "profile": {
    "bio": "Designer UX/UI",
    "location": "Lyon, France"
  }
}
```

**Réponse** :

```json
{
  "success": true,
  "data": {
    "id": "user_456",
    "name": "Marie Dupont",
    "email": "marie@example.com",
    "profile": {
      "bio": "Designer UX/UI",
      "location": "Lyon, France"
    },
    "updatedAt": "2025-01-15T15:30:00Z"
  }
}
```

### Supprimer un utilisateur

```http
DELETE /v1/users/:id
```

**Réponse** (204 No Content) :

```
(pas de contenu)
```

---

## Posts

### Lister les publications

```http
GET /v1/posts
```

**Paramètres** :

| Paramètre | Type | Description |
|-----------|------|-------------|
| `authorId` | string | Filtrer par auteur |
| `status` | string | published, draft, archived |
| `tags` | string | Filtrer par tags (séparés par virgule) |
| `sort` | string | Tri (createdAt:desc, title:asc) |

**Exemple** :

```bash
curl "https://api.example.com/v1/posts?status=published&sort=createdAt:desc" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Réponse** :

```json
{
  "success": true,
  "data": [
    {
      "id": "post_789",
      "title": "Introduction à l'API",
      "slug": "introduction-api",
      "excerpt": "Découvrez comment utiliser notre API...",
      "content": "# Introduction\n\nCet article présente...",
      "status": "published",
      "author": {
        "id": "user_123",
        "name": "Jean Dupont"
      },
      "tags": ["api", "tutorial"],
      "publishedAt": "2025-01-15T12:00:00Z",
      "createdAt": "2025-01-15T10:00:00Z",
      "updatedAt": "2025-01-15T11:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

### Créer une publication

```http
POST /v1/posts
```

**Body** :

```json
{
  "title": "Mon premier article",
  "content": "# Titre\n\nContenu de l'article...",
  "status": "draft",
  "tags": ["test", "demo"],
  "metadata": {
    "featured": false,
    "allowComments": true
  }
}
```

---

## Comments

### Lister les commentaires

```http
GET /v1/comments
```

**Paramètres** :

| Paramètre | Type | Description |
|-----------|------|-------------|
| `postId` | string | Filtrer par publication |
| `authorId` | string | Filtrer par auteur |
| `status` | string | approved, pending, spam |

**Exemple** :

```bash
curl "https://api.example.com/v1/comments?postId=post_789" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Créer un commentaire

```http
POST /v1/comments
```

**Body** :

```json
{
  "postId": "post_789",
  "content": "Excellent article !",
  "parentId": null
}
```

---

## Files

### Uploader un fichier

```http
POST /v1/files
```

**Headers** :

```
Content-Type: multipart/form-data
```

**Exemple** :

```bash
curl -X POST https://api.example.com/v1/files \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@/path/to/file.pdf" \
  -F "folder=documents"
```

**Réponse** :

```json
{
  "success": true,
  "data": {
    "id": "file_abc",
    "name": "document.pdf",
    "size": 1024000,
    "mimeType": "application/pdf",
    "url": "https://cdn.example.com/files/abc123/document.pdf",
    "folder": "documents",
    "uploadedAt": "2025-01-15T16:00:00Z"
  }
}
```

### Télécharger un fichier

```http
GET /v1/files/:id/download
```

### Supprimer un fichier

```http
DELETE /v1/files/:id
```

---

## Analytics

### Obtenir des statistiques

```http
GET /v1/analytics/stats
```

**Paramètres** :

| Paramètre | Type | Description |
|-----------|------|-------------|
| `startDate` | string | Date de début (ISO 8601) |
| `endDate` | string | Date de fin (ISO 8601) |
| `metric` | string | Métrique (views, users, posts) |
| `interval` | string | Intervalle (day, week, month) |

**Exemple** :

```bash
curl "https://api.example.com/v1/analytics/stats?metric=views&interval=day&startDate=2025-01-01&endDate=2025-01-15" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**Réponse** :

```json
{
  "success": true,
  "data": {
    "metric": "views",
    "interval": "day",
    "dataPoints": [
      {
        "date": "2025-01-01",
        "value": 1245
      },
      {
        "date": "2025-01-02",
        "value": 1567
      }
    ],
    "summary": {
      "total": 18234,
      "average": 1215,
      "min": 834,
      "max": 2103
    }
  }
}
```

---

## Webhooks

### Lister les webhooks

```http
GET /v1/webhooks
```

### Créer un webhook

```http
POST /v1/webhooks
```

**Body** :

```json
{
  "url": "https://yourapp.com/webhooks",
  "events": ["user.created", "post.published"],
  "secret": "your_webhook_secret"
}
```

### Tester un webhook

```http
POST /v1/webhooks/:id/test
```

---

## Limites et quotas

| Endpoint | Limite |
|----------|--------|
| GET /users | 1000/heure |
| POST /users | 100/heure |
| POST /files | 50/heure |
| GET /analytics | 500/heure |

## Prochaines étapes

- Consultez les [codes d'erreur](error-codes.md)
- Comprenez l'[authentification](authentication.md)
- Explorez les exemples d'intégration
