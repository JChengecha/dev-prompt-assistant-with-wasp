# Wasp Project Initialization Guide

## Prerequisites
- Wasp CLI (version 0.11.0+)
- Node.js (16.x or later)
- npm or yarn
- Code editor with Wasp/React support

## Installation Steps

### 1. Install Wasp CLI
```bash
# macOS/Linux
curl -sSL https://get.wasp-lang.dev | sh

# Windows (PowerShell)
iwr https://get.wasp-lang.dev -useb | iex
```

### 2. Verify Installation
```bash
wasp --version
```

## Project Creation Workflow

### Initialize New Project
```bash
# Create new Wasp project
wasp start my-project

# Navigate to project directory
cd my-project
```

### Project Structure
```
my-project/
├── .wasp/               # Wasp configuration
├── src/
│   ├── client/          # React frontend
│   │   ├── components/
│   │   └── pages/
│   ├── server/          # Node.js backend
│   │   ├── actions/
│   │   └── queries/
├── migrations/          # Database migrations
├── prisma/              # Database schema
├── .gitignore
├── wasp.config.js
└── package.json
```

## Configuration Best Practices

### Wasp Configuration
```wasp
app MyProject {
  title: "My Awesome App",
  auth: {
    methods: {
      email: {},
      google: {}
    },
    userEntity: User
  }
}

route MainRoute { 
  path: "/", 
  component: MainPage 
}
```

### Environment Setup
1. Create `.env` file
2. Add configuration variables
```
DATABASE_URL=postgresql://username:password@localhost:5432/myproject
GOOGLE_CLIENT_ID=your_client_id
GOOGLE_CLIENT_SECRET=your_client_secret
```

## Initial Development Workflow
```bash
# Start development server
wasp dev

# Build for production
wasp build

# Deploy
wasp deploy
```

## Troubleshooting Common Issues

### 1. Installation and Setup Problems

#### Wasp CLI Installation Failures
- **Symptom**: Unable to install Wasp CLI
- **Potential Solutions**:
  ```bash
  # Verify Node.js version
  node --version

  # Ensure npm is up to date
  npm install -g npm@latest

  # Try alternative installation methods
  curl -sSL https://get.wasp-lang.dev | sh
  ```

#### Dependency Conflicts
- **Symptom**: Package version incompatibilities
- **Diagnostic Steps**:
  ```bash
  # Clear npm cache
  npm cache clean --force

  # Remove node_modules and reinstall
  rm -rf node_modules
  npm install

  # Use exact versions specified in package.json
  npm ci
  ```

### 2. Runtime Errors

#### Common Wasp Compilation Errors
- **Symptom**: Wasp fails to compile project
- **Troubleshooting**:
  - Check syntax in `.wasp` files
  - Verify import statements
  - Ensure all referenced files exist

#### Authentication Issues
- **Common Problems**:
  - Social login not working
  - User authentication failures
- **Diagnostic Checklist**:
  1. Verify environment variables
  2. Check OAuth provider configurations
  3. Validate callback URLs

### 3. Performance and Deployment

#### Slow Application Performance
- **Diagnostic Commands**:
  ```bash
  # Check Wasp server logs
  wasp start --verbose

  # Profile application
  npm run profile

  # Check bundle size
  npm run build --report
  ```

#### Deployment Configuration Errors
- **Troubleshooting Steps**:
  1. Validate `.env` file configurations
  2. Check database connection strings
  3. Verify hosting platform requirements

### 4. Development Environment Setup

#### VSCode Wasp Extension Issues
- **Recommended Setup**:
  ```bash
  # Install Wasp VSCode Extension
  code --install-extension wasp-lang.wasp

  # Reload VSCode
  code --reload
  ```

### 5. Advanced Debugging

#### Logging and Monitoring
- Use `console.log()` in server actions
- Implement error tracking
- Use Wasp's built-in logging mechanisms

### Recommended Debugging Tools
- React DevTools
- Chrome/Firefox Developer Tools
- Prisma Studio for database inspection

## Emergency Recovery

### Reset Project
```bash
# Complete project reset
wasp clean
wasp start

# Regenerate configurations
wasp db reset
```

## Community Support
- [Wasp GitHub Discussions](https://github.com/wasp-lang/wasp/discussions)
- Official Wasp Discord Channel
- Stack Overflow [wasp-lang] tag

## Recommended Extensions
- Wasp VSCode Extension
- Prisma VSCode Extension
- ESLint
- Prettier

## Best Practices
- Use declarative configuration
- Leverage Wasp's built-in generators
- Modularize your code
- Follow React and Node.js best practices

## Version Compatibility
- Wasp: 0.11.x
- React: 18.x
- Node.js: 16.x+

## Learning Resources
- [Official Wasp Documentation](https://wasp-lang.dev/docs)
- [Wasp GitHub Repository](https://github.com/wasp-lang/wasp)
- Community forums and Discord channel
