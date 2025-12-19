# Optimisation des performances

## Introduction

Ce guide couvre les techniques d'optimisation des performances pour améliorer la vitesse et l'efficacité de vos applications.

## Mesure des performances

### Benchmarking

```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
  }

  start(name) {
    this.metrics.set(name, {
      start: performance.now(),
      end: null,
      duration: null
    });
  }

  end(name) {
    const metric = this.metrics.get(name);
    if (!metric) {
      throw new Error(`Metric ${name} not found`);
    }

    metric.end = performance.now();
    metric.duration = metric.end - metric.start;

    return metric.duration;
  }

  async measure(name, fn) {
    this.start(name);
    try {
      return await fn();
    } finally {
      const duration = this.end(name);
      console.log(`${name}: ${duration.toFixed(2)}ms`);
    }
  }

  getMetrics() {
    const results = {};
    for (const [name, metric] of this.metrics) {
      results[name] = {
        duration: metric.duration,
        timestamp: metric.end
      };
    }
    return results;
  }
}

// Utilisation
const monitor = new PerformanceMonitor();

await monitor.measure('database-query', async () => {
  return await database.query('SELECT * FROM users');
});

await monitor.measure('api-call', async () => {
  return await fetch('https://api.example.com/data');
});

console.log(monitor.getMetrics());
```

### Profiling

```javascript
const v8Profiler = require('v8-profiler-next');

class Profiler {
  static startCPUProfile(name = 'profile') {
    v8Profiler.startProfiling(name, true);
  }

  static stopCPUProfile(name = 'profile') {
    const profile = v8Profiler.stopProfiling(name);
    return profile;
  }

  static async profileFunction(fn, name = 'function') {
    this.startCPUProfile(name);
    try {
      return await fn();
    } finally {
      const profile = this.stopCPUProfile(name);
      profile.export((error, result) => {
        if (!error) {
          require('fs').writeFileSync(
            `${name}-${Date.now()}.cpuprofile`,
            result
          );
        }
        profile.delete();
      });
    }
  }
}

// Utilisation
await Profiler.profileFunction(async () => {
  // Code à profiler
  for (let i = 0; i < 1000000; i++) {
    Math.sqrt(i);
  }
}, 'heavy-computation');
```

## Optimisation de la base de données

### Indexation

```sql
-- Créer des index pour les colonnes fréquemment interrogées
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_author_status ON posts(author_id, status);
CREATE INDEX idx_created_at ON posts(created_at DESC);

-- Index composite pour les requêtes complexes
CREATE INDEX idx_search ON posts(status, published_at, author_id);
```

### Requêtes optimisées

✅ **Bien** :

```javascript
// Sélectionner uniquement les colonnes nécessaires
async function getUsers() {
  return await db.query(
    'SELECT id, name, email FROM users WHERE status = ?',
    ['active']
  );
}

// Utiliser les jointures au lieu de requêtes multiples
async function getPostsWithAuthors() {
  return await db.query(`
    SELECT
      p.id, p.title, p.content,
      u.id as author_id, u.name as author_name
    FROM posts p
    INNER JOIN users u ON p.author_id = u.id
    WHERE p.status = 'published'
    LIMIT 20
  `);
}
```

❌ **À éviter** :

```javascript
// Sélectionner toutes les colonnes
async function getUsers() {
  return await db.query('SELECT * FROM users');
}

// N+1 queries
async function getPostsWithAuthors() {
  const posts = await db.query('SELECT * FROM posts');

  for (const post of posts) {
    post.author = await db.query(
      'SELECT * FROM users WHERE id = ?',
      [post.author_id]
    );
  }

  return posts;
}
```

### Connection pooling

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'myapp',
  user: 'dbuser',
  password: 'password',
  max: 20,              // Nombre max de clients
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Utilisation
async function query(text, params) {
  const start = Date.now();
  const client = await pool.connect();

  try {
    const result = await client.query(text, params);
    const duration = Date.now() - start;

    console.log('Query executed', { text, duration, rows: result.rowCount });

    return result;
  } finally {
    client.release();
  }
}
```

## Mise en cache

### Cache multi-niveaux

```javascript
class MultiLevelCache {
  constructor(l1Cache, l2Cache, l3Cache) {
    this.l1 = l1Cache;  // Memory cache (rapide, petite capacité)
    this.l2 = l2Cache;  // Redis (moyen, capacité moyenne)
    this.l3 = l3Cache;  // Database (lent, grande capacité)
  }

  async get(key) {
    // Vérifier L1 (mémoire)
    let value = await this.l1.get(key);
    if (value) {
      console.log('L1 cache hit:', key);
      return value;
    }

    // Vérifier L2 (Redis)
    value = await this.l2.get(key);
    if (value) {
      console.log('L2 cache hit:', key);
      // Remettre en L1
      await this.l1.set(key, value);
      return value;
    }

    // Vérifier L3 (Database)
    value = await this.l3.get(key);
    if (value) {
      console.log('L3 cache hit:', key);
      // Remettre en L2 et L1
      await Promise.all([
        this.l2.set(key, value),
        this.l1.set(key, value)
      ]);
      return value;
    }

    console.log('Cache miss:', key);
    return null;
  }

  async set(key, value, ttl) {
    await Promise.all([
      this.l1.set(key, value, ttl),
      this.l2.set(key, value, ttl),
      this.l3.set(key, value, ttl)
    ]);
  }

  async invalidate(key) {
    await Promise.all([
      this.l1.delete(key),
      this.l2.delete(key),
      this.l3.delete(key)
    ]);
  }
}
```

### Cache-aside pattern

```javascript
class CacheAsideRepository {
  constructor(database, cache, ttl = 3600) {
    this.database = database;
    this.cache = cache;
    this.ttl = ttl;
  }

  async findById(id) {
    const cacheKey = `user:${id}`;

    // Vérifier le cache
    const cached = await this.cache.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }

    // Charger depuis la DB
    const user = await this.database.findById(id);

    if (user) {
      // Mettre en cache
      await this.cache.set(
        cacheKey,
        JSON.stringify(user),
        this.ttl
      );
    }

    return user;
  }

  async update(id, data) {
    // Mettre à jour la DB
    const user = await this.database.update(id, data);

    // Invalider le cache
    await this.cache.delete(`user:${id}`);

    return user;
  }

  async delete(id) {
    await this.database.delete(id);
    await this.cache.delete(`user:${id}`);
  }
}
```

## Optimisation réseau

### Compression

```javascript
const compression = require('compression');
const express = require('express');

const app = express();

// Activer la compression gzip
app.use(compression({
  level: 6,
  threshold: 1024,  // Compresser uniquement si > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

### HTTP/2 Server Push

```javascript
const http2 = require('http2');
const fs = require('fs');

const server = http2.createSecureServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem')
});

server.on('stream', (stream, headers) => {
  if (headers[':path'] === '/') {
    // Push des ressources critiques
    stream.pushStream({ ':path': '/style.css' }, (err, pushStream) => {
      if (!err) {
        pushStream.respond({ ':status': 200 });
        pushStream.end(fs.readFileSync('public/style.css'));
      }
    });

    stream.pushStream({ ':path': '/app.js' }, (err, pushStream) => {
      if (!err) {
        pushStream.respond({ ':status': 200 });
        pushStream.end(fs.readFileSync('public/app.js'));
      }
    });

    // Répondre avec le HTML
    stream.respond({
      'content-type': 'text/html',
      ':status': 200
    });
    stream.end(fs.readFileSync('public/index.html'));
  }
});
```

### CDN et cache HTTP

```javascript
app.use('/static', express.static('public', {
  maxAge: '1y',  // Cache pour 1 an
  immutable: true,
  etag: true
}));

app.get('/api/data', (req, res) => {
  // Cache privé de 5 minutes
  res.set('Cache-Control', 'private, max-age=300');
  res.json({ data: getData() });
});

app.get('/api/public-data', (req, res) => {
  // Cache public de 1 heure
  res.set('Cache-Control', 'public, max-age=3600');
  res.set('ETag', generateETag(data));
  res.json({ data: getPublicData() });
});
```

## Optimisation du code

### Memoization

```javascript
function memoize(fn) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);

    return result;
  };
}

// Utilisation
const fibonacci = memoize((n) => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(40)); // Beaucoup plus rapide avec memoization
```

### Debouncing et Throttling

```javascript
// Debounce: exécute après que les appels s'arrêtent
function debounce(fn, delay) {
  let timeoutId;

  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Throttle: exécute au maximum tous les X ms
function throttle(fn, limit) {
  let inThrottle;

  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Utilisation
const debouncedSearch = debounce(searchFunction, 300);
const throttledScroll = throttle(handleScroll, 100);

input.addEventListener('input', debouncedSearch);
window.addEventListener('scroll', throttledScroll);
```

### Lazy loading

```javascript
class LazyImage {
  constructor() {
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '50px' }
    );
  }

  observe(element) {
    this.observer.observe(element);
  }

  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        const src = img.dataset.src;

        if (src) {
          img.src = src;
          img.removeAttribute('data-src');
          this.observer.unobserve(img);
        }
      }
    });
  }
}

// Utilisation
const lazyLoader = new LazyImage();
document.querySelectorAll('img[data-src]').forEach(img => {
  lazyLoader.observe(img);
});
```

## Worker Threads

### Calculs lourds en arrière-plan

```javascript
const { Worker } = require('worker_threads');

class WorkerPool {
  constructor(workerScript, poolSize = 4) {
    this.workerScript = workerScript;
    this.poolSize = poolSize;
    this.workers = [];
    this.queue = [];

    this.initializeWorkers();
  }

  initializeWorkers() {
    for (let i = 0; i < this.poolSize; i++) {
      this.workers.push({
        worker: new Worker(this.workerScript),
        busy: false
      });
    }
  }

  async execute(data) {
    return new Promise((resolve, reject) => {
      const availableWorker = this.workers.find(w => !w.busy);

      if (availableWorker) {
        this.runTask(availableWorker, data, resolve, reject);
      } else {
        this.queue.push({ data, resolve, reject });
      }
    });
  }

  runTask(workerInfo, data, resolve, reject) {
    workerInfo.busy = true;

    const messageHandler = (result) => {
      workerInfo.busy = false;
      workerInfo.worker.off('message', messageHandler);
      workerInfo.worker.off('error', errorHandler);

      // Traiter la prochaine tâche en queue
      if (this.queue.length > 0) {
        const next = this.queue.shift();
        this.runTask(workerInfo, next.data, next.resolve, next.reject);
      }

      resolve(result);
    };

    const errorHandler = (error) => {
      workerInfo.busy = false;
      workerInfo.worker.off('message', messageHandler);
      workerInfo.worker.off('error', errorHandler);
      reject(error);
    };

    workerInfo.worker.on('message', messageHandler);
    workerInfo.worker.on('error', errorHandler);
    workerInfo.worker.postMessage(data);
  }

  terminate() {
    this.workers.forEach(w => w.worker.terminate());
  }
}

// worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (data) => {
  // Calcul lourd
  const result = heavyComputation(data);
  parentPort.postMessage(result);
});

// Utilisation
const pool = new WorkerPool('./worker.js', 4);

const results = await Promise.all([
  pool.execute({ task: 'compute', value: 100 }),
  pool.execute({ task: 'compute', value: 200 }),
  pool.execute({ task: 'compute', value: 300 })
]);
```

## Monitoring

### Métriques de performance

```javascript
class PerformanceMetrics {
  constructor() {
    this.metrics = {
      requests: 0,
      errors: 0,
      totalDuration: 0,
      avgDuration: 0,
      minDuration: Infinity,
      maxDuration: 0
    };
  }

  record(duration, isError = false) {
    this.metrics.requests++;

    if (isError) {
      this.metrics.errors++;
    }

    this.metrics.totalDuration += duration;
    this.metrics.avgDuration = this.metrics.totalDuration / this.metrics.requests;
    this.metrics.minDuration = Math.min(this.metrics.minDuration, duration);
    this.metrics.maxDuration = Math.max(this.metrics.maxDuration, duration);
  }

  getMetrics() {
    return {
      ...this.metrics,
      errorRate: (this.metrics.errors / this.metrics.requests) * 100,
      requestsPerSecond: this.metrics.requests / (process.uptime() || 1)
    };
  }

  reset() {
    this.metrics = {
      requests: 0,
      errors: 0,
      totalDuration: 0,
      avgDuration: 0,
      minDuration: Infinity,
      maxDuration: 0
    };
  }
}

// Middleware Express
const metrics = new PerformanceMetrics();

app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    const isError = res.statusCode >= 400;
    metrics.record(duration, isError);
  });

  next();
});

app.get('/metrics', (req, res) => {
  res.json(metrics.getMetrics());
});
```

## Résumé

Points clés pour les performances :

- ⚡ Mesurer avant d'optimiser
- 💾 Utiliser le cache intelligemment
- 🗄️ Optimiser les requêtes DB
- 🌐 Minimiser les appels réseau
- 🔄 Paralléliser quand possible
- 📊 Monitorer en continu

## Prochaines étapes

- Consultez les [bonnes pratiques](best-practices.md)
- Découvrez la [sécurité](security.md)
- Explorez les [cas d'usage](../tutorials/use-cases.md)
