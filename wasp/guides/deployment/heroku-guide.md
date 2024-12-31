# Deploying Wasp to Heroku

## Prerequisites
- Heroku CLI installed
- Git repository initialized
- Wasp application ready for deployment

## 1. Project Setup

### Procfile
```
# Procfile
web: npm start
release: npx prisma migrate deploy
```

### package.json Configuration
```json
{
  "scripts": {
    "start": "wasp start",
    "build": "wasp build",
    "postinstall": "wasp build"
  },
  "engines": {
    "node": "16.x"
  }
}
```

### Environment Variables
```bash
# Set required environment variables
heroku config:set NODE_ENV=production
heroku config:set DATABASE_URL=your-database-url
heroku config:set JWT_SECRET=your-jwt-secret
```

## 2. Database Setup

### Provision PostgreSQL
```bash
# Add PostgreSQL addon
heroku addons:create heroku-postgresql:hobby-dev

# Get database URL
heroku config:get DATABASE_URL
```

### Database Migrations
```bash
# Run migrations
heroku run npx prisma migrate deploy
```

## 3. Deployment Steps

### Initial Deployment
```bash
# Login to Heroku
heroku login

# Create Heroku app
heroku create your-app-name

# Add Heroku remote
heroku git:remote -a your-app-name

# Deploy
git push heroku main
```

### Continuous Deployment
```yaml
# .github/workflows/heroku-deploy.yml
name: Heroku Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: akhileshns/heroku-deploy@v3.12.12
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: "your-app-name"
          heroku_email: "your-email@example.com"
```

## 4. Scaling Configuration

### Basic Scaling
```bash
# Scale web dynos
heroku ps:scale web=2

# Enable auto-scaling
heroku ps:autoscale:enable web
```

### Resource Configuration
```bash
# Set dyno type
heroku ps:type web=standard-2x

# Configure memory quota
heroku config:set WEB_MEMORY=512
```

## 5. Monitoring

### Logging
```bash
# View logs
heroku logs --tail

# Enable papertrail
heroku addons:create papertrail
```

### Performance Monitoring
```bash
# Add New Relic
heroku addons:create newrelic

# Configure New Relic
heroku config:set NEW_RELIC_LICENSE_KEY=your-license-key
heroku config:set NEW_RELIC_APP_NAME="Your App Name"
```

## 6. Common Issues

### Database Connection
```bash
# Check database status
heroku pg:info

# Reset database
heroku pg:reset DATABASE_URL
heroku run npx prisma migrate deploy
```

### Memory Issues
```bash
# Check memory usage
heroku ps:mem

# Optimize memory settings
heroku config:set NODE_OPTIONS="--max_old_space_size=2560"
```

## 7. Best Practices

### Security
```bash
# Enable SSL
heroku certs:auto:enable

# Force HTTPS
heroku config:set FORCE_HTTPS=true
```

### Performance
```bash
# Enable gzip compression
heroku config:set ENABLE_COMPRESSION=true

# Configure caching
heroku config:set CACHE_CONTROL="public, max-age=31536000"
```

## 8. Maintenance

### Database Backups
```bash
# Schedule backups
heroku pg:backups:schedule DATABASE_URL --at "04:00 UTC"

# Manual backup
heroku pg:backups:capture
```

### Updates
```bash
# Update dependencies
heroku run npm update

# Update Heroku stack
heroku stack:set heroku-22
```
