# Sécurité

## Introduction

Ce guide présente les meilleures pratiques de sécurité pour protéger vos applications et vos données.

## Authentification

### Hashing de mots de passe

✅ **Bien** :

```javascript
const bcrypt = require('bcrypt');

async function hashPassword(password) {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
}

async function verifyPassword(password, hash) {
  return await bcrypt.compare(password, hash);
}

// Utilisation
const hashedPassword = await hashPassword('MySecurePassword123!');
const isValid = await verifyPassword('MySecurePassword123!', hashedPassword);
```

❌ **À éviter** :

```javascript
// JAMAIS utiliser MD5 ou SHA1 pour les mots de passe
const crypto = require('crypto');
const hash = crypto.createHash('md5').update(password).digest('hex');
```

### Politique de mots de passe

```javascript
class PasswordPolicy {
  static validate(password) {
    const errors = [];

    if (password.length < 8) {
      errors.push('Le mot de passe doit contenir au moins 8 caractères');
    }

    if (!/[a-z]/.test(password)) {
      errors.push('Le mot de passe doit contenir au moins une minuscule');
    }

    if (!/[A-Z]/.test(password)) {
      errors.push('Le mot de passe doit contenir au moins une majuscule');
    }

    if (!/[0-9]/.test(password)) {
      errors.push('Le mot de passe doit contenir au moins un chiffre');
    }

    if (!/[!@#$%^&*]/.test(password)) {
      errors.push('Le mot de passe doit contenir au moins un caractère spécial');
    }

    // Vérifier les mots de passe communs
    const commonPasswords = ['password', '12345678', 'qwerty', 'admin'];
    if (commonPasswords.includes(password.toLowerCase())) {
      errors.push('Ce mot de passe est trop commun');
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }

  static calculateStrength(password) {
    let strength = 0;

    if (password.length >= 8) strength++;
    if (password.length >= 12) strength++;
    if (/[a-z]/.test(password)) strength++;
    if (/[A-Z]/.test(password)) strength++;
    if (/[0-9]/.test(password)) strength++;
    if (/[!@#$%^&*]/.test(password)) strength++;

    return {
      score: strength,
      level: strength < 3 ? 'faible' : strength < 5 ? 'moyen' : 'fort'
    };
  }
}

// Utilisation
const validation = PasswordPolicy.validate('Test123!');
const strength = PasswordPolicy.calculateStrength('MySecure123!Pass');
```

### Authentification multi-facteurs (2FA)

```javascript
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

class TwoFactorAuth {
  static generateSecret(user) {
    return speakeasy.generateSecret({
      name: `MyApp (${user.email})`,
      issuer: 'MyApp'
    });
  }

  static async generateQRCode(secret) {
    return await QRCode.toDataURL(secret.otpauth_url);
  }

  static verifyToken(secret, token) {
    return speakeasy.totp.verify({
      secret: secret.base32,
      encoding: 'base32',
      token: token,
      window: 2  // Accepter ±2 intervalles de temps
    });
  }
}

// Configuration
async function setup2FA(user) {
  const secret = TwoFactorAuth.generateSecret(user);
  const qrCode = await TwoFactorAuth.generateQRCode(secret);

  // Sauvegarder le secret (encrypté)
  await database.update(user.id, {
    twoFactorSecret: encrypt(secret.base32)
  });

  return {
    secret: secret.base32,
    qrCode
  };
}

// Vérification
async function verify2FA(user, token) {
  const secret = decrypt(user.twoFactorSecret);
  return TwoFactorAuth.verifyToken(secret, token);
}
```

## Protection contre les attaques

### SQL Injection

✅ **Bien** :

```javascript
// Requêtes paramétrées
async function getUser(email) {
  return await db.query(
    'SELECT * FROM users WHERE email = ?',
    [email]
  );
}

// ORM
const user = await User.findOne({
  where: { email: userEmail }
});
```

❌ **À éviter** :

```javascript
// Concaténation directe - VULNÉRABLE
async function getUser(email) {
  return await db.query(
    `SELECT * FROM users WHERE email = '${email}'`
  );
}
```

### XSS (Cross-Site Scripting)

```javascript
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

function sanitizeHTML(dirty) {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'target']
  });
}

// Utilisation
app.post('/posts', async (req, res) => {
  const post = {
    title: sanitizeHTML(req.body.title),
    content: sanitizeHTML(req.body.content)
  };

  await database.createPost(post);
  res.json({ success: true });
});

// Dans les templates (exemple avec EJS)
// <%- sanitizeHTML(user.bio) %>
```

### CSRF (Cross-Site Request Forgery)

```javascript
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

const app = express();

app.use(cookieParser());
app.use(csrf({ cookie: true }));

// Ajouter le token aux formulaires
app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

// Vérifier le token
app.post('/action', (req, res) => {
  // Le middleware csrf vérifie automatiquement
  res.json({ success: true });
});

// Gestion des erreurs CSRF
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    res.status(403).json({
      success: false,
      error: {
        code: 'INVALID_CSRF_TOKEN',
        message: 'Token CSRF invalide'
      }
    });
  } else {
    next(err);
  }
});
```

### Clickjacking

```javascript
const helmet = require('helmet');

app.use(helmet());

// Ou configuration manuelle
app.use((req, res, next) => {
  res.setHeader('X-Frame-Options', 'SAMEORIGIN');
  res.setHeader('Content-Security-Policy', "frame-ancestors 'self'");
  next();
});
```

## Sécurisation des APIs

### Rate limiting

```javascript
const rateLimit = require('express-rate-limit');

// Limite globale
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: 'Trop de requêtes, réessayez plus tard'
});

// Limite pour les endpoints sensibles
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: 'Trop de tentatives de connexion'
});

app.use('/api', globalLimiter);
app.use('/api/auth/login', authLimiter);

// Rate limiting par utilisateur
const userLimiter = (maxRequests, windowMs) => {
  const requests = new Map();

  return (req, res, next) => {
    const userId = req.user?.id || req.ip;
    const now = Date.now();

    if (!requests.has(userId)) {
      requests.set(userId, []);
    }

    const userRequests = requests.get(userId)
      .filter(time => now - time < windowMs);

    if (userRequests.length >= maxRequests) {
      return res.status(429).json({
        success: false,
        error: 'Rate limit exceeded'
      });
    }

    userRequests.push(now);
    requests.set(userId, userRequests);
    next();
  };
};

app.use('/api/expensive-operation', userLimiter(10, 60000));
```

### API Key validation

```javascript
class APIKeyValidator {
  constructor(database) {
    this.database = database;
    this.cache = new Map();
  }

  async validate(apiKey) {
    // Vérifier le cache
    if (this.cache.has(apiKey)) {
      const cached = this.cache.get(apiKey);
      if (cached.expiresAt > Date.now()) {
        return cached.data;
      }
      this.cache.delete(apiKey);
    }

    // Vérifier en base
    const key = await this.database.findAPIKey(apiKey);

    if (!key) {
      throw new Error('Invalid API key');
    }

    if (key.revokedAt) {
      throw new Error('API key revoked');
    }

    if (key.expiresAt && key.expiresAt < Date.now()) {
      throw new Error('API key expired');
    }

    // Mettre en cache
    this.cache.set(apiKey, {
      data: key,
      expiresAt: Date.now() + 60000  // Cache 1 minute
    });

    return key;
  }

  middleware() {
    return async (req, res, next) => {
      try {
        const apiKey = req.headers['x-api-key'];

        if (!apiKey) {
          return res.status(401).json({
            success: false,
            error: 'API key required'
          });
        }

        const key = await this.validate(apiKey);

        // Vérifier les scopes
        const requiredScope = req.route.scope;
        if (requiredScope && !key.scopes.includes(requiredScope)) {
          return res.status(403).json({
            success: false,
            error: 'Insufficient scope'
          });
        }

        req.apiKey = key;
        next();
      } catch (error) {
        res.status(401).json({
          success: false,
          error: error.message
        });
      }
    };
  }
}

const validator = new APIKeyValidator(database);
app.use('/api', validator.middleware());
```

## Encryption

### Chiffrement des données sensibles

```javascript
const crypto = require('crypto');

class Encryption {
  constructor(encryptionKey) {
    this.algorithm = 'aes-256-gcm';
    this.key = Buffer.from(encryptionKey, 'hex');
  }

  encrypt(text) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(this.algorithm, this.key, iv);

    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();

    return {
      encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }

  decrypt(encrypted, iv, authTag) {
    const decipher = crypto.createDecipheriv(
      this.algorithm,
      this.key,
      Buffer.from(iv, 'hex')
    );

    decipher.setAuthTag(Buffer.from(authTag, 'hex'));

    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return decrypted;
  }
}

// Utilisation
const encryption = new Encryption(process.env.ENCRYPTION_KEY);

// Chiffrer des données sensibles
const { encrypted, iv, authTag } = encryption.encrypt('Données sensibles');
await database.save({ encrypted, iv, authTag });

// Déchiffrer
const data = await database.load();
const decrypted = encryption.decrypt(data.encrypted, data.iv, data.authTag);
```

### HTTPS/TLS

```javascript
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem'),

  // Configuration TLS recommandée
  minVersion: 'TLSv1.2',
  ciphers: [
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384'
  ].join(':'),

  honorCipherOrder: true
};

const server = https.createServer(options, app);

server.listen(443, () => {
  console.log('HTTPS server running on port 443');
});

// Redirection HTTP vers HTTPS
const http = require('http');
http.createServer((req, res) => {
  res.writeHead(301, {
    'Location': `https://${req.headers.host}${req.url}`
  });
  res.end();
}).listen(80);
```

## Sécurité des sessions

### Configuration sécurisée

```javascript
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  name: 'sessionId',  // Nom personnalisé (pas "connect.sid")

  cookie: {
    secure: true,        // HTTPS uniquement
    httpOnly: true,      // Pas accessible via JavaScript
    sameSite: 'strict',  // Protection CSRF
    maxAge: 24 * 60 * 60 * 1000,  // 24 heures
    domain: '.example.com'
  },

  resave: false,
  saveUninitialized: false,

  rolling: true,  // Renouveler le cookie à chaque requête

  // Régénérer l'ID de session lors de la connexion
  genid: () => crypto.randomBytes(32).toString('hex')
}));

// Régénérer l'ID après authentification
app.post('/login', async (req, res) => {
  const user = await authenticateUser(req.body);

  if (user) {
    req.session.regenerate((err) => {
      if (err) {
        return res.status(500).json({ error: 'Session error' });
      }

      req.session.userId = user.id;
      res.json({ success: true });
    });
  }
});
```

## Headers de sécurité

```javascript
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  noSniff: true,
  xssFilter: true,
  referrerPolicy: { policy: 'same-origin' }
}));

// Headers personnalisés
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Referrer-Policy', 'same-origin');
  res.removeHeader('X-Powered-By');
  next();
});
```

## Audit et logging

### Logging sécurisé

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({
      filename: 'security.log',
      level: 'warn'
    })
  ]
});

// Logger les événements de sécurité
function logSecurityEvent(event, data) {
  logger.warn('Security event', {
    event,
    ...data,
    timestamp: new Date().toISOString()
  });
}

// Exemples
app.post('/login', async (req, res) => {
  const user = await authenticateUser(req.body);

  if (!user) {
    logSecurityEvent('failed_login', {
      email: req.body.email,
      ip: req.ip,
      userAgent: req.headers['user-agent']
    });
  } else {
    logSecurityEvent('successful_login', {
      userId: user.id,
      ip: req.ip
    });
  }
});
```

## Checklist de sécurité

### Développement

- [ ] Utiliser HTTPS partout
- [ ] Valider toutes les entrées utilisateur
- [ ] Utiliser des requêtes paramétrées
- [ ] Hasher les mots de passe avec bcrypt
- [ ] Implémenter le rate limiting
- [ ] Configurer les headers de sécurité
- [ ] Sanitizer les sorties HTML
- [ ] Utiliser CSRF protection
- [ ] Implémenter 2FA pour les comptes sensibles

### Déploiement

- [ ] Utiliser des variables d'environnement
- [ ] Ne jamais committer de secrets
- [ ] Configurer les logs de sécurité
- [ ] Mettre en place des alertes
- [ ] Auditer régulièrement les dépendances
- [ ] Maintenir les dépendances à jour
- [ ] Configurer les backups
- [ ] Tester les procédures de récupération

### Surveillance

- [ ] Monitorer les tentatives de connexion
- [ ] Détecter les comportements anormaux
- [ ] Auditer les accès aux données sensibles
- [ ] Vérifier les permissions régulièrement
- [ ] Scanner les vulnérabilités
- [ ] Analyser les logs de sécurité

## Ressources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [npm audit](https://docs.npmjs.com/cli/v8/commands/npm-audit)

## Prochaines étapes

- Consultez les [bonnes pratiques](best-practices.md)
- Découvrez les [optimisations de performance](performance.md)
- Explorez les [cas d'usage](../tutorials/use-cases.md)
