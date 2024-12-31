# Security Guide

## 1. Authentication & Authorization

### JWT Configuration
```javascript
// src/server/auth/jwt.js
import jwt from 'jsonwebtoken'

const JWT_SECRET = process.env.JWT_SECRET
const JWT_EXPIRY = '24h'

export const generateToken = (user) => {
  return jwt.sign(
    { id: user.id, email: user.email },
    JWT_SECRET,
    { expiresIn: JWT_EXPIRY }
  )
}

export const verifyToken = (token) => {
  return jwt.verify(token, JWT_SECRET)
}
```

### Auth Middleware
```javascript
// src/server/middleware/auth.js
export const authMiddleware = async (req, res, next) => {
  try {
    const token = req.headers.authorization?.split(' ')[1]
    if (!token) throw new Error('No token provided')

    const decoded = verifyToken(token)
    req.user = decoded
    next()
  } catch (error) {
    res.status(401).json({ error: 'Unauthorized' })
  }
}
```

### Role-Based Access
```wasp
entity User {=psl
  id          Int       @id @default(autoincrement())
  email       String    @unique
  password    String
  role        String    @default("user")
psl=}

query adminQuery {
  fn: import { adminQuery } from "@server/queries.js",
  entities: [User]
}

// src/server/middleware/rbac.js
export const checkRole = (role) => (req, res, next) => {
  if (req.user.role !== role) {
    return res.status(403).json({ error: 'Forbidden' })
  }
  next()
}
```

## 2. Data Protection

### Input Validation
```javascript
// src/server/validation/schema.js
import { z } from 'zod'

export const userSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/[A-Z]/).regex(/[0-9]/)
})

export const validateInput = (schema) => (data) => {
  return schema.parse(data)
}
```

### SQL Injection Prevention
```javascript
// Using Prisma's built-in protection
export const safeQuery = async (args, context) => {
  return context.entities.User.findMany({
    where: {
      email: { contains: args.search }
    }
  })
}
```

### XSS Prevention
```javascript
// src/server/middleware/security.js
import helmet from 'helmet'

export const securityMiddleware = [
  helmet(),
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  })
]
```

## 3. API Security

### Rate Limiting
```javascript
// src/server/middleware/rateLimit.js
import rateLimit from 'express-rate-limit'

export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later'
})
```

### CORS Configuration
```javascript
// src/server/middleware/cors.js
import cors from 'cors'

export const corsOptions = {
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true
}
```

## 4. Environment Security

### Environment Variables
```env
# .env.example
NODE_ENV=development
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-secret-key
ALLOWED_ORIGINS=http://localhost:3000
REDIS_URL=redis://localhost:6379
```

### Production Configuration
```javascript
// src/config/production.js
export const productionConfig = {
  ssl: true,
  cookieSecure: true,
  sessionDuration: '1h',
  logLevel: 'error',
  rateLimits: {
    api: 100,
    auth: 5
  }
}
```

## 5. Data Encryption

### Password Hashing
```javascript
// src/server/auth/password.js
import bcrypt from 'bcrypt'

const SALT_ROUNDS = 10

export const hashPassword = async (password) => {
  return bcrypt.hash(password, SALT_ROUNDS)
}

export const verifyPassword = async (password, hash) => {
  return bcrypt.compare(password, hash)
}
```

### Data Encryption
```javascript
// src/server/encryption/crypto.js
import crypto from 'crypto'

const ALGORITHM = 'aes-256-gcm'
const SECRET_KEY = Buffer.from(process.env.ENCRYPTION_KEY, 'hex')

export const encrypt = (text) => {
  const iv = crypto.randomBytes(16)
  const cipher = crypto.createCipheriv(ALGORITHM, SECRET_KEY, iv)
  
  let encrypted = cipher.update(text, 'utf8', 'hex')
  encrypted += cipher.final('hex')
  
  const authTag = cipher.getAuthTag()
  
  return {
    iv: iv.toString('hex'),
    encrypted,
    authTag: authTag.toString('hex')
  }
}

export const decrypt = ({ iv, encrypted, authTag }) => {
  const decipher = crypto.createDecipheriv(
    ALGORITHM,
    SECRET_KEY,
    Buffer.from(iv, 'hex')
  )
  
  decipher.setAuthTag(Buffer.from(authTag, 'hex'))
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8')
  decrypted += decipher.final('utf8')
  
  return decrypted
}
```

## 6. Security Best Practices

### 1. Password Requirements
```javascript
// src/server/validation/password.js
export const passwordRequirements = {
  minLength: 8,
  requireUppercase: true,
  requireNumbers: true,
  requireSpecialChars: true
}

export const validatePassword = (password) => {
  const checks = [
    {
      test: password.length >= passwordRequirements.minLength,
      message: `Password must be at least ${passwordRequirements.minLength} characters`
    },
    {
      test: /[A-Z]/.test(password),
      message: 'Password must contain at least one uppercase letter'
    },
    {
      test: /[0-9]/.test(password),
      message: 'Password must contain at least one number'
    },
    {
      test: /[!@#$%^&*]/.test(password),
      message: 'Password must contain at least one special character'
    }
  ]

  const failures = checks
    .filter(check => !check.test)
    .map(check => check.message)

  return {
    isValid: failures.length === 0,
    errors: failures
  }
}
```

### 2. Session Management
```javascript
// src/server/auth/session.js
import session from 'express-session'
import RedisStore from 'connect-redis'

export const sessionConfig = {
  store: new RedisStore({
    url: process.env.REDIS_URL
  }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000 // 24 hours
  }
}
```

### 3. Security Headers
```javascript
// src/server/middleware/headers.js
export const securityHeaders = (req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff')
  res.setHeader('X-Frame-Options', 'DENY')
  res.setHeader('X-XSS-Protection', '1; mode=block')
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin')
  next()
}
```

## 7. Security Checklist

```markdown
### Authentication
- [ ] JWT implementation secure
- [ ] Password hashing implemented
- [ ] Role-based access control setup
- [ ] Session management configured

### Data Protection
- [ ] Input validation implemented
- [ ] SQL injection prevention active
- [ ] XSS protection enabled
- [ ] CSRF protection configured

### API Security
- [ ] Rate limiting implemented
- [ ] CORS properly configured
- [ ] Security headers set
- [ ] Request validation active

### Environment
- [ ] Environment variables secured
- [ ] Production config optimized
- [ ] Secrets management implemented
- [ ] SSL/TLS configured

### Monitoring
- [ ] Security logging enabled
- [ ] Error tracking setup
- [ ] Audit trail implemented
- [ ] Alerts configured
```
