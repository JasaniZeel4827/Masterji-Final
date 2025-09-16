# 🚀 Deployment Guide

## Overview

This guide provides comprehensive instructions for deploying Masterji to various environments, including production, staging, and development. It covers infrastructure setup, environment configuration, and deployment procedures.

## 🏗️ Infrastructure Requirements

### Production Environment

1. **Compute**
   - VPS: 2+ vCPUs, 4GB+ RAM (e.g., AWS EC2 t3.medium, DigitalOcean Droplet)
   - Container Orchestration: Docker Swarm or Kubernetes (for production)

2. **Database**
   - PostgreSQL 14+
   - Recommended: Managed database service (e.g., AWS RDS, DigitalOcean Managed Databases)
   - Storage: 25GB+ with automated backups

3. **Object Storage**
   - S3-compatible storage (AWS S3, DigitalOcean Spaces, MinIO)
   - 100GB+ storage with versioning enabled

4. **CDN**
   - Cloudflare or similar CDN for static assets
   - Global edge caching

## 🛠️ Prerequisites

1. **Accounts**
   - GitHub (for repository access)
   - Vercel/Netlify (for frontend hosting)
   - AWS/DigitalOcean (for infrastructure)
   - Stripe (for payments)
   - Resend (for emails)

2. **Local Development**
   - Node.js 18+
   - pnpm 8+
   - Docker & Docker Compose
   - Git

## 📦 Environment Setup

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/masterji.git
cd masterji
```

### 2. Install Dependencies

```bash
pnpm install
```

### 3. Environment Variables

Create a `.env` file based on the example:

```bash
cp .env.example .env
```

Update the environment variables with your configuration:

```env
# App
NEXTAUTH_SECRET=your-secret-key-here
NEXTAUTH_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/masterji

# AWS S3
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1
AWS_S3_BUCKET_NAME=masterji-uploads

# OAuth Providers
GITHUB_ID=your-github-app-id
GITHUB_SECRET=your-github-app-secret

# Email
EMAIL_SERVER=smtp://username:password@smtp.example.com:587
EMAIL_FROM=noreply@example.com

# Stripe
STRIPE_SECRET_KEY=your-stripe-secret-key
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your-stripe-publishable-key

# Arcjet
ARCJET_KEY=your-arcjet-key
```

## 🐳 Docker Deployment

### 1. Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/masterji
      - NEXTAUTH_URL=http://localhost:3000
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      # Add other environment variables
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=masterji
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

### 2. Build and Start

```bash
docker-compose up -d --build
```

### 3. Run Database Migrations

```bash
docker-compose exec app npx prisma migrate deploy
```

## ☁️ Vercel Deployment

### 1. Connect Repository

1. Import your GitHub repository to Vercel
2. Configure environment variables in the Vercel dashboard
3. Set the root directory to the project root

### 2. Build Settings

- **Framework Preset**: Next.js
- **Build Command**: `pnpm build`
- **Output Directory**: `.next`
- **Install Command**: `pnpm install`

### 3. Environment Variables

Add all required environment variables in the Vercel dashboard under Project Settings > Environment Variables.

## 🚀 Production Deployment

### 1. Database Setup

1. Create a production PostgreSQL database
2. Set up automated backups
3. Configure connection pooling

### 2. File Storage

1. Create an S3 bucket
2. Enable CORS configuration:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
   <CORSRule>
       <AllowedOrigin>https://yourdomain.com</AllowedOrigin>
       <AllowedMethod>GET</AllowedMethod>
       <AllowedMethod>PUT</AllowedMethod>
       <AllowedMethod>POST</AllowedMethod>
       <AllowedMethod>DELETE</AllowedMethod>
       <AllowedHeader>*</AllowedHeader>
   </CORSRule>
   </CORSConfiguration>
   ```
3. Set up lifecycle policies for old files

### 3. SSL/TLS

1. Set up SSL certificates (Let's Encrypt)
2. Enable HTTP/2
3. Configure HSTS

### 4. CDN Configuration

1. Set up CDN for static assets
2. Configure custom domain
3. Enable HTTP/2 and Brotli compression

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm run lint
      - run: pnpm run test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      - run: pnpm install
      - run: pnpm run build
      - name: Install Vercel CLI
        run: npm install --global vercel@latest
      - name: Deploy to Vercel
        run: vercel --prod --confirm
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

## 📊 Monitoring and Logging

### 1. Application Monitoring

- **Error Tracking**: Sentry or LogRocket
- **Performance Monitoring**: Vercel Analytics or New Relic
- **Uptime Monitoring**: UptimeRobot or Better Stack

### 2. Logging

- **Application Logs**: Winston or Pino
- **Access Logs**: Nginx or Traefik
- **Centralized Logging**: ELK Stack or Papertrail

### 3. Alerting

- **Error Alerts**: Sentry or Rollbar
- **Performance Alerts**: New Relic or Datadog
- **Uptime Alerts**: UptimeRobot or Better Stack

## 🔄 Backup and Recovery

### 1. Database Backups

```bash
# Daily backup script
pg_dump -U postgres -d masterji -f /backups/masterji-$(date +%Y%m%d).sql
```

### 2. File Storage Backups

1. Enable versioning on S3 bucket
2. Set up cross-region replication
3. Regular backup validation

### 3. Disaster Recovery Plan

1. Document recovery procedures
2. Regular backup testing
3. Failover procedures

## 🔒 Security Hardening

### 1. Server Hardening

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Configure firewall
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable

# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 2. Database Security

1. Enable SSL for database connections
2. Use strong passwords
3. Regular security updates

### 3. Application Security

1. Regular dependency updates
2. Security headers
3. Rate limiting

## 📈 Scaling

### 1. Horizontal Scaling

- **Stateless Application**: Scale application instances
- **Database**: Read replicas, connection pooling
- **Caching**: Redis or Memcached

### 2. Vertical Scaling

- **Database**: Larger instance types
- **Application**: More CPU/RAM
- **Storage**: Larger volumes

## 🔄 Zero-Downtime Deployments

### 1. Blue-Green Deployment

1. Deploy new version to green environment
2. Test new version
3. Switch traffic to green
4. Keep blue as rollback option

### 2. Canary Releases

1. Deploy to a subset of users
2. Monitor metrics
3. Gradually increase traffic

## 🔗 Related Documentation

- [Next.js Deployment](https://nextjs.org/docs/deployment)
- [Docker Documentation](https://docs.docker.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
