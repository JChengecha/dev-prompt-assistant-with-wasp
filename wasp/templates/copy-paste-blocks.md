# Wasp Copy-Paste Code Blocks

## Basic Project Setup

### 1. Authentication Ready App
```wasp
app MyApp {
  title: "Authentication Ready App",
  dependencies: [
    ("@tailwindcss/forms", "^0.5.3"),
    ("react-hook-form", "^7.45.1")
  ]
}

route LoginRoute { path: "/login", to: LoginPage }
route SignupRoute { path: "/signup", to: SignupPage }
route DashboardRoute { 
  path: "/dashboard", 
  to: DashboardPage,
  authRequired: true
}

entity User {=psl
  id          Int       @id @default(autoincrement())
  email       String    @unique
  password    String
  createdAt   DateTime  @default(now())
psl=}

auth {
  userEntity: User,
  methods: {
    usernameAndPassword: {},
    google: {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET
    }
  },
  onAuthFailedRedirectTo: "/login"
}
```

### 2. Full CRUD Entity
```wasp
entity Task {=psl
  id          Int       @id @default(autoincrement())
  title       String
  description String?
  status      String    @default("todo")
  dueDate     DateTime?
  user        User      @relation(fields: [userId], references: [id])
  userId      Int
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
psl=}

query getTasks {
  fn: import { getTasks } from "@server/queries.js",
  entities: [Task]
}

query getTask {
  fn: import { getTask } from "@server/queries.js",
  entities: [Task]
}

action createTask {
  fn: import { createTask } from "@server/actions.js",
  entities: [Task]
}

action updateTask {
  fn: import { updateTask } from "@server/actions.js",
  entities: [Task]
}

action deleteTask {
  fn: import { deleteTask } from "@server/actions.js",
  entities: [Task]
}
```

### 3. Server Operations (queries.js)
```javascript
// src/server/queries.js
export const getTasks = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  return context.entities.Task.findMany({
    where: { userId: context.user.id },
    orderBy: { createdAt: 'desc' },
    include: {
      user: {
        select: {
          id: true,
          email: true
        }
      }
    }
  })
}

export const getTask = async ({ id }, context) => {
  if (!context.user) throw new HttpError(401)
  
  const task = await context.entities.Task.findUnique({
    where: { id },
    include: {
      user: {
        select: {
          id: true,
          email: true
        }
      }
    }
  })
  
  if (!task) throw new HttpError(404, 'Task not found')
  if (task.userId !== context.user.id) throw new HttpError(403, 'Not authorized')
  
  return task
}
```

### 4. Server Actions (actions.js)
```javascript
// src/server/actions.js
export const createTask = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  const { title, description, dueDate } = args
  if (!title) throw new HttpError(400, 'Title is required')
  
  return context.entities.Task.create({
    data: {
      title,
      description,
      dueDate,
      userId: context.user.id
    }
  })
}

export const updateTask = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  const { id, ...data } = args
  const task = await context.entities.Task.findUnique({ where: { id } })
  
  if (!task) throw new HttpError(404, 'Task not found')
  if (task.userId !== context.user.id) throw new HttpError(403, 'Not authorized')
  
  return context.entities.Task.update({
    where: { id },
    data
  })
}

export const deleteTask = async ({ id }, context) => {
  if (!context.user) throw new HttpError(401)
  
  const task = await context.entities.Task.findUnique({ where: { id } })
  
  if (!task) throw new HttpError(404, 'Task not found')
  if (task.userId !== context.user.id) throw new HttpError(403, 'Not authorized')
  
  return context.entities.Task.delete({ where: { id } })
}
```

### 5. React Components

#### Task List Component
```jsx
// src/client/components/TaskList.jsx
import React from 'react'
import { useQuery } from '@wasp/queries'
import { useAction } from '@wasp/actions'
import { getTasks, deleteTask } from '@wasp/actions/tasks'

export function TaskList() {
  const { data: tasks, isLoading, error } = useQuery(getTasks)
  const deleteTaskFn = useAction(deleteTask)

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <div className="space-y-4">
      {tasks.map(task => (
        <div key={task.id} className="border p-4 rounded-lg shadow">
          <h3 className="text-lg font-semibold">{task.title}</h3>
          {task.description && (
            <p className="text-gray-600">{task.description}</p>
          )}
          <div className="mt-2 flex justify-end space-x-2">
            <button
              onClick={() => deleteTaskFn({ id: task.id })}
              className="px-3 py-1 bg-red-500 text-white rounded hover:bg-red-600"
            >
              Delete
            </button>
          </div>
        </div>
      ))}
    </div>
  )
}
```

#### Task Form Component
```jsx
// src/client/components/TaskForm.jsx
import React from 'react'
import { useForm } from 'react-hook-form'
import { useAction } from '@wasp/actions'
import { createTask } from '@wasp/actions/tasks'

export function TaskForm() {
  const { register, handleSubmit, reset, formState: { errors } } = useForm()
  const createTaskFn = useAction(createTask)

  const onSubmit = async (data) => {
    try {
      await createTaskFn(data)
      reset()
    } catch (error) {
      console.error('Failed to create task:', error)
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label className="block text-sm font-medium text-gray-700">
          Title
        </label>
        <input
          type="text"
          {...register('title', { required: 'Title is required' })}
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
        />
        {errors.title && (
          <p className="mt-1 text-sm text-red-600">{errors.title.message}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700">
          Description
        </label>
        <textarea
          {...register('description')}
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
          rows={3}
        />
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700">
          Due Date
        </label>
        <input
          type="datetime-local"
          {...register('dueDate')}
          className="mt-1 block w-full rounded-md border-gray-300 shadow-sm"
        />
      </div>

      <button
        type="submit"
        className="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700"
      >
        Create Task
      </button>
    </form>
  )
}
```

### 6. Page Components

#### Dashboard Page
```jsx
// src/client/pages/DashboardPage.jsx
import React from 'react'
import { useAuth } from '@wasp/auth'
import { TaskList } from '../components/TaskList'
import { TaskForm } from '../components/TaskForm'

export function DashboardPage() {
  const { data: user } = useAuth()

  return (
    <div className="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
      <div className="px-4 py-6 sm:px-0">
        <h1 className="text-3xl font-bold text-gray-900">
          Welcome back, {user.email}
        </h1>
        
        <div className="mt-8">
          <h2 className="text-xl font-semibold mb-4">Create New Task</h2>
          <TaskForm />
        </div>

        <div className="mt-8">
          <h2 className="text-xl font-semibold mb-4">Your Tasks</h2>
          <TaskList />
        </div>
      </div>
    </div>
  )
}
```

### 7. Environment Setup (.env)
```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# Authentication
JWT_SECRET="your-secret-key"
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"

# API Keys (if needed)
STRIPE_SECRET_KEY="your-stripe-secret-key"
SENDGRID_API_KEY="your-sendgrid-api-key"
```

### 8. Tailwind Configuration (tailwind.config.js)
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          // ... add more shades
          900: '#0c4a6e',
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
```

### 9. Error Handling Middleware
```javascript
// src/server/middleware/errorHandler.js
export const errorHandler = (err, req, res, next) => {
  console.error(err.stack)

  if (err instanceof HttpError) {
    return res.status(err.statusCode).json({
      error: err.message,
      code: err.statusCode
    })
  }

  if (err.code === 'P2002') {
    return res.status(400).json({
      error: 'A record with this value already exists',
      code: 400
    })
  }

  res.status(500).json({
    error: 'Internal server error',
    code: 500
  })
}
```

### 10. API Documentation Template
```markdown
# API Documentation

## Authentication
All endpoints except /auth/* require authentication via JWT token.
Include token in Authorization header: `Bearer <token>`

## Endpoints

### Tasks

#### GET /api/tasks
Returns all tasks for authenticated user.

Response:
```json
{
  "tasks": [
    {
      "id": 1,
      "title": "Task Title",
      "description": "Task Description",
      "status": "todo",
      "dueDate": "2024-01-01T00:00:00.000Z",
      "userId": 1,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

#### POST /api/tasks
Create new task.

Request Body:
```json
{
  "title": "Task Title",
  "description": "Task Description",
  "dueDate": "2024-01-01T00:00:00.000Z"
}
```
```
