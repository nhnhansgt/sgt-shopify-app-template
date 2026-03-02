# Design: Migrate from SQLite to MySQL

**Date:** 2026-03-02
**Status:** Approved
**Author:** Claude Code + User Collaboration

## Overview

Migrate the Shopify app's session storage from SQLite to MySQL for production-ready database infrastructure.

## Requirements

- **Goal:** Production-ready database for Shopify app
- **Deployment:** Managed cloud database (AWS RDS, Google Cloud SQL, Azure)
- **Data Migration:** Start with fresh database (no existing data migration needed)
- **Environment:** MySQL for all environments (local, dev, staging, production)

## Architecture

### Current State
```
App → Prisma Client → SQLite (file:dev.sqlite)
    ↓
Session storage for Shopify authentication
```

### Target State
```
App → Prisma Client → MySQL (cloud database)
                      ↓
                Managed Cloud (AWS RDS / GCP Cloud SQL / Azure)
    ↓
Session storage for Shopify authentication
```

## Approach

**Selected Approach:** Direct MySQL Connection

**Rationale:**
- Simplest implementation with minimal code changes
- Full compatibility with Prisma 6.19.0
- Suitable for Session storage use case (no complex queries)
- Easy to maintain and debug
- Can scale to more advanced solutions later if needed

## Database Schema Changes

### Prisma Schema (`prisma/schema.prisma`)

**Before:**
```prisma
datasource db {
  provider = "sqlite"
  url      = "file:dev.sqlite"
}
```

**After:**
```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```

### Session Model

No changes required. The Session model is fully compatible with MySQL:
- `String` → `VARCHAR(191)`
- `Boolean` → `TINYINT(1)`
- `DateTime` → `DATETIME`
- `BigInt` → `BIGINT`

## Environment Configuration

### Environment Variables

**`.env` (local development):**
```bash
DATABASE_URL="mysql://root:password@localhost:3306/shopify_app_dev"
```

**`.env.production` (production):**
```bash
DATABASE_URL="mysql://user:pass@production-mysql.xxx.region.rds.amazonaws.com:3306/shopify_app_prod"
```

**Alternative: Separate parameters**
```bash
DATABASE_USER="shopify_app_user"
DATABASE_PASSWORD="secure_password_here"
DATABASE_HOST="mysql-cluster.example.com"
DATABASE_PORT=3306
DATABASE_NAME="shopify_app_db"
```

### MySQL Requirements

- MySQL version: 5.7+ or 8.0+ (recommended)
- Database must be created before running migrations
- User privileges: `CREATE`, `ALTER`, `INDEX`, `SELECT`, `INSERT`, `UPDATE`, `DELETE`

## Migration Strategy

### Phase 1: Preparation (Local)
1. Install MySQL locally or use Docker container
2. Create database: `CREATE DATABASE shopify_app_dev;`
3. Test connection with MySQL client

### Phase 2: Configuration Changes
1. Update `prisma/schema.prisma`
2. Create/update `.env` with `DATABASE_URL`

### Phase 3: Database Migration
```bash
npm run setup              # Generate Prisma Client
npx prisma migrate dev --name switch_to_mysql
npx prisma studio          # Verify schema
```

### Phase 4: Production Deployment
1. Create MySQL database on cloud provider
2. Configure `DATABASE_URL` in production environment
3. Run `npm run setup` in production
4. Verify app connects successfully

## Development Workflow

### Local Development Options

**Option A: MySQL Server Local**
```bash
sudo apt install mysql-server
sudo systemctl start mysql
sudo mysql -e "CREATE DATABASE shopify_app_dev;"
```

**Option B: Docker MySQL (Recommended)**
```bash
docker run --name shopify-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=shopify_app_dev \
  -p 3306:3306 \
  -d mysql:8.0
```

### Commands (Unchanged)
```bash
npm run dev                # Start development server
npm run setup              # prisma generate && prisma migrate deploy
npm run prisma studio      # Open Prisma Studio
npm run prisma migrate dev # Create migration
```

## Testing & Verification

### Pre-Deployment Checklist
- [ ] Run `npm run setup` - verify Prisma generates
- [ ] Run `npx prisma studio` - confirm Session table exists
- [ ] Start dev server and test app installation
- [ ] Verify session stored in MySQL
- [ ] Test concurrent connections
- [ ] Verify data persistence after restart

### Production Verification
- [ ] Test `DATABASE_URL` connection
- [ ] Run `npm run setup` before first start
- [ ] Monitor logs for connection errors
- [ ] Verify Shopify app authentication works

## Files to Modify

1. **`prisma/schema.prisma`** - Change datasource provider
2. **`.env`** - Add DATABASE_URL for local development
3. **`.gitignore`** - Ensure .env files are ignored

## Files to Create

1. **`.env.example`** - Template for environment variables

## Dependencies

No new packages needed. Prisma 6.19.0 already supports MySQL.

## Rollback Plan

If issues occur:
1. Revert `schema.prisma` to `provider = "sqlite"`
2. Revert `.env` to use `file:dev.sqlite`
3. Run `prisma migrate reset` (SQLite will recreate from scratch)

## Next Steps

1. Write implementation plan using writing-plans skill
2. Execute implementation plan
3. Deploy to production
4. Monitor and verify
