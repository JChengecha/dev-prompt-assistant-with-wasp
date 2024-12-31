# Wasp Project Structure Guide

## Directory Structure

```plaintext
my-wasp-app/
├── main.wasp                 # Main Wasp configuration file
├── src/
│   ├── client/              # Client-side code
│   │   ├── App.css          # Global styles
│   │   ├── App.jsx          # Root component
│   │   ├── components/      # Reusable components
│   │   │   ├── common/      # Shared UI components
│   │   │   ├── forms/       # Form components
│   │   │   └── layout/      # Layout components
│   │   ├── pages/          # Page components
│   │   ├── utils/          # Client utilities
│   │   └── hooks/          # Custom React hooks
│   └── server/             # Server-side code
│       ├── actions/        # Wasp actions
│       ├── queries/        # Wasp queries
│       ├── jobs/           # Background jobs
│       ├── middleware/     # Custom middleware
│       └── utils/          # Server utilities
├── migrations/             # Database migrations
├── public/                # Static assets
├── tests/                 # Test files
│   ├── unit/             # Unit tests
│   ├── integration/      # Integration tests
│   └── e2e/              # End-to-end tests
├── .env                  # Environment variables
├── .gitignore           # Git ignore file
└── package.json         # Node.js dependencies
```

## File Templates

### 1. Main Wasp Configuration (main.wasp)
```wasp
app MyApp {
  title: "My Wasp Application",
  
  dependencies: [
    ("react", "^18.2.0"),
    ("react-dom", "^18.2.0")
  ],

  auth: {
    userEntity: User,
    methods: {
      usernameAndPassword: {}
    },
    onAuthFailedRedirectTo: "/login"
  },

  db: {
    system: PostgreSQL
  }
}

// Entities
entity User {=psl
  id          Int       @id @default(autoincrement())
  email       String    @unique
  password    String
  tasks       Task[]
psl=}

// Routes
route LoginRoute { path: "/login", to: LoginPage }
route DashboardRoute {
  path: "/dashboard",
  to: DashboardPage,
  authRequired: true
}

// Operations
query getTasks {
  fn: import { getTasks } from "@server/queries.js",
  entities: [Task]
}

action createTask {
  fn: import { createTask } from "@server/actions.js",
  entities: [Task]
}
```

### 2. React Components

#### Page Component (src/client/pages/Dashboard.jsx)
```jsx
import React from 'react'
import { useQuery } from '@wasp/queries'
import { getTasks } from '@wasp/queries'
import { TaskList } from '../components/TaskList'
import { Layout } from '../components/layout/Layout'

export function DashboardPage() {
  const { data: tasks, isLoading, error } = useQuery(getTasks)

  if (isLoading) return <Layout>Loading...</Layout>
  if (error) return <Layout>Error: {error.message}</Layout>

  return (
    <Layout>
      <h1>Dashboard</h1>
      <TaskList tasks={tasks} />
    </Layout>
  )
}
```

#### Layout Component (src/client/components/layout/Layout.jsx)
```jsx
import React from 'react'
import { useAuth } from '@wasp/auth'
import { Navbar } from './Navbar'
import { Footer } from './Footer'

export function Layout({ children }) {
  const { user, isLoading } = useAuth()

  return (
    <div className="min-h-screen bg-gray-100">
      <Navbar user={user} isLoading={isLoading} />
      <main className="container mx-auto px-4 py-8">
        {children}
      </main>
      <Footer />
    </div>
  )
}
```

### 3. Server Operations

#### Query (src/server/queries/getTasks.js)
```javascript
export const getTasks = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  return context.entities.Task.findMany({
    where: { userId: context.user.id },
    include: {
      user: true
    },
    orderBy: { createdAt: 'desc' }
  })
}
```

#### Action (src/server/actions/createTask.js)
```javascript
export const createTask = async (args, context) => {
  if (!context.user) throw new HttpError(401)
  
  const { description } = args
  if (!description) throw new HttpError(400, 'Description is required')
  
  return context.entities.Task.create({
    data: {
      description,
      userId: context.user.id
    }
  })
}
```

### 4. Tests

#### Unit Test (tests/unit/components/TaskList.test.jsx)
```javascript
import { render, screen } from '@testing-library/react'
import { TaskList } from '@client/components/TaskList'

describe('TaskList', () => {
  it('renders tasks correctly', () => {
    const tasks = [
      { id: 1, description: 'Test task', isDone: false }
    ]
    
    render(<TaskList tasks={tasks} />)
    expect(screen.getByText('Test task')).toBeInTheDocument()
  })
})
```

#### Integration Test (tests/integration/tasks.test.js)
```javascript
import { createTask, getTasks } from '@server/actions'
import { prisma } from '@wasp/dbClient'

describe('Task Operations', () => {
  beforeEach(async () => {
    await prisma.task.deleteMany({})
  })

  it('creates and retrieves tasks', async () => {
    const user = await createTestUser()
    const context = { user, entities: prisma }
    
    await createTask({ description: 'Test task' }, context)
    const tasks = await getTasks({}, context)
    
    expect(tasks).toHaveLength(1)
    expect(tasks[0].description).toBe('Test task')
  })
})
```

### 5. Configuration Files

#### Environment Variables (.env)
```env
# Database
DATABASE_URL="postgresql://user:pass@localhost:5432/dbname"

# Authentication
JWT_SECRET="your-secret-key"
GOOGLE_CLIENT_ID="your-client-id"
GOOGLE_CLIENT_SECRET="your-client-secret"

# API Keys
STRIPE_SECRET_KEY="your-stripe-key"
SENDGRID_API_KEY="your-sendgrid-key"

# Feature Flags
ENABLE_PREMIUM_FEATURES=false
```

#### Package.json
```json
{
  "name": "my-wasp-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@tailwindcss/forms": "^0.5.3",
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/react": "^13.4.0",
    "axios": "^1.4.0",
    "date-fns": "^2.30.0",
    "react-query": "^3.39.3"
  },
  "devDependencies": {
    "autoprefixer": "^10.4.14",
    "postcss": "^8.4.24",
    "tailwindcss": "^3.3.2",
    "typescript": "^5.1.3"
  }
}
```

## Best Practices

1. **File Organization**
   - Group related components together
   - Use feature-based organization for larger apps
   - Keep components small and focused

2. **Naming Conventions**
   - Use PascalCase for components
   - Use camelCase for functions and variables
   - Use kebab-case for file names

3. **Code Structure**
   - Separate business logic from UI
   - Use custom hooks for shared logic
   - Keep components pure when possible

4. **Testing**
   - Write tests alongside components
   - Focus on user interactions
   - Use meaningful test descriptions

5. **State Management**
   - Use React Query for server state
   - Use React Context for app state
   - Keep state close to where it's used
