# Bonnes pratiques

## Introduction

Ce guide présente les bonnes pratiques recommandées pour utiliser l'application de manière optimale.

## Architecture

### Organisation du code

#### Structure de projet recommandée

```
mon-projet/
├── src/
│   ├── api/           # Couche API
│   ├── components/    # Composants réutilisables
│   ├── services/      # Logique métier
│   ├── utils/         # Utilitaires
│   ├── types/         # Définitions TypeScript
│   └── config/        # Configuration
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/
└── scripts/
```

#### Séparation des responsabilités

✅ **Bien** :

```javascript
// services/userService.js
class UserService {
  async getUser(id) {
    const user = await this.repository.findById(id);
    return this.transform(user);
  }

  transform(user) {
    return {
      id: user.id,
      name: user.name,
      email: user.email
    };
  }
}

// api/userController.js
class UserController {
  async getUser(req, res) {
    try {
      const user = await this.userService.getUser(req.params.id);
      res.json({ success: true, data: user });
    } catch (error) {
      this.handleError(error, res);
    }
  }
}
```

❌ **À éviter** :

```javascript
// Tout mélangé dans le contrôleur
async function getUser(req, res) {
  const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
  const transformed = {
    id: user.id,
    name: user.name,
    email: user.email
  };
  res.json({ data: transformed });
}
```

### Gestion de la configuration

✅ **Bien** :

```javascript
// config/index.js
const config = {
  api: {
    baseUrl: process.env.API_BASE_URL || 'http://localhost:3000',
    timeout: parseInt(process.env.API_TIMEOUT) || 5000,
    retries: parseInt(process.env.API_RETRIES) || 3
  },
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT) || 5432,
    name: process.env.DB_NAME || 'myapp'
  },
  cache: {
    enabled: process.env.CACHE_ENABLED === 'true',
    ttl: parseInt(process.env.CACHE_TTL) || 3600
  }
};

// Validation
if (!config.api.baseUrl) {
  throw new Error('API_BASE_URL is required');
}

module.exports = config;
```

❌ **À éviter** :

```javascript
// Valeurs en dur
const API_URL = 'http://localhost:3000';
const TIMEOUT = 5000;
```

## Gestion des erreurs

### Try-catch approprié

✅ **Bien** :

```javascript
async function processData(data) {
  try {
    const validated = await validateData(data);
    const processed = await processValidData(validated);
    return { success: true, result: processed };
  } catch (error) {
    if (error instanceof ValidationError) {
      return { success: false, error: 'Validation failed', details: error.details };
    }

    if (error instanceof NetworkError) {
      // Retry logic
      return await retryRequest(() => processData(data));
    }

    // Log unexpected errors
    logger.error('Unexpected error in processData:', error);
    throw error;
  }
}
```

### Erreurs personnalisées

```javascript
class AppError extends Error {
  constructor(message, code, statusCode = 500) {
    super(message);
    this.code = code;
    this.statusCode = statusCode;
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, details = {}) {
    super(message, 'VALIDATION_ERROR', 422);
    this.details = details;
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND', 404);
    this.resource = resource;
    this.id = id;
  }
}

// Utilisation
throw new ValidationError('Invalid email', { field: 'email', value: 'invalid' });
throw new NotFoundError('User', 'user_123');
```

## Performance

### Mise en cache

```javascript
class CachedService {
  constructor(cache, ttl = 3600) {
    this.cache = cache;
    this.ttl = ttl;
  }

  async getData(key, fetchFn) {
    // Vérifier le cache
    const cached = await this.cache.get(key);
    if (cached) {
      return JSON.parse(cached);
    }

    // Récupérer les données
    const data = await fetchFn();

    // Mettre en cache
    await this.cache.set(key, JSON.stringify(data), this.ttl);

    return data;
  }

  async invalidate(key) {
    await this.cache.delete(key);
  }
}

// Utilisation
const service = new CachedService(redisClient, 3600);

const user = await service.getData(
  `user:${userId}`,
  () => database.findUser(userId)
);
```

### Lazy loading

```javascript
class LazyLoader {
  constructor() {
    this.loaded = new Map();
  }

  async load(key, loader) {
    if (this.loaded.has(key)) {
      return this.loaded.get(key);
    }

    const result = await loader();
    this.loaded.set(key, result);
    return result;
  }
}

// Utilisation
const loader = new LazyLoader();

const config = await loader.load('config', () => loadConfig());
const schema = await loader.load('schema', () => loadSchema());
```

### Pagination

✅ **Bien** :

```javascript
async function getUsers(page = 1, limit = 20) {
  // Validation
  const validLimit = Math.min(Math.max(limit, 1), 100);
  const validPage = Math.max(page, 1);

  const offset = (validPage - 1) * validLimit;

  const [users, total] = await Promise.all([
    database.query('SELECT * FROM users LIMIT ? OFFSET ?', [validLimit, offset]),
    database.query('SELECT COUNT(*) as count FROM users')
  ]);

  return {
    data: users,
    pagination: {
      page: validPage,
      limit: validLimit,
      total: total[0].count,
      totalPages: Math.ceil(total[0].count / validLimit),
      hasNext: validPage * validLimit < total[0].count,
      hasPrevious: validPage > 1
    }
  };
}
```

## Sécurité

### Validation des entrées

✅ **Bien** :

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  name: Joi.string().min(2).max(100).required(),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(0).max(150),
  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .required()
});

async function createUser(data) {
  const { error, value } = userSchema.validate(data);

  if (error) {
    throw new ValidationError(
      'Validation failed',
      error.details.map(d => ({
        field: d.path.join('.'),
        message: d.message
      }))
    );
  }

  return await database.createUser(value);
}
```

### Protection contre les injections

✅ **Bien** :

```javascript
// Requêtes paramétrées
async function getUserByEmail(email) {
  return await db.query(
    'SELECT * FROM users WHERE email = ?',
    [email]
  );
}

// ORM
async function getUserByEmail(email) {
  return await User.findOne({ where: { email } });
}
```

❌ **À éviter** :

```javascript
// Concaténation de chaînes - VULNÉRABLE
async function getUserByEmail(email) {
  return await db.query(
    `SELECT * FROM users WHERE email = '${email}'`
  );
}
```

### Sanitization

```javascript
const sanitizeHtml = require('sanitize-html');

function sanitizeUserInput(input) {
  return sanitizeHtml(input, {
    allowedTags: ['b', 'i', 'em', 'strong', 'a', 'p'],
    allowedAttributes: {
      'a': ['href']
    }
  });
}

// Utilisation
const post = {
  title: sanitizeUserInput(req.body.title),
  content: sanitizeUserInput(req.body.content)
};
```

### Rate limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limite par IP
  message: 'Trop de requêtes, réessayez plus tard',
  standardHeaders: true,
  legacyHeaders: false,
  handler: (req, res) => {
    res.status(429).json({
      success: false,
      error: {
        code: 'RATE_LIMIT_EXCEEDED',
        message: 'Trop de requêtes'
      }
    });
  }
});

app.use('/api', limiter);
```

## Testing

### Tests unitaires

```javascript
describe('UserService', () => {
  let userService;
  let mockRepository;

  beforeEach(() => {
    mockRepository = {
      findById: jest.fn(),
      create: jest.fn(),
      update: jest.fn()
    };

    userService = new UserService(mockRepository);
  });

  describe('getUser', () => {
    it('should return transformed user data', async () => {
      const mockUser = {
        id: 'user_123',
        name: 'Test User',
        email: 'test@example.com',
        password: 'hashed_password'
      };

      mockRepository.findById.mockResolvedValue(mockUser);

      const result = await userService.getUser('user_123');

      expect(result).toEqual({
        id: 'user_123',
        name: 'Test User',
        email: 'test@example.com'
      });

      expect(result.password).toBeUndefined();
      expect(mockRepository.findById).toHaveBeenCalledWith('user_123');
    });

    it('should throw NotFoundError when user does not exist', async () => {
      mockRepository.findById.mockResolvedValue(null);

      await expect(userService.getUser('invalid_id'))
        .rejects
        .toThrow(NotFoundError);
    });
  });
});
```

### Tests d'intégration

```javascript
describe('User API', () => {
  let app;
  let database;

  beforeAll(async () => {
    database = await setupTestDatabase();
    app = createApp(database);
  });

  afterAll(async () => {
    await database.close();
  });

  beforeEach(async () => {
    await database.clear();
  });

  it('should create a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        name: 'Test User',
        email: 'test@example.com',
        password: 'SecurePass123!'
      })
      .expect(201);

    expect(response.body.success).toBe(true);
    expect(response.body.data).toMatchObject({
      name: 'Test User',
      email: 'test@example.com'
    });
    expect(response.body.data.password).toBeUndefined();
  });
});
```

## Logging

### Logger structuré

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'my-app',
    environment: process.env.NODE_ENV
  },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// Utilisation
logger.info('User created', { userId: 'user_123', email: 'test@example.com' });
logger.error('Database connection failed', { error: err.message, stack: err.stack });
```

## Documentation

### JSDoc

```javascript
/**
 * Récupère un utilisateur par son ID
 * @param {string} id - L'ID de l'utilisateur
 * @param {Object} options - Options de récupération
 * @param {boolean} options.includeDeleted - Inclure les utilisateurs supprimés
 * @returns {Promise<User>} L'utilisateur trouvé
 * @throws {NotFoundError} Si l'utilisateur n'existe pas
 * @example
 * const user = await getUser('user_123', { includeDeleted: false });
 */
async function getUser(id, options = {}) {
  // ...
}
```

## Résumé

Les bonnes pratiques essentielles :

- ✅ Séparer les responsabilités
- ✅ Gérer les erreurs proprement
- ✅ Valider toutes les entrées
- ✅ Utiliser le cache intelligemment
- ✅ Écrire des tests
- ✅ Logger de manière structurée
- ✅ Documenter le code

## Prochaines étapes

- Consultez les [optimisations de performance](performance.md)
- Découvrez les [pratiques de sécurité](security.md)
- Explorez les [cas d'usage](../tutorials/use-cases.md)
