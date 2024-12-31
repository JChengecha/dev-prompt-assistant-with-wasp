# Stage Transition Templates

## Initialization ➡️ Configuration

### Checklist
```markdown
- [ ] Project directory structure verified
- [ ] Node.js and Wasp CLI installed
- [ ] Git repository initialized
- [ ] Dependencies installed
- [ ] Environment variables configured
```

### Verification
```bash
# Verify Wasp installation
wasp --version

# Check project structure
ls -la

# Verify dependencies
npm list

# Test environment
wasp start
```

## Configuration ➡️ Feature Development

### Checklist
```markdown
- [ ] .wasp file configured
- [ ] Database schema defined
- [ ] Authentication setup complete
- [ ] Routes configured
- [ ] Basic pages created
```

### Verification
```bash
# Verify database
wasp db migrate-dev

# Check auth setup
wasp start
# Test login/signup flows

# Verify routes
curl http://localhost:3000
```

## Feature Development ➡️ Testing

### Checklist
```markdown
- [ ] All planned features implemented
- [ ] API endpoints documented
- [ ] Components tested manually
- [ ] Error handling implemented
- [ ] Loading states added
```

### Verification
```bash
# Run tests
npm test

# Check coverage
npm run test:coverage

# Lint code
npm run lint
```

## Testing ➡️ Deployment

### Checklist
```markdown
- [ ] All tests passing
- [ ] Performance benchmarks met
- [ ] Security audit passed
- [ ] Documentation updated
- [ ] Production config ready
```

### Verification
```bash
# Full test suite
npm run test:all

# Build check
wasp build

# Security audit
npm audit
```

## Deployment ➡️ Post-MVP

### Checklist
```markdown
- [ ] Production deployment successful
- [ ] Monitoring configured
- [ ] Backup system verified
- [ ] Error tracking active
- [ ] Analytics implemented
```

### Verification
```bash
# Check deployment
curl https://your-app.com

# Verify monitoring
# Check dashboard

# Test backup
# Run backup verification
```
