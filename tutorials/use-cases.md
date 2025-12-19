# Cas d'usage

## Introduction

Cette page présente des cas d'usage réels et pratiques pour vous inspirer dans vos projets.

## Cas d'usage 1 : Application de gestion de tâches

### Contexte

Créer une application de todo list avec synchronisation en temps réel.

### Fonctionnalités

- Création, lecture, mise à jour et suppression de tâches
- Synchronisation multi-utilisateurs
- Notifications en temps réel
- Filtrage et recherche avancés

### Implémentation

```javascript
class TaskManager {
  constructor(client, eventBus) {
    this.client = client;
    this.eventBus = eventBus;
    this.tasks = [];

    this.setupListeners();
  }

  setupListeners() {
    this.eventBus.on('task:created', (task) => {
      this.tasks.push(task);
      this.notify('Nouvelle tâche créée');
    });

    this.eventBus.on('task:updated', (task) => {
      const index = this.tasks.findIndex(t => t.id === task.id);
      if (index !== -1) {
        this.tasks[index] = task;
      }
    });

    this.eventBus.on('task:deleted', (taskId) => {
      this.tasks = this.tasks.filter(t => t.id !== taskId);
    });
  }

  async createTask(data) {
    const task = await this.client.create({
      ...data,
      status: 'pending',
      createdAt: Date.now()
    });

    this.eventBus.emit('task:created', task);
    return task;
  }

  async updateTask(id, updates) {
    const task = await this.client.update(id, updates);
    this.eventBus.emit('task:updated', task);
    return task;
  }

  async deleteTask(id) {
    await this.client.delete(id);
    this.eventBus.emit('task:deleted', id);
  }

  getTasks(filter = {}) {
    return this.tasks.filter(task => {
      if (filter.status && task.status !== filter.status) {
        return false;
      }
      if (filter.priority && task.priority !== filter.priority) {
        return false;
      }
      return true;
    });
  }

  searchTasks(query) {
    const lowerQuery = query.toLowerCase();
    return this.tasks.filter(task =>
      task.title.toLowerCase().includes(lowerQuery) ||
      task.description.toLowerCase().includes(lowerQuery)
    );
  }

  notify(message) {
    console.log('🔔', message);
  }
}
```

### Utilisation

```javascript
const taskManager = new TaskManager(client, eventBus);

// Créer une tâche
const task = await taskManager.createTask({
  title: 'Finir la documentation',
  description: 'Compléter tous les cas d\'usage',
  priority: 'high'
});

// Filtrer les tâches
const highPriorityTasks = taskManager.getTasks({ priority: 'high' });

// Rechercher
const results = taskManager.searchTasks('documentation');
```

## Cas d'usage 2 : API Gateway

### Contexte

Créer un gateway pour router les requêtes vers différents microservices.

### Architecture

```javascript
class APIGateway {
  constructor() {
    this.services = new Map();
    this.middlewares = [];
    this.cache = new Map();
  }

  registerService(name, config) {
    this.services.set(name, {
      baseUrl: config.baseUrl,
      timeout: config.timeout || 5000,
      retries: config.retries || 3
    });
  }

  use(middleware) {
    this.middlewares.push(middleware);
  }

  async route(serviceName, path, options = {}) {
    const service = this.services.get(serviceName);
    if (!service) {
      throw new Error(`Service ${serviceName} not found`);
    }

    // Vérifier le cache
    const cacheKey = `${serviceName}:${path}`;
    if (options.cache && this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    // Appliquer les middlewares
    let request = { serviceName, path, options };
    for (const middleware of this.middlewares) {
      request = await middleware(request);
    }

    // Faire la requête
    const response = await this.makeRequest(service, path, options);

    // Mettre en cache si nécessaire
    if (options.cache) {
      this.cache.set(cacheKey, response);
      setTimeout(() => this.cache.delete(cacheKey), options.cacheTTL || 60000);
    }

    return response;
  }

  async makeRequest(service, path, options) {
    const url = `${service.baseUrl}${path}`;

    for (let attempt = 1; attempt <= service.retries; attempt++) {
      try {
        const response = await fetch(url, {
          ...options,
          timeout: service.timeout
        });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        return await response.json();
      } catch (error) {
        if (attempt === service.retries) {
          throw error;
        }

        // Backoff exponentiel
        await this.sleep(Math.pow(2, attempt) * 100);
      }
    }
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Configuration
const gateway = new APIGateway();

gateway.registerService('users', {
  baseUrl: 'http://users-service:3001',
  timeout: 3000
});

gateway.registerService('orders', {
  baseUrl: 'http://orders-service:3002',
  timeout: 5000
});

// Middleware d'authentification
gateway.use(async (request) => {
  request.options.headers = {
    ...request.options.headers,
    'Authorization': `Bearer ${getToken()}`
  };
  return request;
});

// Middleware de logging
gateway.use(async (request) => {
  console.log(`→ ${request.serviceName}${request.path}`);
  return request;
});

// Utilisation
const user = await gateway.route('users', '/api/users/123', {
  cache: true,
  cacheTTL: 30000
});
```

## Cas d'usage 3 : Pipeline de traitement de données

### Contexte

Traiter un grand volume de données avec transformation et validation.

### Implémentation

```javascript
class DataPipeline {
  constructor() {
    this.steps = [];
  }

  addStep(name, fn, options = {}) {
    this.steps.push({
      name,
      fn,
      parallel: options.parallel || false,
      retryOnError: options.retryOnError || false
    });
    return this;
  }

  async process(data) {
    let result = data;
    const stats = {
      processed: 0,
      errors: 0,
      duration: 0
    };

    const startTime = Date.now();

    for (const step of this.steps) {
      console.log(`Exécution de l'étape: ${step.name}`);

      try {
        if (step.parallel && Array.isArray(result)) {
          result = await Promise.all(
            result.map(item => this.executeStep(step, item))
          );
        } else {
          result = await this.executeStep(step, result);
        }

        stats.processed++;
      } catch (error) {
        console.error(`Erreur dans ${step.name}:`, error);
        stats.errors++;

        if (!step.retryOnError) {
          throw error;
        }
      }
    }

    stats.duration = Date.now() - startTime;

    return { result, stats };
  }

  async executeStep(step, data) {
    return await step.fn(data);
  }
}

// Exemple d'utilisation
const pipeline = new DataPipeline();

pipeline
  .addStep('validate', (data) => {
    return data.filter(item => item.id && item.value);
  })
  .addStep('transform', (data) => {
    return data.map(item => ({
      ...item,
      value: item.value * 2,
      processed: true
    }));
  }, { parallel: true })
  .addStep('enrich', async (data) => {
    // Enrichir avec des données externes
    const enriched = [];
    for (const item of data) {
      const additional = await fetchAdditionalData(item.id);
      enriched.push({ ...item, ...additional });
    }
    return enriched;
  })
  .addStep('save', async (data) => {
    await database.bulkInsert(data);
    return data;
  }, { retryOnError: true });

// Traiter les données
const rawData = [
  { id: 1, value: 10 },
  { id: 2, value: 20 },
  { id: 3, value: 30 }
];

const { result, stats } = await pipeline.process(rawData);
console.log('Traitement terminé:', stats);
```

## Cas d'usage 4 : Rate Limiter

### Contexte

Implémenter un système de limitation de taux pour protéger les APIs.

### Implémentation

```javascript
class RateLimiter {
  constructor(options = {}) {
    this.maxRequests = options.maxRequests || 100;
    this.windowMs = options.windowMs || 60000;
    this.requests = new Map();
  }

  async checkLimit(identifier) {
    const now = Date.now();
    const windowStart = now - this.windowMs;

    // Nettoyer les anciennes entrées
    if (!this.requests.has(identifier)) {
      this.requests.set(identifier, []);
    }

    const userRequests = this.requests.get(identifier);
    const validRequests = userRequests.filter(time => time > windowStart);

    if (validRequests.length >= this.maxRequests) {
      const oldestRequest = Math.min(...validRequests);
      const resetTime = oldestRequest + this.windowMs;
      const retryAfter = Math.ceil((resetTime - now) / 1000);

      return {
        allowed: false,
        retryAfter,
        limit: this.maxRequests,
        remaining: 0
      };
    }

    validRequests.push(now);
    this.requests.set(identifier, validRequests);

    return {
      allowed: true,
      limit: this.maxRequests,
      remaining: this.maxRequests - validRequests.length,
      reset: windowStart + this.windowMs
    };
  }

  middleware() {
    return async (req, res, next) => {
      const identifier = req.ip || req.user?.id || 'anonymous';
      const result = await this.checkLimit(identifier);

      res.setHeader('X-RateLimit-Limit', result.limit);
      res.setHeader('X-RateLimit-Remaining', result.remaining);

      if (!result.allowed) {
        res.setHeader('X-RateLimit-Reset', result.reset);
        res.setHeader('Retry-After', result.retryAfter);
        return res.status(429).json({
          error: 'Too many requests',
          retryAfter: result.retryAfter
        });
      }

      next();
    };
  }
}

// Utilisation
const limiter = new RateLimiter({
  maxRequests: 100,
  windowMs: 60000 // 1 minute
});

app.use('/api', limiter.middleware());
```

## Résumé

Ces cas d'usage démontrent :

- 📋 Gestion de tâches avec événements
- 🌐 API Gateway avec cache et retry
- ⚙️ Pipeline de traitement de données
- 🛡️ Rate limiting pour protection d'API

## Prochaines étapes

- Appliquez ces patterns à vos projets
- Consultez les [bonnes pratiques](../guides/best-practices.md)
- Explorez la [référence API](../api-reference/introduction.md)
