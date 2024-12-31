# MVP Deployment Guide

## Overview
Comprehensive guide for deploying a Wasp MVP to production.

## Pre-Deployment Checklist

### 1. Environment Setup
```bash
# Create production environment file
cat > .env.production << EOL
DATABASE_URL=postgresql://user:pass@host:5432/db
JWT_SECRET=your-secret-key
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
EOL
```

### 2. Database Migration
```bash
# Generate production migrations
wasp db migrate-prod

# Apply migrations
wasp db deploy
```

### 3. Build Configuration
```wasp
app MyApp {
  title: "Production App",
  head: [
    "<meta property='og:title' content='My Wasp App'/>",
    "<meta property='og:description' content='Built with Wasp'/>"
  ],
  client: {
    rootComponent: import Root from "@client/Root.jsx",
  },
  db: {
    system: PostgreSQL,
    prisma: {
      clientPreviewFeatures: ["extendedWhereUnique"]
    }
  }
}
```

## Deployment Steps

### 1. Platform Setup (Example: Heroku)
```bash
# Login to Heroku
heroku login

# Create new app
heroku create my-wasp-app

# Add PostgreSQL
heroku addons:create heroku-postgresql:hobby-dev
```

### 2. Environment Configuration
```bash
# Set environment variables
heroku config:set NODE_ENV=production
heroku config:set JWT_SECRET=your-secret-key
heroku config:set DATABASE_URL=your-db-url
```

### 3. Application Deployment
```bash
# Build application
wasp build

# Deploy to Heroku
git push heroku main
```

## Production Monitoring

### 1. Application Monitoring
```javascript
// Example: Basic health check endpoint
route HealthCheckRoute { 
  path: "/health", 
  fn: import { healthCheck } from "@server/health.js"
}

// health.js
export const healthCheck = async (req, res) => {
  try {
    await prisma.$queryRaw`SELECT 1`
    return { status: 'healthy' }
  } catch (error) {
    return { status: 'unhealthy', error: error.message }
  }
}
```

### 2. Error Tracking
```javascript
// Example: Error logging setup
import * as Sentry from "@sentry/node"

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV
})

app.use(Sentry.Handlers.requestHandler())
app.use(Sentry.Handlers.errorHandler())
```

### 3. Performance Monitoring
```javascript
// Example: Response time monitoring
app.use((req, res, next) => {
  const start = Date.now()
  res.on('finish', () => {
    const duration = Date.now() - start
    console.log(`${req.method} ${req.url} - ${duration}ms`)
  })
  next()
})
```

## Scaling Considerations

### 1. Database Optimization
```sql
-- Example: Add indexes for common queries
CREATE INDEX idx_user_email ON "User"(email);
CREATE INDEX idx_task_user ON "Task"(userId);
```

### 2. Caching Strategy
```javascript
// Example: Redis caching setup
import Redis from 'ioredis'

const redis = new Redis(process.env.REDIS_URL)

export const getCachedData = async (key) => {
  const cached = await redis.get(key)
  if (cached) return JSON.parse(cached)
  
  const data = await fetchData()
  await redis.set(key, JSON.stringify(data), 'EX', 3600)
  return data
}
```

### 3. Load Balancing
```nginx
# Example: Nginx load balancer configuration
upstream wasp_app {
  server 127.0.0.1:3000;
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
}

server {
  listen 80;
  server_name myapp.com;

  location / {
    proxy_pass http://wasp_app;
  }
}
```

## Security Measures

### 1. SSL Configuration
```bash
# Install SSL certificate
heroku certs:auto:enable

# Force SSL
heroku config:set FORCE_SSL=true
```

### 2. Security Headers
```javascript
// Example: Security middleware
app.use(helmet())
app.use(cors())
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
}))
```

## Deployment Checklist

### Pre-Launch
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] SSL certificate installed
- [ ] Security headers configured
- [ ] Monitoring tools setup

### Post-Launch
- [ ] Verify application health
- [ ] Monitor error rates
- [ ] Check performance metrics
- [ ] Test critical flows
- [ ] Backup verification

## Rollback Plan

### 1. Version Control
```bash
# Tag releases
git tag v1.0.0

# Rollback to previous version
git checkout v0.9.0
wasp build
git push heroku v0.9.0:main
```

### 2. Database Rollback
```bash
# Revert last migration
wasp db migrate-prod reset

# Restore from backup
pg_restore -d database_name backup_file
```
