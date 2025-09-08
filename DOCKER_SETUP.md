# Docker Setup Guide for Acquisitions API

This guide explains how to run the Acquisitions API using Docker with different configurations for development and production environments.

## 🏗️ Architecture Overview

### Development Environment

- **Neon Local**: Runs a local proxy that creates ephemeral database branches
- **Application**: Connects to Neon Local proxy instead of cloud database
- **Benefits**: Fresh database for each session, no impact on production data

### Production Environment

- **Neon Cloud Database**: Direct connection to your production Neon database
- **Application**: Optimized production build with resource limits
- **Benefits**: Production-ready deployment with proper security and performance

## 📋 Prerequisites

1. **Docker & Docker Compose** installed on your system
2. **Neon Account** with a project created at [console.neon.tech](https://console.neon.tech)
3. **Neon API Key** (get from Neon Console → Account Settings → API Keys)

## 🔧 Initial Setup

### 1. Get Neon Credentials

From your [Neon Console](https://console.neon.tech):

- **NEON_API_KEY**: Go to Account Settings → API Keys
- **NEON_PROJECT_ID**: Found in Project Settings → General
- **DATABASE_URL**: Copy from your dashboard (for production)

### 2. Configure Environment Files

#### Development Configuration

Edit `.env.development`:

```bash
# Required for Neon Local
NEON_API_KEY=neon_api_1ABCDEFGHijklmnop1234567890
NEON_PROJECT_ID=steep-forest-12345678
PARENT_BRANCH_ID=main

# Application settings
JWT_SECRET=your-development-jwt-secret-key-here
PORT=3000
LOG_LEVEL=debug
```

#### Production Configuration

Edit `.env.production`:

```bash
# Direct Neon Cloud connection
DATABASE_URL=postgres://username:password@ep-cool-darkness-123456.us-east-2.aws.neon.tech/dbname?sslmode=require

# Application settings
JWT_SECRET=your-strong-production-jwt-secret-key-here
PORT=3000
LOG_LEVEL=info
CORS_ORIGIN=https://yourdomain.com
```

## 🚀 Development Workflow

### Starting Development Environment

```bash
# Start with Neon Local (creates fresh ephemeral database)
npm run docker:dev

# Or manually
docker-compose -f docker-compose.dev.yml --env-file .env.development up --build
```

This will:

1. Start Neon Local proxy on port 5432
2. Create an ephemeral database branch from your main branch
3. Start your application with hot-reload on port 3000
4. Mount your source code for live development

### Development Commands

```bash
# View logs
npm run docker:logs:dev

# Stop and remove containers + volumes
npm run docker:dev:down

# Rebuild containers
npm run docker:build:dev

# Run database migrations (inside running container)
docker exec acquisitions-app-dev npm run db:migrate

# Open Drizzle Studio (inside running container)
docker exec acquisitions-app-dev npm run db:studio
```

### Development Features

- **Hot Reload**: Code changes automatically restart the server
- **Fresh Database**: Each `docker:dev` creates a new database branch
- **Volume Mounts**: Source code and logs are mounted for easy access
- **Debug Logging**: Verbose logging for development

## 🏭 Production Deployment

### Starting Production Environment

```bash
# Start in production mode (detached)
npm run docker:prod

# Or manually
docker-compose -f docker-compose.prod.yml --env-file .env.production up --build -d
```

This will:

1. Build optimized production image
2. Connect directly to your Neon Cloud database
3. Run with resource limits and health checks
4. Start in detached mode

### Production Commands

```bash
# View logs
npm run docker:logs:prod

# Stop production containers
npm run docker:prod:down

# Rebuild production image
npm run docker:build:prod

# Scale the application (if needed)
docker-compose -f docker-compose.prod.yml up --scale app=3 -d
```

### Production Features

- **Optimized Build**: Multi-stage Docker build for smaller image size
- **Resource Limits**: CPU and memory constraints
- **Health Checks**: Built-in application health monitoring
- **Security**: Non-root user, minimal attack surface

## 🔍 Database Management

### Running Migrations

#### Development

```bash
# Inside the running dev container
docker exec acquisitions-app-dev npm run db:migrate

# Or connect to Neon Local directly
docker exec acquisitions-neon-local psql -U neon -d neondb
```

#### Production

```bash
# Inside the running prod container
docker exec acquisitions-app-prod npm run db:migrate

# Or run one-time migration container
docker run --rm -it --env-file .env.production acquisitions-app npm run db:migrate
```

### Database Studio

```bash
# Development (with Neon Local)
docker exec acquisitions-app-dev npm run db:studio
# Visit http://localhost:4983

# Production (connects to cloud)
docker exec acquisitions-app-prod npm run db:studio
```

## 🌐 Network Configuration

### Development

- **Application**: http://localhost:3000
- **Neon Local**: localhost:5432
- **Drizzle Studio**: http://localhost:4983

### Production

- **Application**: http://localhost:3000 (configure reverse proxy)
- **Database**: Direct connection to Neon Cloud

## 🔒 Security Considerations

### Development

- Ephemeral databases automatically deleted
- Debug logging may expose sensitive information
- Use only for development

### Production

- Strong JWT secrets
- CORS properly configured
- Resource limits enforced
- Health checks for reliability

## 🐛 Troubleshooting

### Common Issues

#### "Cannot connect to Neon Local"

```bash
# Check if Neon Local is healthy
docker-compose -f docker-compose.dev.yml ps

# Check Neon Local logs
docker logs acquisitions-neon-local

# Verify environment variables
docker-compose -f docker-compose.dev.yml config
```

#### "Database migration failed"

```bash
# Check database connection
docker exec acquisitions-app-dev npm run db:studio

# Manual migration
docker exec -it acquisitions-app-dev npm run db:generate
docker exec -it acquisitions-app-dev npm run db:migrate
```

#### "Port already in use"

```bash
# Stop all containers
docker-compose -f docker-compose.dev.yml down
docker-compose -f docker-compose.prod.yml down

# Check what's using the port
lsof -i :3000
lsof -i :5432
```

### Cleanup Commands

```bash
# Remove all containers and volumes
docker-compose -f docker-compose.dev.yml down -v
docker-compose -f docker-compose.prod.yml down -v

# Remove Docker images
docker rmi acquisitions-app

# Clean up Docker system
docker system prune -a
```

## 📁 File Structure

```
acquisitions/
├── Dockerfile                 # Multi-stage Docker build
├── docker-compose.dev.yml     # Development with Neon Local
├── docker-compose.prod.yml    # Production with Neon Cloud
├── .dockerignore              # Files excluded from build
├── .env.development           # Development environment vars
├── .env.production            # Production environment vars
├── .neon_local/               # Neon Local metadata (git ignored)
└── logs/                      # Application logs (mounted)
```

## 🎯 Best Practices

1. **Always use environment-specific files**
2. **Never commit real credentials to git**
3. **Use ephemeral branches for development testing**
4. **Monitor resource usage in production**
5. **Regularly update Docker images**
6. **Use Docker health checks**
7. **Implement proper logging and monitoring**

## 🚀 Quick Start Checklist

- [ ] Install Docker and Docker Compose
- [ ] Create Neon account and get API credentials
- [ ] Update `.env.development` with your Neon credentials
- [ ] Update `.env.production` with your production database URL
- [ ] Run `npm run docker:dev` to start development
- [ ] Visit http://localhost:3000 to verify the application
- [ ] Run database migrations if needed
- [ ] For production, use `npm run docker:prod`

---

For additional help, check the [Neon Local documentation](https://neon.com/docs/local/neon-local) or create an issue in the project repository.
