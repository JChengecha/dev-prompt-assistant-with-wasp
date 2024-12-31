# Docker Development Guide

## Project Setup

### 1. Dockerfile
```dockerfile
# Dockerfile
FROM node:16-alpine

# Install Wasp
RUN npm install -g wasp-lang

# Install PostgreSQL client
RUN apk add --no-cache postgresql-client

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy project files
COPY . .

# Expose ports
EXPOSE 3000 3001

# Start development server
CMD ["wasp", "start"]
```

### 2. Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"  # Client
      - "3001:3001"  # Server
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/wasp_db
    depends_on:
      - db

  db:
    image: postgres:14-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=wasp_db
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### 3. Development Environment Variables
```env
# .env.development
NODE_ENV=development
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/wasp_db
JWT_SECRET=your-development-secret
```

## Development Workflow

### 1. Starting the Environment
```bash
# Build and start containers
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down
```

### 2. Database Management
```bash
# Access PostgreSQL CLI
docker-compose exec db psql -U postgres -d wasp_db

# Run migrations
docker-compose exec app wasp db migrate-dev

# Reset database
docker-compose exec app wasp db reset
```

### 3. Development Commands
```bash
# Install new dependency
docker-compose exec app npm install package-name

# Run tests
docker-compose exec app npm test

# Build project
docker-compose exec app npm run build
```

## Environment Configurations

### 1. Development
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    environment:
      - NODE_ENV=development
    volumes:
      - .:/app
      - /app/node_modules
```

### 2. Staging
```yaml
# docker-compose.staging.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.staging
    environment:
      - NODE_ENV=staging
    volumes:
      - /app/node_modules
```

### 3. Production
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.prod
    environment:
      - NODE_ENV=production
```

## Custom Configurations

### 1. Custom Dockerfile
```dockerfile
# Dockerfile.custom
FROM node:16-alpine

# Custom dependencies
RUN apk add --no-cache \
    postgresql-client \
    python3 \
    make \
    g++

# Install global packages
RUN npm install -g \
    wasp-lang \
    nodemon \
    typescript

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies with custom registry
RUN npm install --registry=https://your-registry.com

# Copy project files
COPY . .

# Custom environment setup
ENV CUSTOM_VAR=value

# Custom startup script
CMD ["./scripts/start.sh"]
```

### 2. Custom Docker Compose
```yaml
# docker-compose.custom.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.custom
    ports:
      - "3000:3000"
      - "3001:3001"
    volumes:
      - .:/app
      - /app/node_modules
      - ./custom-config:/app/config
    environment:
      - NODE_ENV=development
      - CUSTOM_VAR=value
    depends_on:
      - db
      - redis

  db:
    image: postgres:14-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=custom_user
      - POSTGRES_PASSWORD=custom_password
      - POSTGRES_DB=custom_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./custom-init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 3. Custom Scripts
```bash
#!/bin/bash
# scripts/start.sh
set -e

# Wait for database
./scripts/wait-for-it.sh db:5432

# Run migrations
wasp db migrate-dev

# Start application
if [ "$NODE_ENV" = "development" ]; then
    nodemon wasp start
else
    wasp start
fi
```

## Development Tools

### 1. VS Code Configuration
```json
// .vscode/settings.json
{
  "remote.containers.defaultExtensions": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-azuretools.vscode-docker"
  ],
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

### 2. Development Scripts
```json
// package.json
{
  "scripts": {
    "docker:dev": "docker-compose -f docker-compose.yml -f docker-compose.dev.yml up",
    "docker:staging": "docker-compose -f docker-compose.yml -f docker-compose.staging.yml up",
    "docker:prod": "docker-compose -f docker-compose.yml -f docker-compose.prod.yml up",
    "docker:custom": "docker-compose -f docker-compose.yml -f docker-compose.custom.yml up",
    "docker:clean": "docker-compose down -v --remove-orphans",
    "docker:rebuild": "docker-compose build --no-cache"
  }
}
```

## Best Practices

### 1. Security
```dockerfile
# Use specific versions
FROM node:16.14.2-alpine3.15

# Run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Scan for vulnerabilities
RUN npm audit
```

### 2. Performance
```dockerfile
# Multi-stage builds
FROM node:16-alpine AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM node:16-alpine
COPY --from=builder /app/dist ./dist
CMD ["npm", "start"]
```

### 3. Development Workflow
```bash
# 1. Start development environment
npm run docker:dev

# 2. Make changes
# Files are synced via volume mounts

# 3. View logs
docker-compose logs -f app

# 4. Run tests
docker-compose exec app npm test

# 5. Clean up
npm run docker:clean
```
