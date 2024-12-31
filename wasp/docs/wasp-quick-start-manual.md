# Wasp Quick Start Manual

> A modular guide to quickly start and develop your Wasp project

## 📚 Table of Contents
1. [Quick Setup](#quick-setup)
2. [Project Development](#project-development)
3. [Common Tasks](#common-tasks)
4. [Troubleshooting](#troubleshooting)

## 🚀 Quick Setup

### Prerequisites
- Node.js 16.x+
- npm or yarn
- Code editor (VSCode recommended)

### 1. Install Wasp
```bash
# Windows (PowerShell)
iwr https://get.wasp-lang.dev -useb | iex

# Verify installation
wasp --version
```

### 2. Create Project
```bash
# Initialize project
wasp new my-project

# Navigate to project
cd my-project
```

### 3. Start Development
```bash
# Start development server
wasp start
```

## 🔨 Project Development

### 1. Basic Configuration
Edit `main.wasp`:
```wasp
app MyProject {
  title: "My Wasp App"
}

route RootRoute { path: "/", component: MainPage }
page MainPage {
  component: import Main from "@client/MainPage.jsx"
}
```

### 2. Add Authentication
```wasp
app MyProject {
  auth: {
    userEntity: User,
    methods: {
      email: {}
    },
    onAuthFailedRedirectTo: "/login"
  }
}

entity User {=psl
  id       Int     @id @default(autoincrement())
  email    String  @unique
  password String
psl=}
```

### 3. Create Database Entity
```wasp
entity Task {=psl
  id          Int      @id @default(autoincrement())
  description String
  isDone      Boolean  @default(false)
  user        User     @relation(fields: [userId], references: [id])
  userId      Int
psl=}
```

## 🛠 Common Tasks

### Add a New Page
1. Define route in `main.wasp`:
```wasp
route TasksRoute { path: "/tasks", component: TasksPage }
page TasksPage {
  component: import Tasks from "@client/TasksPage.jsx"
}
```

2. Create component in `src/client/TasksPage.jsx`:
```jsx
import React from 'react'

export function TasksPage() {
  return <div>Tasks Page</div>
}
```

### Create Server Action
1. Define action in `main.wasp`:
```wasp
action createTask {
  fn: import { createTask } from "@server/actions.js",
  entities: [Task]
}
```

2. Implement in `src/server/actions.js`:
```javascript
export const createTask = async (args, context) => {
  if (!context.user) throw new Error('Not authorized')
  
  return context.entities.Task.create({
    data: {
      description: args.description,
      userId: context.user.id
    }
  })
}
```

### Add Database Query
1. Define query in `main.wasp`:
```wasp
query getTasks {
  fn: import { getTasks } from "@server/queries.js",
  entities: [Task]
}
```

2. Implement in `src/server/queries.js`:
```javascript
export const getTasks = async (args, context) => {
  if (!context.user) throw new Error('Not authorized')
  
  return context.entities.Task.findMany({
    where: { userId: context.user.id }
  })
}
```

## ⚡ Quick Tips

### Development
- Use `wasp start` for development
- Use `wasp db migrate-dev` for database changes
- Use `wasp build` for production build

### Database
- View database: `wasp db studio`
- Reset database: `wasp db reset`
- Create migration: `wasp db migrate-dev`

### Testing
- Run tests: `wasp test`
- Debug mode: `wasp start --debug`

## 🔧 Troubleshooting

### Common Issues

1. **Installation Failed**
   ```bash
   # Clear npm cache
   npm cache clean --force
   # Reinstall Wasp
   iwr https://get.wasp-lang.dev -useb | iex
   ```

2. **Database Issues**
   ```bash
   # Reset database
   wasp db reset
   # Regenerate migrations
   wasp db migrate-dev
   ```

3. **Build Errors**
   ```bash
   # Clean build
   wasp clean
   # Rebuild
   wasp build
   ```

## 📦 Next Steps

1. **Add Features**
   - Authentication
   - Database entities
   - API endpoints
   - Client components

2. **Optimize**
   - Add caching
   - Optimize queries
   - Implement error handling

3. **Deploy**
   - Build for production
   - Configure environment
   - Deploy application

## 🔗 Quick Links
- [Documentation](https://wasp-lang.dev/docs)
- [GitHub](https://github.com/wasp-lang/wasp)
- [Discord Community](https://discord.gg/rzdnErX)
