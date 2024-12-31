# Testing Guide

## 1. Unit Testing

### Test Setup
```javascript
// src/test/setup.js
import { setupServer } from 'msw/node'
import { rest } from 'msw'
import '@testing-library/jest-dom'

export const server = setupServer(
  rest.get('/api/tasks', (req, res, ctx) => {
    return res(ctx.json([
      { id: 1, description: 'Test task', isDone: false }
    ]))
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

### Component Tests
```javascript
// src/client/components/TaskList.test.jsx
import { render, screen } from '@testing-library/react'
import { TaskList } from './TaskList'

describe('TaskList', () => {
  it('renders tasks', async () => {
    render(<TaskList />)
    
    expect(await screen.findByText('Test task')).toBeInTheDocument()
  })
})
```

### API Tests
```javascript
// src/server/queries.test.js
import { getTasks } from './queries'

describe('getTasks', () => {
  it('returns tasks', async () => {
    const context = {
      entities: {
        Task: {
          findMany: jest.fn().mockResolvedValue([
            { id: 1, description: 'Test task', isDone: false }
          ])
        }
      }
    }

    const tasks = await getTasks({}, context)
    expect(tasks).toHaveLength(1)
    expect(tasks[0].description).toBe('Test task')
  })
})
```

## 2. Integration Testing

### API Integration
```javascript
// src/test/api.test.js
import request from 'supertest'
import { app } from '../server/app'

describe('API', () => {
  it('creates task', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ description: 'New task' })
    
    expect(response.status).toBe(200)
    expect(response.body.description).toBe('New task')
  })
})
```

### Database Integration
```javascript
// src/test/db.test.js
import { prisma } from '../server/db'

describe('Database', () => {
  beforeEach(async () => {
    await prisma.task.deleteMany()
  })

  it('creates and retrieves task', async () => {
    await prisma.task.create({
      data: { description: 'Test task' }
    })

    const tasks = await prisma.task.findMany()
    expect(tasks).toHaveLength(1)
  })
})
```

## 3. E2E Testing

### Cypress Setup
```javascript
// cypress/support/commands.js
Cypress.Commands.add('login', (email, password) => {
  cy.visit('/login')
  cy.get('input[name=email]').type(email)
  cy.get('input[name=password]').type(password)
  cy.get('button[type=submit]').click()
})
```

### E2E Tests
```javascript
// cypress/integration/tasks.spec.js
describe('Tasks', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password')
  })

  it('creates task', () => {
    cy.visit('/')
    cy.get('input[name=description]').type('New task')
    cy.get('button[type=submit]').click()
    cy.contains('New task').should('be.visible')
  })
})
```

## 4. Performance Testing

### Load Testing
```javascript
// tests/load/tasks.js
import http from 'k6/http'
import { check, sleep } from 'k6'

export const options = {
  vus: 10,
  duration: '30s',
}

export default function () {
  const res = http.get('http://localhost:3000/api/tasks')
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  })

  sleep(1)
}
```

### API Performance
```javascript
// src/server/middleware/performance.js
export const performanceMiddleware = (req, res, next) => {
  const start = process.hrtime()
  
  res.on('finish', () => {
    const [seconds, nanoseconds] = process.hrtime(start)
    const duration = seconds * 1000 + nanoseconds / 1000000
    
    console.log(`${req.method} ${req.url} - ${duration}ms`)
  })
  
  next()
}
```

## 5. Test Configuration

### Jest Config
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/test/setup.js'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/test/**',
  ],
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
}
```

### Cypress Config
```javascript
// cypress.json
{
  "baseUrl": "http://localhost:3000",
  "viewportWidth": 1280,
  "viewportHeight": 720,
  "video": false,
  "screenshotOnRunFailure": true,
  "integrationFolder": "cypress/integration",
  "supportFile": "cypress/support/index.js",
  "pluginsFile": "cypress/plugins/index.js",
  "fixturesFolder": "cypress/fixtures"
}
```

## 6. CI Integration

### GitHub Actions
```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm test
        
      - name: Run E2E tests
        run: |
          npm start & npx wait-on http://localhost:3000
          npm run test:e2e
        
      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

## 7. Test Scripts

### Package.json
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:e2e": "cypress run",
    "test:e2e:open": "cypress open",
    "test:load": "k6 run tests/load/tasks.js"
  }
}
```
