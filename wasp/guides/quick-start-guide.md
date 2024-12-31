# Wasp Quick Start Guide

## 1. Initial Setup (5 minutes)

### Prerequisites
```bash
# Check Node.js version
node --version  # Should be >= 16.x

# Install Wasp CLI
npm install -g wasp-lang

# Verify installation
wasp --version
```

### Create Project
```bash
# Create new project
wasp new my-app
cd my-app

# Start development server
wasp start
```

## 2. Docker Setup (10 minutes)

### 1. Create Docker Files
```bash
# Create Dockerfile
cat > Dockerfile << EOL
FROM node:16-alpine
RUN npm install -g wasp-lang
WORKDIR /app
COPY . .
EXPOSE 3000 3001
CMD ["wasp", "start"]
EOL

# Create docker-compose.yml
cat > docker-compose.yml << EOL
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
      - "3001:3001"
    volumes:
      - .:/app
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/wasp_db
    depends_on:
      - db
  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=wasp_db
EOL
```

### 2. Start Development Environment
```bash
# Start containers
docker-compose up -d

# View logs
docker-compose logs -f
```

## 3. Basic App Setup (15 minutes)

### 1. Configure App (.wasp)
```wasp
app MyApp {
  title: "Quick Start App",
  dependencies: [
    ("@tailwindcss/forms", "^0.5.3")
  ]
}

route RootRoute { path: "/", to: MainPage }
page MainPage {
  component: import Main from "@client/MainPage.jsx"
}

entity Task {=psl
  id          Int     @id @default(autoincrement())
  description String
  isDone      Boolean @default(false)
psl=}

query getTasks {
  fn: import { getTasks } from "@server/queries.js",
  entities: [Task]
}

action createTask {
  fn: import { createTask } from "@server/actions.js",
  entities: [Task]
}
```

### 2. Create Components
```jsx
// src/client/MainPage.jsx
import React, { useState } from 'react'
import { useQuery } from '@wasp/queries'
import { useAction } from '@wasp/actions'
import { getTasks, createTask } from '@wasp/actions/tasks'

export function MainPage() {
  const [description, setDescription] = useState('')
  const { data: tasks, isLoading } = useQuery(getTasks)
  const createTaskFn = useAction(createTask)

  if (isLoading) return 'Loading...'

  return (
    <div className="max-w-md mx-auto mt-10">
      <form onSubmit={async (e) => {
        e.preventDefault()
        await createTaskFn({ description })
        setDescription('')
      }}>
        <input
          type="text"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          className="w-full px-3 py-2 border rounded"
        />
        <button type="submit" className="mt-2 px-4 py-2 bg-blue-500 text-white rounded">
          Add Task
        </button>
      </form>

      <ul className="mt-4">
        {tasks?.map((task) => (
          <li key={task.id} className="py-2">
            {task.description}
          </li>
        ))}
      </ul>
    </div>
  )
}
```

### 3. Implement Server Logic
```javascript
// src/server/queries.js
export const getTasks = async (args, context) => {
  return context.entities.Task.findMany({
    orderBy: { id: 'desc' }
  })
}

// src/server/actions.js
export const createTask = async ({ description }, context) => {
  return context.entities.Task.create({
    data: { description }
  })
}
```

## 4. Version Control (5 minutes)

### 1. Initialize Git
```bash
git init
git add .
git commit -m "feat: initial commit"
```

### 2. Create .gitignore
```bash
cat > .gitignore << EOL
node_modules/
.wasp/
.env
dist/
*.log
EOL
```

## 5. Next Steps

1. Add authentication:
```wasp
auth {
  userEntity: User,
  methods: {
    usernameAndPassword: {}
  },
  onAuthFailedRedirectTo: "/login"
}
```

2. Add database migrations:
```bash
wasp db migrate-dev
```

3. Deploy:
```bash
wasp build
```

## Common Issues

### 1. Database Connection
```bash
# Check database
docker-compose exec db psql -U postgres -d wasp_db

# Reset database
wasp db reset
```

### 2. Port Conflicts
```bash
# Check ports
netstat -ano | findstr :3000
netstat -ano | findstr :3001

# Stop containers
docker-compose down
```

### 3. Build Issues
```bash
# Clean build
rm -rf .wasp
wasp clean

# Rebuild
wasp build
```
