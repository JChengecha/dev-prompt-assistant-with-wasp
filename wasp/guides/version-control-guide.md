# Version Control Strategy Guide

## Branching Strategy

### Main Branches
```
main           Production branch
├── develop    Development branch
├── staging    Staging branch
└── feature/*  Feature branches
```

### Branch Naming Convention
```
feature/   Feature development
fix/       Bug fixes
docs/      Documentation
refactor/  Code refactoring
test/      Testing
```

Example:
```bash
# Feature branch
git checkout -b feature/user-authentication

# Bug fix branch
git checkout -b fix/login-validation

# Documentation branch
git checkout -b docs/api-documentation
```

## Commit Message Conventions

### Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- feat: New feature
- fix: Bug fix
- docs: Documentation
- style: Formatting
- refactor: Code restructuring
- test: Testing
- chore: Maintenance

### Examples
```bash
# Feature commit
git commit -m "feat(auth): implement Google OAuth login

- Add Google OAuth provider
- Create login button component
- Handle OAuth callback

Closes #123"

# Bug fix commit
git commit -m "fix(api): correct user validation logic

- Add email format validation
- Handle empty password case
- Add error messages

Fixes #456"

# Documentation commit
git commit -m "docs(readme): update installation guide

- Add Docker setup instructions
- Update environment variables section
- Add troubleshooting tips"
```

## Release Process

### 1. Version Bumping
```bash
# Update version in package.json
npm version patch  # For bug fixes
npm version minor  # For new features
npm version major  # For breaking changes
```

### 2. Release Branch
```bash
# Create release branch
git checkout develop
git checkout -b release/1.2.0

# Update version
npm version 1.2.0

# Commit changes
git commit -m "chore(release): bump version to 1.2.0"
```

### 3. Release Notes Template
```markdown
# Release 1.2.0

## New Features
- Feature 1: Description
- Feature 2: Description

## Bug Fixes
- Fix 1: Description
- Fix 2: Description

## Breaking Changes
- Change 1: Description
- Change 2: Description

## Dependencies
- Updated package1 to v2.0.0
- Added package2 v1.0.0

## Migration Guide
Steps to migrate from 1.1.x to 1.2.0...
```

### 4. Release Workflow
```bash
# 1. Merge develop into release branch
git checkout release/1.2.0
git merge develop

# 2. Run tests
npm test

# 3. Build project
npm run build

# 4. Create release tag
git tag -a v1.2.0 -m "Release 1.2.0"

# 5. Merge into main
git checkout main
git merge release/1.2.0

# 6. Push changes
git push origin main
git push origin v1.2.0

# 7. Clean up
git branch -d release/1.2.0
```

## Git Hooks

### pre-commit
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run tests
npm test

# Run linter
npm run lint

# Run type checking
npm run type-check
```

### pre-push
```bash
#!/bin/sh
# .git/hooks/pre-push

# Run full test suite
npm run test:full

# Build project
npm run build
```

## CI/CD Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm ci
      - run: npm test
      - run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v2
      - name: Deploy
        run: |
          # Deploy steps
```

## Version Control Best Practices

### 1. Branch Management
- Keep branches short-lived
- Delete merged branches
- Regularly sync with base branch

### 2. Commit Guidelines
- Write clear commit messages
- Keep commits atomic
- Reference issues in commits

### 3. Pull Request Process
- Use PR template
- Request code reviews
- Update documentation

### 4. Code Review Checklist
```markdown
## Code Review Checklist

### Functionality
- [ ] Code works as expected
- [ ] Edge cases handled
- [ ] Error handling implemented

### Code Quality
- [ ] Follows style guide
- [ ] No duplicate code
- [ ] Clear naming

### Testing
- [ ] Tests added/updated
- [ ] Tests pass
- [ ] Coverage maintained

### Documentation
- [ ] Code documented
- [ ] API docs updated
- [ ] README updated
```

## Git Configuration

### Global Settings
```bash
# User configuration
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Default branch
git config --global init.defaultBranch main

# Auto-formatting
git config --global core.autocrlf input
```

### Project Settings
```bash
# .gitignore
node_modules/
.env
.wasp/
dist/
*.log
.DS_Store

# .gitattributes
*.js text eol=lf
*.jsx text eol=lf
*.ts text eol=lf
*.tsx text eol=lf
*.md text eol=lf
*.json text eol=lf
```
