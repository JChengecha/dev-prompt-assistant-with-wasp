# Advanced Scaling Guide

## 1. Database Scaling

### Connection Pooling
```javascript
// src/server/db.js
import { Pool } from 'pg'

const pool = new Pool({
  user: process.env.DB_USER,
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  password: process.env.DB_PASSWORD,
  port: process.env.DB_PORT,
  max: 20, // Maximum number of clients
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
})

export const query = async (text, params) => {
  const start = Date.now()
  const res = await pool.query(text, params)
  const duration = Date.now() - start
  
  console.log('executed query', { text, duration, rows: res.rowCount })
  return res
}
```

### Read Replicas
```javascript
// src/server/db/replica.js
const primary = new Pool({
  host: process.env.DB_PRIMARY_HOST,
  // ... other config
})

const replica = new Pool({
  host: process.env.DB_REPLICA_HOST,
  // ... other config
})

export const executeQuery = async (query, params, useReplica = false) => {
  const pool = useReplica ? replica : primary
  return pool.query(query, params)
}
```

## 2. Caching Strategies

### Redis Caching
```javascript
// src/server/cache/redis.js
import Redis from 'ioredis'

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
  password: process.env.REDIS_PASSWORD,
  retryStrategy: (times) => {
    const delay = Math.min(times * 50, 2000)
    return delay
  }
})

export const cacheMiddleware = async (req, res, next) => {
  const key = `cache:${req.url}`
  
  try {
    const cached = await redis.get(key)
    if (cached) {
      return res.json(JSON.parse(cached))
    }
    
    res.sendResponse = res.json
    res.json = (body) => {
      redis.set(key, JSON.stringify(body), 'EX', 3600)
      res.sendResponse(body)
    }
    
    next()
  } catch (error) {
    console.error('Cache error:', error)
    next()
  }
}
```

### Distributed Caching
```javascript
// src/server/cache/distributed.js
import Redis from 'ioredis'

const nodes = [
  { host: 'redis-1', port: 6379 },
  { host: 'redis-2', port: 6379 },
  { host: 'redis-3', port: 6379 }
]

const cluster = new Redis.Cluster(nodes, {
  redisOptions: {
    password: process.env.REDIS_PASSWORD
  },
  scaleReads: 'slave'
})

export const distributedCache = {
  async get(key) {
    return cluster.get(key)
  },
  
  async set(key, value, ttl = 3600) {
    return cluster.set(key, value, 'EX', ttl)
  },
  
  async del(key) {
    return cluster.del(key)
  }
}
```

## 3. Load Balancing

### Nginx Configuration
```nginx
# nginx.conf
upstream wasp_backend {
    least_conn;  # Least connections algorithm
    server backend1.example.com:3000;
    server backend2.example.com:3000;
    server backend3.example.com:3000;
    
    keepalive 32;  # Keep connections alive
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://wasp_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        
        # Load balancing improvements
        proxy_next_upstream error timeout invalid_header http_500;
        proxy_connect_timeout 2;
    }
    
    # Static file caching
    location /static/ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

### Health Checks
```javascript
// src/server/health.js
export const healthCheck = async (req, res) => {
  try {
    // Check database
    await context.entities.User.count()
    
    // Check cache
    await redis.ping()
    
    // Check external services
    await checkExternalServices()
    
    res.json({ status: 'healthy' })
  } catch (error) {
    res.status(500).json({
      status: 'unhealthy',
      error: error.message
    })
  }
}
```

## 4. Microservices Architecture

### Service Discovery
```javascript
// src/server/discovery.js
import consul from 'consul'

const client = new consul({
  host: process.env.CONSUL_HOST,
  port: process.env.CONSUL_PORT,
})

export const registerService = async () => {
  await client.agent.service.register({
    name: process.env.SERVICE_NAME,
    address: process.env.SERVICE_HOST,
    port: parseInt(process.env.SERVICE_PORT),
    check: {
      http: `http://${process.env.SERVICE_HOST}:${process.env.SERVICE_PORT}/health`,
      interval: '10s'
    }
  })
}

export const discoverService = async (serviceName) => {
  const result = await client.catalog.service.nodes(serviceName)
  return result[0]
}
```

### Message Queue
```javascript
// src/server/queue/rabbitmq.js
import amqp from 'amqplib'

export class MessageQueue {
  constructor() {
    this.connection = null
    this.channel = null
  }
  
  async connect() {
    this.connection = await amqp.connect(process.env.RABBITMQ_URL)
    this.channel = await this.connection.createChannel()
  }
  
  async publish(queue, message) {
    await this.channel.assertQueue(queue)
    return this.channel.sendToQueue(queue, Buffer.from(JSON.stringify(message)))
  }
  
  async subscribe(queue, callback) {
    await this.channel.assertQueue(queue)
    return this.channel.consume(queue, (msg) => {
      const content = JSON.parse(msg.content.toString())
      callback(content)
      this.channel.ack(msg)
    })
  }
}
```

## 5. Performance Optimization

### Query Optimization
```javascript
// src/server/optimization/query.js
export const optimizedQuery = async (args, context) => {
  // Use select to only fetch needed fields
  const result = await context.entities.User.findMany({
    select: {
      id: true,
      email: true,
      profile: {
        select: {
          name: true,
          avatar: true
        }
      }
    },
    where: {
      isActive: true
    }
  })

  return result
}

// Batch operations
export const batchOperation = async (items, context) => {
  return context.entities.User.createMany({
    data: items,
    skipDuplicates: true
  })
}
```

### Memory Management
```javascript
// src/server/optimization/memory.js
import v8 from 'v8'

export const monitorMemory = () => {
  const used = process.memoryUsage()
  
  console.log({
    rss: `${Math.round(used.rss / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)}MB`,
    heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)}MB`,
    external: `${Math.round(used.external / 1024 / 1024)}MB`,
  })
  
  const stats = v8.getHeapStatistics()
  console.log({
    heapSizeLimit: `${Math.round(stats.heap_size_limit / 1024 / 1024)}MB`,
    totalAvailable: `${Math.round(stats.total_available_size / 1024 / 1024)}MB`,
  })
}

// Garbage collection optimization
global.gc && global.gc()
```

## 6. Monitoring and Logging

### Prometheus Metrics
```javascript
// src/server/monitoring/metrics.js
import client from 'prom-client'

const register = new client.Registry()

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.1, 0.5, 1, 2, 5]
})

register.registerMetric(httpRequestDuration)

export const metricsMiddleware = (req, res, next) => {
  const start = process.hrtime()
  
  res.on('finish', () => {
    const duration = process.hrtime(start)
    const durationInSeconds = duration[0] + duration[1] / 1e9
    
    httpRequestDuration
      .labels(req.method, req.route.path, res.statusCode)
      .observe(durationInSeconds)
  })
  
  next()
}
```

### Structured Logging
```javascript
// src/server/logging/logger.js
import winston from 'winston'

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  defaultMeta: { service: 'wasp-app' },
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
})

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }))
}

export const loggerMiddleware = (req, res, next) => {
  const start = Date.now()
  
  res.on('finish', () => {
    logger.info({
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: Date.now() - start,
      userAgent: req.get('user-agent')
    })
  })
  
  next()
}
```

## 7. Security Enhancements

### Rate Limiting
```javascript
// src/server/security/rateLimit.js
import rateLimit from 'express-rate-limit'
import RedisStore from 'rate-limit-redis'
import Redis from 'ioredis'

const redis = new Redis(process.env.REDIS_URL)

export const createRateLimiter = (options = {}) => {
  return rateLimit({
    store: new RedisStore({
      client: redis,
      prefix: 'rate-limit:'
    }),
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    ...options
  })
}

// Apply different limits to different routes
export const apiLimiter = createRateLimiter({ max: 100 })
export const authLimiter = createRateLimiter({ max: 5 })
```

### JWT Enhancement
```javascript
// src/server/security/jwt.js
import jwt from 'jsonwebtoken'
import { createHash } from 'crypto'

const generateFingerprint = () => {
  return createHash('sha256')
    .update(Math.random().toString())
    .digest('hex')
}

export const createTokens = async (user) => {
  const fingerprint = generateFingerprint()
  const fingerprintHash = createHash('sha256')
    .update(fingerprint)
    .digest('hex')
  
  const accessToken = jwt.sign(
    {
      userId: user.id,
      hash: fingerprintHash
    },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  )
  
  const refreshToken = jwt.sign(
    {
      userId: user.id,
      hash: fingerprintHash,
      version: user.tokenVersion
    },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' }
  )
  
  return {
    accessToken,
    refreshToken,
    fingerprint
  }
}
```

## 8. Testing at Scale

### Load Testing
```javascript
// tests/load/scenarios.js
import { check, sleep } from 'k6'
import http from 'k6/http'

export const options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up
    { duration: '5m', target: 100 }, // Stay at peak
    { duration: '2m', target: 0 }    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests must complete below 500ms
    http_req_failed: ['rate<0.01']    // Less than 1% can fail
  }
}

export default function () {
  const BASE_URL = 'http://test.loadenv.com'
  
  // Login request
  const loginRes = http.post(`${BASE_URL}/auth/login`, {
    username: 'testuser',
    password: 'testpass'
  })
  
  check(loginRes, {
    'logged in successfully': (resp) => resp.json('token') !== ''
  })
  
  // Get data
  const authToken = loginRes.json('token')
  const headers = { Authorization: `Bearer ${authToken}` }
  
  const dataRes = http.get(`${BASE_URL}/api/data`, { headers })
  
  check(dataRes, {
    'status is 200': (r) => r.status === 200
  })
  
  sleep(1)
}
```

### Integration Testing
```javascript
// tests/integration/api.test.js
import request from 'supertest'
import { app } from '../../src/server/app'
import { redis } from '../../src/server/cache'
import { queue } from '../../src/server/queue'

describe('API Integration', () => {
  beforeAll(async () => {
    await redis.connect()
    await queue.connect()
  })
  
  afterAll(async () => {
    await redis.disconnect()
    await queue.disconnect()
  })
  
  it('handles concurrent requests', async () => {
    const requests = Array(10).fill().map(() =>
      request(app)
        .post('/api/data')
        .send({ value: 'test' })
    )
    
    const responses = await Promise.all(requests)
    
    responses.forEach(response => {
      expect(response.status).toBe(200)
    })
  })
})
```
