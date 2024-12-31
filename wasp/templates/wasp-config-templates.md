# Wasp Configuration Templates

## Basic Application Setup

### 1. Main Application Configuration
```wasp
app MyApp {
  title: "My Wasp Application",
  head: [
    "<meta name='description' content='Built with Wasp'/>",
    "<meta name='viewport' content='width=device-width, initial-scale=1'/>"
  ],
  dependencies: [
    ("react", "^18.2.0"),
    ("react-dom", "^18.2.0"),
    ("react-query", "^3.39.0"),
    ("tailwindcss", "^3.3.0")
  ]
}
```

### 2. Database Configuration
```wasp
app MyApp {
  db: {
    system: PostgreSQL,
    prisma: {
      clientPreviewFeatures: ["extendedWhereUnique"]
    }
  }
}
```

### 3. Authentication Setup
```wasp
auth {
  userEntity: User,
  methods: {
    usernameAndPassword: {},
    google: {
      clientID: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET"
    }
  },
  onAuthFailedRedirectTo: "/login"
}
```

## Entity Definitions

### 1. User Entity
```wasp
entity User {=psl
  id          Int       @id @default(autoincrement())
  email       String    @unique
  password    String
  username    String?
  createdAt   DateTime  @default(now())
  tasks       Task[]
psl=}
```

### 2. Task Entity
```wasp
entity Task {=psl
  id          Int       @id @default(autoincrement())
  description String
  isDone      Boolean   @default(false)
  createdAt   DateTime  @default(now())
  user        User      @relation(fields: [userId], references: [id])
  userId      Int
psl=}
```

## Route Configurations

### 1. Basic Routes
```wasp
route LoginRoute { path: "/login", to: LoginPage }
route SignupRoute { path: "/signup", to: SignupPage }
route DashboardRoute { path: "/dashboard", to: DashboardPage }
route TaskRoute { path: "/task/:taskId", to: TaskPage }
```

### 2. Protected Routes
```wasp
route DashboardRoute {
  path: "/dashboard",
  to: DashboardPage,
  authRequired: true
}
```

## Operations

### 1. Queries
```wasp
query getTasks {
  fn: import { getTasks } from "@server/queries.js",
  entities: [Task]
}

query getTask {
  fn: import { getTask } from "@server/queries.js",
  entities: [Task]
}
```

### 2. Actions
```wasp
action createTask {
  fn: import { createTask } from "@server/actions.js",
  entities: [Task]
}

action updateTask {
  fn: import { updateTask } from "@server/actions.js",
  entities: [Task]
}
```

## API Integration

### 1. External API Configuration
```wasp
app MyApp {
  dependencies: [
    ("axios", "^1.4.0")
  ],
  env: {
    API_URL: "string",
    API_KEY: "string"
  }
}
```

### 2. Jobs Configuration
```wasp
job emailSender {
  executor: EmailSender,
  perform: {
    fn: import { sendEmail } from "@server/jobs/email.js"
  },
  schedule: {
    cron: "0 * * * *"  // Every hour
  }
}
```

## Client Components

### 1. Page Component
```jsx
// src/client/pages/Dashboard.jsx
import React from 'react'
import { useQuery } from '@wasp/queries'
import { getTasks } from '@wasp/queries'

export function DashboardPage() {
  const { data: tasks, isLoading, error } = useQuery(getTasks)

  if (isLoading) return 'Loading...'
  if (error) return 'Error: ' + error.message

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">Dashboard</h1>
      <div className="mt-4">
        {tasks.map(task => (
          <TaskItem key={task.id} task={task} />
        ))}
      </div>
    </div>
  )
}
```

### 2. Shared Component
```jsx
// src/client/components/TaskItem.jsx
import React from 'react'
import { useAction } from '@wasp/actions'
import { updateTask } from '@wasp/actions'

export function TaskItem({ task }) {
  const updateTaskFn = useAction(updateTask)

  const toggleTask = () => {
    updateTaskFn({
      id: task.id,
      isDone: !task.isDone
    })
  }

  return (
    <div className="flex items-center gap-2 p-2 border rounded">
      <input
        type="checkbox"
        checked={task.isDone}
        onChange={toggleTask}
      />
      <span className={task.isDone ? 'line-through' : ''}>
        {task.description}
      </span>
    </div>
  )
}
```

## Server Operations

### 1. Query Implementation
```javascript
// src/server/queries.js
export const getTasks = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  return context.entities.Task.findMany({
    where: { userId: context.user.id },
    orderBy: { createdAt: 'desc' }
  })
}
```

### 2. Action Implementation
```javascript
// src/server/actions.js
export const createTask = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  return context.entities.Task.create({
    data: {
      description: args.description,
      userId: context.user.id
    }
  })
}
```
