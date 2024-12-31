# Wasp Feature Development Workflow Guide

## Overview
This guide provides a structured approach to implementing features in a Wasp application, bridging the gap between requirements and actual implementation.

## 1. Requirements to Implementation Translation

### Feature Decomposition Process
1. **Break Down Requirements**
   - Identify core user stories
   - Define specific functionality
   - Determine data models and interactions

2. **Wasp-Specific Mapping**
   ```markdown
   Requirement → Wasp Implementation
   User Story → Routes/Pages
   Data Requirement → Prisma Entities
   User Action → Wasp Actions/Queries
   Authentication Needs → Auth Configuration
   ```

## 2. Wasp Entity Creation Workflow

### Entity Definition Template
```typescript
// Example: User Entity
entity User {
  id: number @id @autoincrement
  email: string @unique
  name: string
  role: UserRole
  posts: Post[]
}

enum UserRole {
  USER
  ADMIN
}
```

### Best Practices
- Use TypeScript for type safety
- Define clear relationships
- Implement validation constraints
- Consider performance implications

## 3. Action and Query Implementation

### Action Creation Pattern
```typescript
// src/server/actions.ts
action createPost {
  fn: import { createPostFn } from './postActions.js',
  entities: [Post, User]
}

// Corresponding server-side implementation
export const createPostFn = async (args, context) => {
  const newPost = await context.entities.Post.create({
    data: {
      title: args.title,
      content: args.content,
      author: { connect: { id: context.user.id } }
    }
  })
  return newPost
}
```

### Query Implementation
```typescript
query getPosts {
  fn: import { getPostsFn } from './postQueries.js',
  entities: [Post]
}

// Server-side query logic
export const getPostsFn = async (args, context) => {
  return context.entities.Post.findMany({
    where: { authorId: context.user.id },
    include: { author: true }
  })
}
```

## 4. Frontend Component Development

### Component Structure
```jsx
// src/client/pages/PostsPage.jsx
import React from 'react'
import { useQuery, useAction } from 'wasp'
import { getPosts, createPost } from 'wasp/server/actions'

const PostsPage = () => {
  const { data: posts, isLoading } = useQuery(getPosts)
  const createPostAction = useAction(createPost)

  const handleCreatePost = async (postData) => {
    await createPostAction(postData)
  }

  return (
    <div>
      {/* Render posts */}
      {/* Create post form */}
    </div>
  )
}
```

## 5. Feature Implementation Checklist

### Pre-Implementation
- [ ] Requirements are clearly defined
- [ ] Data models are designed
- [ ] Authentication requirements are understood
- [ ] Performance considerations are noted

### Implementation Steps
1. Define Prisma schema
2. Create server-side actions/queries
3. Implement frontend components
4. Add client-side state management
5. Implement error handling
6. Add input validation

### Post-Implementation
- [ ] Unit tests created
- [ ] Integration tests written
- [ ] Performance tested
- [ ] Security review completed

## 6. Common Pitfalls and Solutions

### Data Modeling Challenges
- **Pitfall**: Over-complicated entity relationships
- **Solution**: Start simple, normalize incrementally

### Performance Considerations
- Use Prisma's `select` and `include` judiciously
- Implement pagination
- Use database indexing

### Authentication Integration
```typescript
// Secure action example
action secureAction {
  fn: import { secureFn } from './secureActions.js',
  entities: [User],
  auth: true  // Requires authentication
}
```

## 7. Debugging and Troubleshooting

### Logging and Monitoring
```typescript
// Add contextual logging
console.log('Creating post', { 
  userId: context.user.id, 
  postData: args 
})
```

### Error Handling Patterns
```typescript
try {
  // Feature implementation
} catch (error) {
  // Log error
  console.error('Feature implementation failed', error)
  // Handle specific error types
  if (error instanceof ValidationError) {
    // Handle validation errors
  }
}
```

## 8. Feature Iteration Strategy

### Continuous Improvement
1. Implement MVP feature
2. Gather user feedback
3. Refactor and optimize
4. Add additional functionality

## 9. Integration Considerations

### External Service Integration
- Use dependency injection
- Create abstraction layers
- Implement robust error handling

## 10. Performance Optimization

### Query Optimization
- Use `select` to limit returned fields
- Implement caching strategies
- Use database indexing

## Conclusion

This workflow provides a comprehensive, step-by-step approach to feature development in Wasp, ensuring a structured and efficient implementation process.

### Next Steps
1. Review requirements
2. Design data model
3. Implement server-side logic
4. Create frontend components
5. Add tests and validation
6. Iterate and improve

### Resources
- [Wasp Documentation](https://wasp-lang.dev/docs)
- [Prisma ORM Docs](https://www.prisma.io/docs)
- [React Documentation](https://reactjs.org/docs)
