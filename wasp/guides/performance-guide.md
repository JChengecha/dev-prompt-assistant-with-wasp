# Performance Guide

## 1. Database Optimization

### Query Optimization
```javascript
// src/server/db/optimization.js
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

### Indexing Strategy
```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())

  @@index([email])
  @@index([createdAt])
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  authorId  Int
  status    String
  createdAt DateTime @default(now())

  @@index([authorId, status])
  @@index([createdAt, status])
}
```

## 2. Caching

### Redis Cache
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

### Memory Cache
```javascript
// src/server/cache/memory.js
import LRU from 'lru-cache'

const cache = new LRU({
  max: 500,
  maxAge: 1000 * 60 * 60 // 1 hour
})

export const memoryCacheMiddleware = (req, res, next) => {
  const key = req.url
  const cached = cache.get(key)
  
  if (cached) {
    return res.json(cached)
  }
  
  res.sendResponse = res.json
  res.json = (body) => {
    cache.set(key, body)
    res.sendResponse(body)
  }
  
  next()
}
```

## 3. Frontend Optimization

### Code Splitting
```javascript
// src/client/routes/index.js
import { lazy, Suspense } from 'react'

const Dashboard = lazy(() => import('../pages/Dashboard'))
const Profile = lazy(() => import('../pages/Profile'))

export const Routes = () => (
  <Suspense fallback={<Loading />}>
    <Switch>
      <Route path="/dashboard" component={Dashboard} />
      <Route path="/profile" component={Profile} />
    </Switch>
  </Suspense>
)
```

### Image Optimization
```javascript
// src/client/components/Image.jsx
const Image = ({ src, alt, width, height }) => {
  const [loaded, setLoaded] = useState(false)
  
  return (
    <div className="relative">
      {!loaded && <Skeleton width={width} height={height} />}
      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        onLoad={() => setLoaded(true)}
        className={`transition-opacity duration-300 ${
          loaded ? 'opacity-100' : 'opacity-0'
        }`}
      />
    </div>
  )
}
```

## 4. API Optimization

### Response Compression
```javascript
// src/server/middleware/compression.js
import compression from 'compression'

export const compressionMiddleware = compression({
  level: 6,
  threshold: 100 * 1024, // 100kb
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false
    }
    return compression.filter(req, res)
  }
})
```

### Request Batching
```javascript
// src/server/api/batch.js
export const batchHandler = async (req, res) => {
  const { operations } = req.body
  
  const results = await Promise.all(
    operations.map(async (op) => {
      try {
        const result = await executeOperation(op)
        return { success: true, data: result }
      } catch (error) {
        return { success: false, error: error.message }
      }
    })
  )
  
  res.json({ results })
}
```

## 5. Performance Monitoring

### Server Monitoring
```javascript
// src/server/monitoring/performance.js
export const performanceMonitor = (req, res, next) => {
  const start = process.hrtime()
  
  res.on('finish', () => {
    const [seconds, nanoseconds] = process.hrtime(start)
    const duration = seconds * 1000 + nanoseconds / 1000000
    
    console.log({
      method: req.method,
      url: req.url,
      duration: `${duration}ms`,
      status: res.statusCode,
      contentLength: res.get('Content-Length')
    })
    
    if (duration > 1000) {
      console.warn(`Slow request: ${req.method} ${req.url}`)
    }
  })
  
  next()
}
```

### Client Monitoring
```javascript
// src/client/monitoring/performance.js
export const trackPageLoad = () => {
  const navigation = performance.getEntriesByType('navigation')[0]
  const paint = performance.getEntriesByType('paint')
  
  const metrics = {
    // Navigation timing
    loadTime: navigation.loadEventEnd - navigation.startTime,
    domContentLoaded: navigation.domContentLoadedEventEnd - navigation.startTime,
    
    // Paint timing
    firstPaint: paint.find(p => p.name === 'first-paint')?.startTime,
    firstContentfulPaint: paint.find(p => p.name === 'first-contentful-paint')?.startTime,
    
    // Resource timing
    resources: performance.getEntriesByType('resource').map(r => ({
      name: r.name,
      duration: r.duration,
      size: r.transferSize
    }))
  }
  
  console.log('Performance metrics:', metrics)
  
  // Send metrics to analytics
  sendToAnalytics('performance', metrics)
}
```

## 6. Performance Best Practices

### Database
```markdown
1. Use appropriate indexes
2. Optimize queries
3. Implement caching
4. Use connection pooling
5. Regular maintenance
```

### API
```markdown
1. Implement response compression
2. Use request batching
3. Cache responses
4. Optimize payload size
5. Use HTTP/2
```

### Frontend
```markdown
1. Code splitting
2. Lazy loading
3. Image optimization
4. Bundle optimization
5. Cache management
```

## 7. Performance Checklist

```markdown
### Database
- [ ] Indexes created for common queries
- [ ] Query performance analyzed
- [ ] Caching implemented
- [ ] Connection pooling configured
- [ ] Regular maintenance scheduled

### API
- [ ] Response compression enabled
- [ ] Request batching implemented
- [ ] API responses cached
- [ ] Payload size optimized
- [ ] HTTP/2 enabled

### Frontend
- [ ] Code splitting implemented
- [ ] Assets lazy loaded
- [ ] Images optimized
- [ ] Bundle size minimized
- [ ] Cache strategy defined

### Monitoring
- [ ] Performance metrics tracked
- [ ] Alerts configured
- [ ] Regular performance testing
- [ ] Resource usage monitored
- [ ] User experience metrics tracked
```
