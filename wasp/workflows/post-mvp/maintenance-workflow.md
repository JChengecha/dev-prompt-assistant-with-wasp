# Post-MVP Maintenance Workflow

## 1. Monitoring & Maintenance

### 1.1 Performance Monitoring
```javascript
// src/server/monitoring/performance.js
import { performance } from 'perf_hooks'

export const monitorPerformance = (req, res, next) => {
  const start = performance.now()
  
  res.on('finish', () => {
    const duration = performance.now() - start
    console.log(`${req.method} ${req.url} - ${duration}ms`)
    
    if (duration > 1000) {
      notifySlowResponse({
        endpoint: req.url,
        duration,
        timestamp: new Date()
      })
    }
  })
  
  next()
}
```

### 1.2 Error Tracking
```javascript
// src/server/monitoring/errors.js
import * as Sentry from "@sentry/node"

export const setupErrorTracking = () => {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 1.0,
    beforeSend(event) {
      if (event.exception) {
        notifyTeam({
          type: 'error',
          message: event.exception.values[0].value,
          stack: event.exception.values[0].stacktrace
        })
      }
      return event
    }
  })
}
```

## 2. Updates & Patches

### 2.1 Database Migrations
```javascript
// migrations/20240101000000_update_schema.js
export const up = async (prisma) => {
  await prisma.$executeRaw`
    ALTER TABLE "User" 
    ADD COLUMN "lastLoginAt" TIMESTAMP;
  `
}

export const down = async (prisma) => {
  await prisma.$executeRaw`
    ALTER TABLE "User" 
    DROP COLUMN "lastLoginAt";
  `
}
```

### 2.2 Feature Updates
```wasp
// New feature implementation
entity FeatureFlag {=psl
  id          Int       @id @default(autoincrement())
  name        String    @unique
  isEnabled   Boolean   @default(false)
  createdAt   DateTime  @default(now())
psl=}

query getFeatureFlags {
  fn: import { getFeatureFlags } from "@server/queries.js",
  entities: [FeatureFlag]
}
```

## 3. Scaling Strategies

### 3.1 Database Optimization
```sql
-- Performance optimization
CREATE INDEX idx_user_email ON "User"(email);
CREATE INDEX idx_task_status ON "Task"(status);

-- Query optimization
SELECT u.*, COUNT(t.id) as taskCount
FROM "User" u
LEFT JOIN "Task" t ON u.id = t.userId
GROUP BY u.id;
```

### 3.2 Caching Implementation
```javascript
// src/server/cache/redis.js
import Redis from 'ioredis'

const redis = new Redis(process.env.REDIS_URL)

export const cacheMiddleware = async (req, res, next) => {
  const key = `cache:${req.url}`
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
}
```

## 4. Security Updates

### 4.1 Security Patches
```javascript
// src/server/middleware/security.js
import helmet from 'helmet'
import rateLimit from 'express-rate-limit'

export const securityMiddleware = [
  helmet(),
  rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100
  }),
  (req, res, next) => {
    res.setHeader('Content-Security-Policy', "default-src 'self'")
    next()
  }
]
```

### 4.2 Authentication Updates
```wasp
auth {
  userEntity: User,
  methods: {
    usernameAndPassword: {
      passwordMinLength: 10,
      passwordStrengthCheck: true
    },
    google: {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET
    }
  },
  onAuthFailedRedirectTo: "/login"
}
```

## 5. Backup & Recovery

### 5.1 Database Backup
```bash
#!/bin/bash
# backup.sh

# Set variables
BACKUP_DIR="/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DB_NAME="my_wasp_app"

# Create backup
pg_dump $DB_NAME > "$BACKUP_DIR/backup_$TIMESTAMP.sql"

# Compress backup
gzip "$BACKUP_DIR/backup_$TIMESTAMP.sql"

# Remove old backups (keep last 7 days)
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete
```

### 5.2 Recovery Procedures
```javascript
// src/server/recovery/restore.js
import { exec } from 'child_process'
import { promisify } from 'util'

const execAsync = promisify(exec)

export const restoreDatabase = async (backupFile) => {
  try {
    await execAsync(`pg_restore -d ${process.env.DB_NAME} ${backupFile}`)
    console.log('Database restored successfully')
  } catch (error) {
    console.error('Restore failed:', error)
    throw error
  }
}
```

## 6. Documentation Updates

### 6.1 API Documentation
```markdown
# API Documentation

## Endpoints

### GET /api/v2/users
Returns a list of users with pagination.

Parameters:
- page: Page number (default: 1)
- limit: Items per page (default: 10)
- sort: Sort field (default: 'createdAt')
- order: Sort order ('asc' or 'desc')

Response:
```json
{
  "users": [...],
  "total": 100,
  "page": 1,
  "totalPages": 10
}
```
```

### 6.2 Maintenance Documentation
```markdown
# Maintenance Procedures

## Daily Checks
1. Monitor error rates
2. Check API response times
3. Review system resources
4. Verify backup completion

## Weekly Tasks
1. Review performance metrics
2. Update documentation
3. Clean up old logs
4. Check security alerts

## Monthly Tasks
1. Security audit
2. Dependency updates
3. Performance optimization
4. Capacity planning
```

## 7. Performance Optimization

### 7.1 Frontend Optimization
```javascript
// src/client/optimization/lazyLoading.js
import { lazy, Suspense } from 'react'

const DashboardPage = lazy(() => import('../pages/Dashboard'))

export function LazyDashboard() {
  return (
    <Suspense fallback={<Loading />}>
      <DashboardPage />
    </Suspense>
  )
}
```

### 7.2 Backend Optimization
```javascript
// src/server/optimization/queryOptimization.js
export const optimizedQuery = async (args, context) => {
  const results = await context.entities.$queryRaw`
    SELECT 
      u.id,
      u.email,
      COUNT(t.id) as taskCount,
      MAX(t.updatedAt) as lastTaskUpdate
    FROM "User" u
    LEFT JOIN "Task" t ON u.id = t.userId
    WHERE u.isActive = true
    GROUP BY u.id
    HAVING COUNT(t.id) > 0
  `
  
  return results
}
```
