# Wasp Development Environment Setup Guide

## Prerequisites

### 1. Node.js Setup
```bash
# Install Node Version Manager (nvm)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Install and use Node.js LTS
nvm install --lts
nvm use --lts
```

### 2. Wasp Installation
```bash
# Install Wasp CLI
curl -sSL https://get.wasp-lang.dev/installer.sh | sh

# Verify installation
wasp version
```

### 3. Database Setup
```bash
# Install PostgreSQL (Ubuntu/Debian)
sudo apt update
sudo apt install postgresql postgresql-contrib

# Start PostgreSQL service
sudo service postgresql start

# Create database
sudo -u postgres createdb my_wasp_db
```

## Project Initialization

### 1. Create New Project
```bash
# Create new Wasp project
wasp new MyApp
cd MyApp

# Initialize Git repository
git init
git add .
git commit -m "Initial commit"
```

### 2. Development Dependencies
```bash
# Install development dependencies
npm install --save-dev
  @testing-library/react
  @testing-library/jest-dom
  prettier
  eslint
  typescript
```

### 3. Environment Configuration
```bash
# Create development environment file
cat > .env.development << EOL
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/my_wasp_db"
JWT_SECRET="your-development-secret"
GOOGLE_CLIENT_ID="your-client-id"
GOOGLE_CLIENT_SECRET="your-client-secret"
EOL
```

## IDE Setup

### 1. VS Code Configuration
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "[wasp]": {
    "editor.defaultFormatter": "wasp-lang.wasp"
  }
}
```

### 2. VS Code Extensions
- Wasp Language Support
- ESLint
- Prettier
- GitLens
- Prisma

### 3. Git Configuration
```bash
# Create .gitignore
cat > .gitignore << EOL
.wasp/
node_modules/
.env*
*.log
dist/
build/
.DS_Store
EOL
```

## Development Workflow

### 1. Start Development Server
```bash
# Start Wasp development server
wasp start

# In another terminal, start client development
cd .wasp/out/web-app
npm start
```

### 2. Database Management
```bash
# Generate Prisma client
wasp db migrate-dev

# Reset database
wasp db reset

# View database studio
wasp db studio
```

### 3. Testing Setup
```bash
# Run tests
wasp test

# Run specific test
wasp test src/client/components/TaskList.test.jsx
```

## Troubleshooting

### 1. Common Issues
```bash
# Clear Wasp cache
rm -rf .wasp/

# Reinstall dependencies
npm clean-install

# Reset database
wasp db reset
```

### 2. Logs
```bash
# View server logs
tail -f .wasp/server.log

# View client logs
tail -f .wasp/client.log
```

## Production Preparation

### 1. Build Configuration
```bash
# Create production environment file
cat > .env.production << EOL
DATABASE_URL="your-production-db-url"
JWT_SECRET="your-production-secret"
GOOGLE_CLIENT_ID="your-production-client-id"
GOOGLE_CLIENT_SECRET="your-production-client-secret"
EOL
```

### 2. Build Process
```bash
# Build for production
wasp build

# Test production build locally
cd .wasp/build
npm start
```

## Monitoring Setup

### 1. Error Tracking
```javascript
// src/server/middleware/errorTracking.js
import * as Sentry from "@sentry/node"

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV
})
```

### 2. Performance Monitoring
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
