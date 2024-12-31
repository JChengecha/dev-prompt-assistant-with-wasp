# Wasp Troubleshooting Guide

## Common Issues & Solutions

### Installation Issues

#### 1. Wasp CLI Installation Fails
```bash
Error: Failed to install wasp-cli
```

**Solution**:
1. Check Node.js version:
```bash
node --version  # Should be >= 16.x
```

2. Clear npm cache:
```bash
npm cache clean --force
```

3. Install with specific version:
```bash
npm install -g wasp-lang@latest
```

#### 2. Database Connection Issues
```bash
Error: P1001: Can't reach database server
```

**Solution**:
1. Check DATABASE_URL in .env:
```env
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"
```

2. Verify PostgreSQL is running:
```bash
docker ps | grep postgres
# or
pg_isready -h localhost -p 5432
```

3. Reset database:
```bash
wasp db reset
```

### Build Issues

#### 1. TypeScript Errors
```bash
Error: Type error: Property 'x' does not exist on type 'y'
```

**Solution**:
1. Check tsconfig.json:
```json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "incremental": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

2. Update dependencies:
```bash
npm update
```

#### 2. Prisma Schema Errors
```bash
Error: P1012: Prisma schema validation error
```

**Solution**:
1. Validate schema:
```bash
npx prisma validate
```

2. Format schema:
```bash
npx prisma format
```

3. Reset database and migrations:
```bash
wasp db reset
```

### Runtime Issues

#### 1. Authentication Errors
```bash
Error: JWT verification failed
```

**Solution**:
1. Check JWT_SECRET in .env:
```env
JWT_SECRET="your-secret-key"
```

2. Verify auth configuration:
```wasp
auth {
  userEntity: User,
  methods: {
    usernameAndPassword: {}
  },
  onAuthFailedRedirectTo: "/login"
}
```

#### 2. API Endpoint Errors
```bash
Error: 404 Not Found
```

**Solution**:
1. Check route configuration:
```wasp
route DashboardRoute { path: "/dashboard", to: DashboardPage }
page DashboardPage {
  component: import Dashboard from "@client/pages/Dashboard.jsx"
}
```

2. Verify API endpoint implementation:
```javascript
// src/server/api.js
export const myApi = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  // Implementation
}
```

## Debugging Tips

### 1. Server-side Debugging

#### Using Console Logs
```javascript
// src/server/actions.js
export const myAction = async (args, context) => {
  console.log('Action args:', args)
  console.log('User context:', context.user)
  
  try {
    const result = await someOperation()
    console.log('Operation result:', result)
    return result
  } catch (error) {
    console.error('Operation failed:', error)
    throw error
  }
}
```

#### Using Debug Package
```javascript
// src/server/debug.js
import debug from 'debug'

const log = debug('app:server')
const dbLog = debug('app:db')
const authLog = debug('app:auth')

export const debugMiddleware = (req, res, next) => {
  log('Request:', req.method, req.url)
  next()
}
```

### 2. Client-side Debugging

#### React DevTools
1. Install React Developer Tools browser extension
2. Use Components tab to inspect state and props
3. Use Profiler tab to identify performance issues

#### Network Debugging
1. Use browser DevTools Network tab
2. Filter by XHR/Fetch requests
3. Inspect request/response data

### 3. Database Debugging

#### Prisma Studio
```bash
# Start Prisma Studio
npx prisma studio
```

#### Query Debugging
```javascript
// src/server/queries.js
const debugQuery = async (query) => {
  console.time('Query execution')
  const result = await query
  console.timeEnd('Query execution')
  return result
}

export const getUsers = async (args, context) => {
  return debugQuery(
    context.entities.User.findMany({
      include: { posts: true }
    })
  )
}
```

## Performance Monitoring

### 1. Server Monitoring
```javascript
// src/server/monitoring.js
export const performanceMiddleware = (req, res, next) => {
  const start = process.hrtime()
  
  res.on('finish', () => {
    const [seconds, nanoseconds] = process.hrtime(start)
    const duration = seconds * 1000 + nanoseconds / 1000000
    
    console.log(`${req.method} ${req.url} - ${duration}ms`)
    
    if (duration > 1000) {
      console.warn(`Slow request: ${req.method} ${req.url}`)
    }
  })
  
  next()
}
```

### 2. Client Monitoring
```javascript
// src/client/monitoring.js
export const trackPageLoad = () => {
  const navigation = performance.getEntriesByType('navigation')[0]
  const paint = performance.getEntriesByType('paint')
  
  console.log({
    loadTime: navigation.loadEventEnd - navigation.startTime,
    firstPaint: paint.find(p => p.name === 'first-paint')?.startTime,
    firstContentfulPaint: paint.find(p => p.name === 'first-contentful-paint')?.startTime
  })
}
```

## Common Error Patterns

### 1. Authentication Errors
- Check JWT token expiration
- Verify user session
- Validate auth headers

### 2. Database Errors
- Check connection string
- Verify schema migrations
- Handle concurrent operations

### 3. API Errors
- Validate request payload
- Handle timeout errors
- Implement retry logic

## Best Practices

### 1. Error Handling
```javascript
// src/server/errors.js
export class AppError extends Error {
  constructor(statusCode, message) {
    super(message)
    this.statusCode = statusCode
  }
}

export const errorHandler = (err, req, res, next) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: err.message
    })
  }
  
  console.error('Unhandled error:', err)
  res.status(500).json({
    error: 'Internal server error'
  })
}
```

### 2. Logging
```javascript
// src/server/logger.js
import winston from 'winston'

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
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
```

### 3. Testing
```javascript
// src/test/setup.js
import { setupServer } from 'msw/node'
import { rest } from 'msw'

export const server = setupServer(
  rest.get('/api/test', (req, res, ctx) => {
    return res(ctx.json({ message: 'Test response' }))
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```
