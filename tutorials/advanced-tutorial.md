# Tutoriel avancé

## Introduction

Ce tutoriel couvre des concepts avancés et des techniques d'optimisation. Vous apprendrez :

- Gestion avancée de l'état
- Optimisation des performances
- Patterns architecturaux
- Intégrations complexes

## Prérequis

Avant de commencer, assurez-vous d'avoir complété le [tutoriel de base](basic-tutorial.md).

## Architecture avancée

### Plugin system

Créez des plugins réutilisables :

```javascript
class MonPlugin {
  constructor(options = {}) {
    this.name = 'MonPlugin';
    this.version = '1.0.0';
    this.options = options;
  }

  async initialize(app) {
    console.log('Plugin initialisé');

    // Ajouter des middlewares
    app.use(this.middleware.bind(this));

    // Enregistrer des routes
    app.addRoutes(this.routes());
  }

  middleware(req, res, next) {
    req.plugin = this;
    next();
  }

  routes() {
    return [
      {
        path: '/plugin/status',
        handler: (req, res) => {
          res.json({
            plugin: this.name,
            version: this.version
          });
        }
      }
    ];
  }
}

// Utilisation
app.use(new MonPlugin({ debug: true }));
```

### Service Layer Pattern

```javascript
class UserService {
  constructor(repository, cache) {
    this.repository = repository;
    this.cache = cache;
  }

  async getUser(id) {
    // Vérifier le cache
    const cached = await this.cache.get(`user:${id}`);
    if (cached) return JSON.parse(cached);

    // Charger depuis la base
    const user = await this.repository.findById(id);

    // Mettre en cache
    await this.cache.set(`user:${id}`, JSON.stringify(user), 3600);

    return user;
  }

  async updateUser(id, data) {
    const user = await this.repository.update(id, data);

    // Invalider le cache
    await this.cache.delete(`user:${id}`);

    return user;
  }
}
```

## Optimisation des performances

### Batch processing

```javascript
class BatchProcessor {
  constructor(batchSize = 100, delay = 1000) {
    this.batchSize = batchSize;
    this.delay = delay;
    this.queue = [];
    this.processing = false;
  }

  async add(item) {
    this.queue.push(item);

    if (this.queue.length >= this.batchSize) {
      await this.process();
    } else if (!this.processing) {
      this.scheduleProcess();
    }
  }

  scheduleProcess() {
    setTimeout(() => this.process(), this.delay);
  }

  async process() {
    if (this.queue.length === 0) return;

    this.processing = true;
    const batch = this.queue.splice(0, this.batchSize);

    try {
      await this.processBatch(batch);
    } catch (error) {
      console.error('Erreur de traitement:', error);
      // Remettre dans la queue
      this.queue.unshift(...batch);
    } finally {
      this.processing = false;

      if (this.queue.length > 0) {
        this.scheduleProcess();
      }
    }
  }

  async processBatch(items) {
    return Promise.all(items.map(item => this.processItem(item)));
  }

  async processItem(item) {
    // Logique de traitement
    console.log('Traitement de:', item);
  }
}

// Utilisation
const processor = new BatchProcessor(50, 500);
for (let i = 0; i < 1000; i++) {
  processor.add({ id: i, data: `item-${i}` });
}
```

### Connection pooling

```javascript
class ConnectionPool {
  constructor(createConnection, { min = 2, max = 10 }) {
    this.createConnection = createConnection;
    this.min = min;
    this.max = max;
    this.pool = [];
    this.activeConnections = 0;
    this.waitQueue = [];
  }

  async initialize() {
    for (let i = 0; i < this.min; i++) {
      const conn = await this.createConnection();
      this.pool.push(conn);
    }
  }

  async acquire() {
    // Connexion disponible
    if (this.pool.length > 0) {
      return this.pool.pop();
    }

    // Créer une nouvelle connexion si possible
    if (this.activeConnections < this.max) {
      this.activeConnections++;
      return await this.createConnection();
    }

    // Attendre qu'une connexion se libère
    return new Promise(resolve => {
      this.waitQueue.push(resolve);
    });
  }

  release(connection) {
    if (this.waitQueue.length > 0) {
      const resolve = this.waitQueue.shift();
      resolve(connection);
    } else {
      this.pool.push(connection);
    }
  }

  async execute(fn) {
    const conn = await this.acquire();
    try {
      return await fn(conn);
    } finally {
      this.release(conn);
    }
  }
}

// Utilisation
const pool = new ConnectionPool(
  () => createDatabaseConnection(),
  { min: 5, max: 20 }
);

await pool.initialize();

const result = await pool.execute(async (conn) => {
  return conn.query('SELECT * FROM users');
});
```

## Patterns avancés

### Event Sourcing

```javascript
class EventStore {
  constructor() {
    this.events = [];
    this.snapshots = new Map();
  }

  append(aggregateId, event) {
    this.events.push({
      aggregateId,
      event,
      timestamp: Date.now()
    });
  }

  getEvents(aggregateId, fromVersion = 0) {
    return this.events.filter(
      e => e.aggregateId === aggregateId && e.version >= fromVersion
    );
  }

  createSnapshot(aggregateId, state, version) {
    this.snapshots.set(aggregateId, { state, version });
  }

  rebuild(aggregateId) {
    const snapshot = this.snapshots.get(aggregateId);
    let state = snapshot ? snapshot.state : {};
    const fromVersion = snapshot ? snapshot.version : 0;

    const events = this.getEvents(aggregateId, fromVersion);

    for (const { event } of events) {
      state = this.apply(state, event);
    }

    return state;
  }

  apply(state, event) {
    // Appliquer l'événement à l'état
    switch (event.type) {
      case 'USER_CREATED':
        return { ...state, ...event.data };
      case 'USER_UPDATED':
        return { ...state, ...event.data };
      default:
        return state;
    }
  }
}
```

### CQRS Pattern

```javascript
// Command side
class CommandHandler {
  constructor(eventStore) {
    this.eventStore = eventStore;
  }

  async handleCreateUser(command) {
    const event = {
      type: 'USER_CREATED',
      data: command.data,
      timestamp: Date.now()
    };

    this.eventStore.append(command.userId, event);

    // Publier l'événement
    await this.publish(event);
  }

  async publish(event) {
    // Publier vers les projections
    console.log('Événement publié:', event);
  }
}

// Query side
class QueryHandler {
  constructor(readModel) {
    this.readModel = readModel;
  }

  async getUser(userId) {
    return this.readModel.findUser(userId);
  }

  async searchUsers(criteria) {
    return this.readModel.searchUsers(criteria);
  }
}

// Read model (projection)
class UserReadModel {
  constructor() {
    this.users = new Map();
  }

  handleEvent(event) {
    switch (event.type) {
      case 'USER_CREATED':
        this.users.set(event.data.id, event.data);
        break;
      case 'USER_UPDATED':
        const user = this.users.get(event.data.id);
        this.users.set(event.data.id, { ...user, ...event.data });
        break;
    }
  }

  findUser(userId) {
    return this.users.get(userId);
  }

  searchUsers(criteria) {
    return Array.from(this.users.values()).filter(user =>
      Object.entries(criteria).every(([key, value]) =>
        user[key] === value
      )
    );
  }
}
```

## Monitoring et observabilité

### Instrumentation

```javascript
class Metrics {
  constructor() {
    this.counters = new Map();
    this.timers = new Map();
  }

  increment(name, value = 1) {
    const current = this.counters.get(name) || 0;
    this.counters.set(name, current + value);
  }

  timing(name, duration) {
    if (!this.timers.has(name)) {
      this.timers.set(name, []);
    }
    this.timers.get(name).push(duration);
  }

  async measure(name, fn) {
    const start = Date.now();
    try {
      return await fn();
    } finally {
      this.timing(name, Date.now() - start);
    }
  }

  getStats() {
    const stats = {};

    for (const [name, values] of this.timers) {
      stats[name] = {
        count: values.length,
        avg: values.reduce((a, b) => a + b, 0) / values.length,
        min: Math.min(...values),
        max: Math.max(...values)
      };
    }

    return stats;
  }
}

// Utilisation
const metrics = new Metrics();

async function monOperation() {
  return metrics.measure('operation.duration', async () => {
    metrics.increment('operation.count');
    // Votre logique ici
  });
}
```

## Résumé

Dans ce tutoriel avancé, vous avez appris :

- ✅ Architecture avec plugins
- ✅ Optimisation des performances
- ✅ Patterns avancés (Event Sourcing, CQRS)
- ✅ Monitoring et observabilité

## Prochaines étapes

- Consultez les [cas d'usage réels](use-cases.md)
- Explorez les [bonnes pratiques](../guides/best-practices.md)
- Découvrez les optimisations de [performance](../guides/performance.md)
