# Comprehensive Wasp Testing Guide for MVP Development

## Overview
This guide provides a structured approach to testing a Wasp application, ensuring quality, reliability, and performance throughout the MVP development process.

## 1. Testing Strategy Overview

### Testing Pyramid for Wasp Applications
```
       [E2E Tests]        (Few tests)
     /             \
   [Integration Tests]    (Some tests)
 /                   \
[Unit Tests]           [Component Tests]  (Many tests)
```

### Testing Objectives
- Validate individual components
- Ensure system integration
- Verify end-to-end functionality
- Maintain code quality
- Catch potential issues early

## 2. Unit Testing in Wasp

### Server-Side Action/Query Testing
```typescript
// src/server/actions.test.ts
import { createPostAction } from './actions'
import { prismaMock } from './test-utils'

describe('createPostAction', () => {
  it('creates a new post successfully', async () => {
    const mockPost = {
      id: 1,
      title: 'Test Post',
      content: 'Test Content'
    }

    prismaMock.post.create.mockResolvedValue(mockPost)

    const result = await createPostAction({
      title: 'Test Post',
      content: 'Test Content'
    }, mockContext)

    expect(result).toEqual(mockPost)
    expect(prismaMock.post.create).toHaveBeenCalledWith({
      data: {
        title: 'Test Post',
        content: 'Test Content'
      }
    })
  })

  it('handles validation errors', async () => {
    await expect(createPostAction({
      title: '',
      content: ''
    }, mockContext)).rejects.toThrow(ValidationError)
  })
})
```

### Frontend Component Testing
```jsx
// src/client/components/PostForm.test.jsx
import { render, fireEvent, waitFor } from '@testing-library/react'
import PostForm from './PostForm'
import { useAction } from 'wasp'

jest.mock('wasp', () => ({
  useAction: jest.fn()
}))

describe('PostForm Component', () => {
  it('submits form with valid data', async () => {
    const mockCreatePost = jest.fn()
    useAction.mockReturnValue(mockCreatePost)

    const { getByLabelText, getByText } = render(<PostForm />)

    fireEvent.change(getByLabelText('Title'), { 
      target: { value: 'Test Post' } 
    })
    fireEvent.change(getByLabelText('Content'), { 
      target: { value: 'Test Content' } 
    })

    fireEvent.click(getByText('Submit'))

    await waitFor(() => {
      expect(mockCreatePost).toHaveBeenCalledWith({
        title: 'Test Post',
        content: 'Test Content'
      })
    })
  })
})
```

## 3. Integration Testing

### API and Database Integration
```typescript
// src/server/integrations.test.ts
import { testDbConnection } from './testUtils'
import { createUser, authenticateUser } from './authActions'

describe('Authentication Integration', () => {
  beforeAll(async () => {
    await testDbConnection()
  })

  it('creates user and authenticates successfully', async () => {
    const userData = {
      email: 'test@example.com',
      password: 'securePassword123'
    }

    const user = await createUser(userData)
    const authResult = await authenticateUser(userData)

    expect(user).toBeDefined()
    expect(authResult.token).toBeTruthy()
  })
})
```

## 4. End-to-End (E2E) Testing

### Cypress E2E Test Example
```typescript
// cypress/integration/user-flow.spec.ts
describe('User Authentication Flow', () => {
  it('allows user to register, login, and create a post', () => {
    // Registration
    cy.visit('/register')
    cy.get('input[name="email"]').type('newuser@example.com')
    cy.get('input[name="password"]').type('securePassword123')
    cy.get('button[type="submit"]').click()

    // Login verification
    cy.url().should('include', '/dashboard')

    // Create post
    cy.get('a[href="/create-post"]').click()
    cy.get('input[name="title"]').type('E2E Test Post')
    cy.get('textarea[name="content"]').type('This is an end-to-end test post')
    cy.get('button[type="submit"]').click()

    // Verify post creation
    cy.get('.post-list').should('contain', 'E2E Test Post')
  })
})
```

## 5. Performance Testing

### Load Testing Strategy
```typescript
// performance-test.js
import { performance } from 'k6'
import http from 'k6/http'

export const options = {
  vus: 50,  // 50 virtual users
  duration: '5m',  // 5-minute test
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests under 500ms
    http_req_failed: ['rate<0.01']  // Less than 1% request failure
  }
}

export default function () {
  const payload = JSON.stringify({
    title: 'Performance Test Post',
    content: 'Testing system load'
  })

  const params = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${__ENV.API_TOKEN}`
    }
  }

  http.post('https://yourapp.com/api/posts', payload, params)
}
```

## 6. Security Testing Checklist

### Authentication Tests
- [ ] Validate user registration
- [ ] Test password complexity
- [ ] Verify JWT token generation
- [ ] Check role-based access control

### Input Validation
- [ ] Test XSS prevention
- [ ] Validate SQL injection protection
- [ ] Check input length limits
- [ ] Verify data sanitization

## 7. Test Configuration

### Testing Tools
- Jest for unit and integration testing
- React Testing Library for component tests
- Cypress for E2E testing
- K6 for performance testing

### Configuration Example
```json
{
  "jest": {
    "testEnvironment": "node",
    "setupFiles": ["./jest.setup.js"],
    "collectCoverage": true,
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": -10
      }
    }
  }
}
```

## 8. Continuous Integration

### GitHub Actions Workflow
```yaml
name: Wasp Application CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      
      - name: Install Dependencies
        run: npm ci
      
      - name: Run Unit Tests
        run: npm run test:unit
      
      - name: Run Integration Tests
        run: npm run test:integration
      
      - name: Run E2E Tests
        run: npm run test:e2e
      
      - name: Performance Testing
        run: npm run test:performance
```

## 9. Test Maintenance

### Best Practices
- Keep tests independent
- Use mock data
- Regularly update test cases
- Monitor test coverage
- Refactor tests with code changes

## Conclusion

A comprehensive testing strategy is crucial for MVP success. This guide provides a structured approach to testing Wasp applications, covering unit, integration, E2E, performance, and security testing.

### Next Steps
1. Set up testing infrastructure
2. Write initial test cases
3. Integrate with CI/CD
4. Continuously improve test coverage

### Resources
- [Wasp Documentation](https://wasp-lang.dev/docs)
- [Jest Documentation](https://jestjs.io/)
- [Cypress Documentation](https://docs.cypress.io/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
