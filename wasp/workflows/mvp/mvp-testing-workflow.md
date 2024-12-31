# MVP Testing Workflow

## Overview
Comprehensive testing strategy for Wasp MVP applications.

## Testing Layers

### 1. Unit Testing
```javascript
// Example: Testing a task creation action
import { createTask } from './actions'

test('createTask creates a new task', async () => {
  const task = await createTask({
    description: 'Test task',
    userId: 1
  })
  expect(task.description).toBe('Test task')
})
```

### 2. Integration Testing
```javascript
// Example: Testing authentication flow
describe('Authentication Flow', () => {
  test('User can sign up and login', async () => {
    // Sign up
    const user = await signup({
      email: 'test@example.com',
      password: 'password123'
    })
    
    // Login
    const session = await login({
      email: 'test@example.com',
      password: 'password123'
    })
    
    expect(session.user.id).toBe(user.id)
  })
})
```

### 3. API Testing
```javascript
// Example: Testing REST endpoints
describe('Task API', () => {
  test('GET /api/tasks returns tasks', async () => {
    const response = await fetch('/api/tasks')
    const tasks = await response.json()
    expect(Array.isArray(tasks)).toBe(true)
  })
})
```

### 4. UI Testing
```javascript
// Example: Testing React components
import { render, screen } from '@testing-library/react'
import TaskList from './TaskList'

test('TaskList displays tasks', () => {
  const tasks = [
    { id: 1, description: 'Test task' }
  ]
  render(<TaskList tasks={tasks} />)
  expect(screen.getByText('Test task')).toBeInTheDocument()
})
```

## Performance Testing

### 1. Load Testing
```bash
# Using k6 for load testing
k6 run load-test.js
```

### 2. Response Time
```javascript
// Example: Response time monitoring
const startTime = performance.now()
const response = await fetch('/api/tasks')
const endTime = performance.now()
console.log(`Response time: ${endTime - startTime}ms`)
```

## User Acceptance Testing

### Test Cases Template
```markdown
Feature: Task Management
Scenario: Create New Task
  Given I am logged in
  When I create a new task with description "Test task"
  Then I should see the task in my task list
  And the task should have status "pending"
```

## Security Testing

### 1. Authentication Tests
```javascript
test('Protected routes require authentication', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})
```

### 2. Authorization Tests
```javascript
test('Users can only access their own data', async () => {
  const user1Tasks = await getTasks({ userId: 1 })
  expect(user1Tasks).toEqual(
    expect.not.arrayContaining([
      expect.objectContaining({ userId: 2 })
    ])
  )
})
```

## Validation Checklist

### Functionality
- [ ] All CRUD operations work
- [ ] Authentication flows properly
- [ ] Error handling is appropriate
- [ ] Data validation is correct

### Performance
- [ ] Page load under 3 seconds
- [ ] API responses under 200ms
- [ ] Database queries optimized
- [ ] No memory leaks

### Security
- [ ] Authentication working
- [ ] Authorization rules enforced
- [ ] Input validation complete
- [ ] No exposed sensitive data

### User Experience
- [ ] UI is responsive
- [ ] Error messages are clear
- [ ] Navigation is intuitive
- [ ] Forms work correctly
