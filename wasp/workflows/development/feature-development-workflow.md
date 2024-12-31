# Feature Development Workflow

## Overview
Step-by-step guide for implementing features in a Wasp application.

## Workflow Steps

### 1. Feature Planning
```markdown
1. Feature Specification
   - User story
   - Acceptance criteria
   - Technical requirements

2. Component Design
   - UI/UX mockups
   - Data flow diagram
   - API endpoints

3. Database Schema
   - Entity relationships
   - Migration plan
   - Data validation rules
```

### 2. Implementation Process

#### 2.1 Entity Definition
```wasp
entity Feature {=psl
  id          Int       @id @default(autoincrement())
  name        String
  description String?
  status      String    @default("active")
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
psl=}
```

#### 2.2 API Layer
```wasp
query getFeature {
  fn: import { getFeature } from "@server/queries.js",
  entities: [Feature]
}

action updateFeature {
  fn: import { updateFeature } from "@server/actions.js",
  entities: [Feature]
}
```

#### 2.3 Server Implementation
```javascript
// src/server/queries.js
export const getFeature = async ({ id }, context) => {
  if (!context.user) throw new HttpError(401)
  
  return context.entities.Feature.findUnique({
    where: { id }
  })
}

// src/server/actions.js
export const updateFeature = async ({ id, data }, context) => {
  if (!context.user) throw new HttpError(401)
  
  return context.entities.Feature.update({
    where: { id },
    data
  })
}
```

#### 2.4 Client Components
```jsx
// src/client/pages/FeaturePage.jsx
import React from 'react'
import { useQuery } from '@wasp/queries'
import { useAction } from '@wasp/actions'
import { getFeature, updateFeature } from '@wasp/actions/feature'

export function FeaturePage({ id }) {
  const { data: feature, isLoading } = useQuery(getFeature, { id })
  const updateFeatureFn = useAction(updateFeature)

  if (isLoading) return 'Loading...'

  return (
    <div>
      <h1>{feature.name}</h1>
      <p>{feature.description}</p>
      <button onClick={() => updateFeatureFn({ id, data: { status: 'completed' } })}>
        Complete
      </button>
    </div>
  )
}
```

### 3. Testing Strategy

#### 3.1 Unit Tests
```javascript
// tests/unit/feature.test.js
describe('Feature', () => {
  it('updates status correctly', async () => {
    const feature = await createTestFeature()
    const updatedFeature = await updateFeature(
      { id: feature.id, data: { status: 'completed' } },
      { user: testUser, entities }
    )
    expect(updatedFeature.status).toBe('completed')
  })
})
```

#### 3.2 Integration Tests
```javascript
// tests/integration/feature.test.js
describe('Feature API', () => {
  it('returns 401 for unauthorized access', async () => {
    const response = await request(app)
      .get('/api/features/1')
    expect(response.status).toBe(401)
  })
})
```

### 4. Documentation

#### 4.1 API Documentation
```markdown
## Feature API

### GET /api/features/:id
Returns a specific feature.

Parameters:
- id: Feature ID (required)

Response:
```json
{
  "id": 1,
  "name": "Feature Name",
  "description": "Feature Description",
  "status": "active"
}
```
```

#### 4.2 Component Documentation
```markdown
## FeaturePage Component

Displays and manages a single feature.

Props:
- id: Feature ID (required)

State:
- feature: Current feature data
- isLoading: Loading state
- error: Error state

Actions:
- updateFeature: Updates feature status
```

### 5. Quality Checks

```markdown
## Feature Implementation Checklist

1. Code Quality
   - [ ] Follows project style guide
   - [ ] Properly documented
   - [ ] Error handling implemented
   - [ ] Performance considered

2. Testing
   - [ ] Unit tests written
   - [ ] Integration tests written
   - [ ] Edge cases covered
   - [ ] Performance tested

3. Security
   - [ ] Authentication implemented
   - [ ] Authorization checked
   - [ ] Input validation added
   - [ ] XSS prevention

4. UX/UI
   - [ ] Responsive design
   - [ ] Loading states
   - [ ] Error states
   - [ ] Success feedback
```
