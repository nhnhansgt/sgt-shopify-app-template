# MySQL Migration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Migrate Shopify app session storage from SQLite to MySQL for production-ready database infrastructure.

**Architecture:** Update Prisma datasource configuration from SQLite to MySQL, configure environment variables for database connection, and apply schema migrations to create Session table in MySQL. No code changes required - only configuration updates.

**Tech Stack:** Prisma 6.19.0, MySQL 8.0+, Node.js 20+, Shopify App Session Storage (Prisma adapter)

---

## Prerequisites

Before starting this implementation plan, ensure you have:
- MySQL 8.0+ available (local or remote instance)
- Access to create databases and users on MySQL server
- MySQL client tools available (for testing)
- Project dependencies installed: `npm install`

---

## Task 1: Verify Current State

**Files:**
- Read: `prisma/schema.prisma`
- Read: `package.json`
- Check: `.env` (if exists)

**Step 1: Verify current Prisma configuration**

Read `prisma/schema.prisma` to confirm current datasource:
```bash
cat prisma/schema.prisma | grep -A 3 "datasource db"
```

Expected output:
```prisma
datasource db {
  provider = "sqlite"
  url      = "file:dev.sqlite"
}
```

**Step 2: Check if .env file exists**

```bash
ls -la .env .env.example 2>/dev/null || echo "No .env files found"
```

**Step 3: Check package.json for Prisma version**

```bash
cat package.json | grep -A 2 '"@prisma/client"'
cat package.json | grep -A 2 '"prisma"'
```

Expected: Prisma version should be 6.19.0 or higher

**Step 4: Commit current state (baseline)**

```bash
git status
git add -A
git commit -m "chore: baseline commit before MySQL migration"
```

---

## Task 2: Setup MySQL Database

**Files:**
- System: MySQL server configuration

**Step 1: Create database and user**

If you're using local MySQL, create the database and user:

```bash
mysql -u root -e "CREATE DATABASE shopify_app_dev;"
mysql -u root -e "CREATE USER 'shopify_app'@'localhost' IDENTIFIED BY 'apppassword';"
mysql -u root -e "GRANT ALL PRIVILEGES ON shopify_app_dev.* TO 'shopify_app'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"
```

**Note:** Adjust the MySQL root username and password as needed. If you're using a remote MySQL instance, create the database and user on your server.

**Step 2: Verify connection**

```bash
mysql -ushopify_app -papppassword -e "SELECT 1"
```

Expected: Output shows `+---+ | 1 | +---+`

**Note:** If you're using a remote MySQL instance or cloud database (AWS RDS, Google Cloud SQL, etc.), note your connection details (host, port, user, password) for Task 3.

---

## Task 3: Create Environment Configuration Template

**Files:**
- Create: `.env.example`
- Create: `.env` (local development)

**Step 1: Create .env.example template**

Create file `.env.example`:

```bash
# Prisma Database Configuration
# For local development:
DATABASE_URL="mysql://shopify_app:apppassword@localhost:3306/shopify_app_dev"

# For production (managed cloud database):
# DATABASE_URL="mysql://username:password@host:3306/database_name"

# Alternative: Separate connection parameters
# DATABASE_USER="shopify_app"
# DATABASE_PASSWORD="apppassword"
# DATABASE_HOST="localhost"
# DATABASE_PORT=3306
# DATABASE_NAME="shopify_app_dev"
```

**Step 2: Create .env for local development**

Create file `.env`:

```bash
DATABASE_URL="mysql://shopify_app:apppassword@localhost:3306/shopify_app_dev"
```

**Step 3: Verify .gitignore includes .env**

```bash
cat .gitignore | grep "^\.env$"
```

If not found, add to `.gitignore`:
```bash
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore
echo ".env.*.local" >> .gitignore
```

**Step 4: Commit environment configuration**

```bash
git add .env.example .gitignore
git commit -m "chore: add MySQL environment configuration"
```

---

## Task 4: Update Prisma Schema

**Files:**
- Modify: `prisma/schema.prisma:11-14`

**Step 1: Update datasource configuration**

Edit `prisma/schema.prisma`, change lines 11-14:

**BEFORE:**
```prisma
datasource db {
  provider = "sqlite"
  url      = "file:dev.sqlite"
}
```

**AFTER:**
```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}
```

**Step 2: Verify schema syntax**

```bash
npx prisma validate
```

Expected: Output says "The schema is valid"

**Step 3: Generate Prisma Client for MySQL**

```bash
npx prisma generate
```

Expected: Output shows "Generated Prisma Client" with no errors

**Step 4: Commit schema changes**

```bash
git add prisma/schema.prisma
git commit -m "feat: switch Prisma datasource from SQLite to MySQL"
```

---

## Task 5: Create Initial Migration

**Files:**
- Create: `prisma/migrations/YYYYMMDDHHMMSS_switch_to_mysql/migration.sql`

**Step 1: Create migration directory**

```bash
npx prisma migrate dev --name switch_to_mysql --create-only
```

Expected: Creates migration file in `prisma/migrations/` directory

**Step 2: Review generated migration**

```bash
cat prisma/migrations/*/migration.sql
```

Expected output should look like:
```sql
-- CreateTable
CREATE TABLE `Session` (
    `id` VARCHAR(191) NOT NULL,
    `shop` VARCHAR(191) NOT NULL,
    `state` VARCHAR(191) NOT NULL,
    `isOnline` BOOLEAN NOT NULL DEFAULT false,
    `scope` VARCHAR(191) NULL,
    `expires` DATETIME(3) NULL,
    `accessToken` VARCHAR(191) NOT NULL,
    `userId` BIGINT NULL,
    `firstName` VARCHAR(191) NULL,
    `lastName` VARCHAR(191) NULL,
    `email` VARCHAR(191) NULL,
    `accountOwner` BOOLEAN NOT NULL DEFAULT false,
    `locale` VARCHAR(191) NULL,
    `collaborator` BOOLEAN NULL DEFAULT false,
    `emailVerified` BOOLEAN NULL DEFAULT false,
    `refreshToken` VARCHAR(191) NULL,
    `refreshTokenExpires` DATETIME(3) NULL,

    PRIMARY KEY (`id`)
) DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**Step 3: Apply migration to database**

```bash
npx prisma migrate dev
```

Expected: Output shows migration applied successfully

**Step 4: Verify database schema**

```bash
npx prisma db pull
npx prisma studio
```

In Prisma Studio, verify `Session` table exists with all columns

**Step 5: Commit migration**

```bash
git add prisma/migrations
git commit -m "feat: create initial MySQL migration for Session table"
```

---

## Task 6: Verify Application Works with MySQL

**Files:**
- Test: `app/routes/app._index.tsx`
- Test: Application startup

**Step 1: Start development server**

```bash
npm run dev
```

Expected: Server starts without database connection errors

**Step 2: Check logs for errors**

Look for:
- ✅ No "database connection" errors
- ✅ No "PrismaClientInitializationError"
- ✅ Server listening on port

**Step 3: Test app installation on development shop**

If you have a dev shop:
1. Navigate to `http://localhost:3000`
2. Follow app installation flow
3. Verify authentication works

**Step 4: Verify session stored in MySQL**

Open Prisma Studio in another terminal:
```bash
npx prisma studio
```

Check `Session` table - you should see a new session record

**Step 5: Stop dev server**

Press `Ctrl+C` in the terminal

**Step 6: Commit verification**

```bash
git add -A
git commit -m "test: verify application works with MySQL backend"
```

---

## Task 7: Update Documentation

**Files:**
- Modify: `CLAUDE.md`
- Modify: `README.md` (if exists)

**Step 1: Update CLAUDE.md with MySQL information**

Edit `CLAUDE.md`, find the "Database" section and update:

**BEFORE:**
```markdown
## Database

Default: SQLite via Prisma (`file:dev.sqlite`)

To switch databases:
...
```

**AFTER:**
```markdown
## Database

**Default:** MySQL via Prisma

**Connection:** Configure via `DATABASE_URL` in `.env`

**Local Development:**
```bash
# Create database (if not exists)
mysql -u root -e "CREATE DATABASE shopify_app_dev;"
```

**Environment Variables:**
```bash
DATABASE_URL="mysql://user:password@host:3306/database_name"
```

**To switch databases:**
1. Update `datasource db` provider in `prisma/schema.prisma`
2. Update `DATABASE_URL` in `.env`
3. Run `npm run setup`
```

**Step 2: Add setup instructions to README**

If `README.md` exists, add section:

```markdown
## Quick Start

### Prerequisites

- Node.js >= 20.19
- MySQL 8.0+

### Setup

1. Install dependencies:
   ```bash
   npm install
   ```

2. Create database:
   ```bash
   mysql -u root -e "CREATE DATABASE shopify_app_dev;"
   ```

3. Setup database schema:
   ```bash
   npm run setup
   ```

4. Start development server:
   ```bash
   npm run dev
   ```
```

**Step 3: Commit documentation updates**

```bash
git add CLAUDE.md README.md
git commit -m "docs: update documentation for MySQL configuration"
```

---

## Task 8: Create Production Deployment Guide

**Files:**
- Create: `docs/deployment/mysql-production.md`

**Step 1: Create production deployment guide**

Create file `docs/deployment/mysql-production.md`:

```markdown
# MySQL Production Deployment Guide

## Cloud Database Setup

### AWS RDS

1. Create RDS MySQL instance:
   - MySQL version 8.0+
   - Instance class: db.t3.micro (dev) or higher (prod)
   - Storage: 20GB minimum, autoscaling enabled
   - VPC: Same VPC as application

2. Configure security:
   - VPC security group allowing access from app server
   - Password authentication
   - IAM role for backup/restore

3. Connection string:
   ```
   DATABASE_URL="mysql://user:pass@instance.xxx.region.rds.amazonaws.com:3306/shopify_app_prod"
   ```

### Google Cloud SQL

1. Create Cloud SQL instance:
   - MySQL 8.0
   - Machine type: db-f1-micro (dev) or higher (prod)
   - Region: Same as application

2. Configure connections:
   - Private IP (recommended)
   - Or Cloud SQL proxy

3. Connection string:
   ```
   DATABASE_URL="mysql://user:pass@localhost:3306/shopify_app_prod"
   # (when using Cloud SQL proxy)
   ```

### Azure Database for MySQL

1. Create MySQL server:
   - MySQL 8.0
   - Pricing tier: Basic (dev) or General Purpose (prod)
   - Compute + storage scaling

2. Configure firewall:
   - Allow access from app server IP
   - Enable SSL enforcement

3. Connection string:
   ```
   DATABASE_URL="mysql://user:pass@server.mysql.database.azure.com:3306/shopify_app_prod"
   ```

## Environment Configuration

### Set production DATABASE_URL

```bash
# Via Shopify CLI
npm run env var DATABASE_URL "mysql://user:pass@host:3306/db"

# Or manually in production environment
export DATABASE_URL="mysql://user:pass@host:3306/db"
```

### Initial Setup

```bash
# Generate Prisma Client
npm run setup

# Verify connection
npx prisma db pull
```

## Monitoring

### Check connection pool

```sql
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
```

### Check database size

```sql
SELECT
  table_schema AS 'Database',
  ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
WHERE table_schema = 'shopify_app_prod';
```

## Backup Strategy

- Enable automated backups (daily)
- Point-in-time recovery (7 days retention)
- Manual backup before schema changes

```bash
# Manual backup
mysqldump -h host -u user -p shopify_app_prod > backup.sql
```
```

**Step 2: Commit deployment guide**

```bash
git add docs/deployment/mysql-production.md
git commit -m "docs: add MySQL production deployment guide"
```

---

## Task 9: Cleanup SQLite Files

**Files:**
- Delete: `prisma/dev.sqlite`
- Delete: `prisma/dev.sqlite-journal`
- Delete: `prisma/migrations/*` (old SQLite migrations, if any)

**Step 1: Check for SQLite artifacts**

```bash
find . -name "*.sqlite*" -type f
ls -la prisma/ | grep sqlite
```

**Step 2: Remove SQLite database files**

```bash
rm -f prisma/dev.sqlite
rm -f prisma/dev.sqlite-journal
rm -f prisma/*.db
rm -f prisma/*.sqlite
```

**Step 3: Check for old SQLite migrations**

```bash
ls -la prisma/migrations/
```

If you see migration files from before the MySQL migration, remove them:
```bash
# Only remove OLD migrations, keep the MySQL one
# Use caution - verify files are old before deleting
```

**Step 4: Update .gitignore if needed**

Add SQLite patterns if not present:
```bash
cat .gitignore | grep "\.sqlite" || echo "*.sqlite" >> .gitignore
```

**Step 5: Commit cleanup**

```bash
git add -A
git commit -m "chore: remove SQLite database files and artifacts"
```

---

## Task 10: Final Verification and Testing

**Files:**
- Test: All application routes
- Test: Authentication flow
- Test: Session management

**Step 1: Run full test suite (if tests exist)**

```bash
npm run test
```

Expected: All tests pass

**Step 2: Verify Prisma configuration**

```bash
npx prisma validate
npx prisma format
```

Expected: No errors, schema properly formatted

**Step 3: Check TypeScript compilation**

```bash
npm run typecheck
```

Expected: No TypeScript errors

**Step 4: Test complete user flow**

1. Start fresh database:
   ```bash
   mysql -e "DROP DATABASE IF EXISTS shopify_app_dev;"
   mysql -e "CREATE DATABASE shopify_app_dev;"
   npm run setup
   ```

2. Start app:
   ```bash
   npm run dev
   ```

3. Install app on test shop
4. Verify authentication works
5. Check session in Prisma Studio
6. Reinstall app - verify new session created
7. Uninstall app - verify session deleted (if webhook configured)

**Step 5: Check for SQLite references**

```bash
grep -r "sqlite" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.json" --exclude-dir=node_modules --exclude-dir=.git .
```

Expected: No results (or only in git history/docs)

**Step 6: Final commit**

```bash
git add -A
git commit -m "test: complete verification of MySQL migration"
```

**Step 7: Create summary commit**

```bash
git tag -a v1.0.0-mysql-migration -m "Migrate from SQLite to MySQL

- Switch Prisma datasource to MySQL
- Update environment configuration
- Create initial migration
- Update documentation
- Remove SQLite artifacts
"
```

---

## Rollback Procedure

If you need to rollback to SQLite at any point:

### Step 1: Clean MySQL database (optional)

If you want to remove MySQL data:
```bash
mysql -e "DROP DATABASE IF EXISTS shopify_app_dev;"
```

### Step 2: Revert Prisma schema

```bash
git checkout main~1 prisma/schema.prisma
```

Or manually edit:
```prisma
datasource db {
  provider = "sqlite"
  url      = "file:dev.sqlite"
}
```

### Step 3: Revert .env file

```bash
# Remove DATABASE_URL from .env
# Or revert to previous version
```

### Step 4: Reset database

```bash
npx prisma migrate reset
npm run setup
```

### Step 5: Restart application

```bash
npm run dev
```

---

## Troubleshooting

### Issue: "Can't reach database server"

**Solution:**
1. Check DATABASE_URL in `.env`
2. Verify MySQL is accessible:
   ```bash
   mysql -h localhost -u shopify_app -papppassword -e "SELECT 1"
   ```
3. Check firewall and network settings if using remote MySQL

### Issue: "Authentication plugin 'caching_sha2_password' cannot be loaded"

**Solution:**
Update MySQL user to use native password authentication:
```bash
mysql -e "ALTER USER 'shopify_app'@'localhost' IDENTIFIED WITH mysql_native_password BY 'apppassword';"
mysql -e "FLUSH PRIVILEGES;"
```

### Issue: "Table doesn't exist"

**Solution:**
Run migrations:
```bash
npx prisma migrate deploy
```

### Issue: "Prisma Client generation failed"

**Solution:**
```bash
rm -rf node_modules/.prisma
npx prisma generate
```

---

## Success Criteria

Migration is complete when:
- ✅ Prisma schema uses `provider = "mysql"`
- ✅ `.env` contains valid `DATABASE_URL`
- ✅ Migration files exist in `prisma/migrations/`
- ✅ `Session` table exists in MySQL database
- ✅ Application starts without database errors
- ✅ Authentication flow works correctly
- ✅ Sessions are stored in MySQL (verify with Prisma Studio)
- ✅ No SQLite files remain in project
- ✅ Documentation updated
- ✅ All tests pass (if tests exist)
- ✅ TypeScript compilation succeeds

---

## Post-Migration Checklist

### Development
- [ ] `.env.example` documents required environment variables
- [ ] CLAUDE.md updated with MySQL instructions
- [ ] README.md includes MySQL setup steps

### Production
- [ ] MySQL database created on cloud provider
- [ ] `DATABASE_URL` configured in production environment
- [ ] Backup strategy configured
- [ ] Connection pooling configured (if needed)
- [ ] Monitoring setup for database metrics

### Testing
- [ ] Local development tested
- [ ] Authentication flow verified
- [ ] Session storage verified
- [ ] App install/reinstall tested
- [ ] TypeScript compilation passes
- [ ] No SQLite references remain in code

---

## Next Steps After Migration

1. **Performance Monitoring:**
   - Monitor query performance with Prisma logs
   - Check connection pool usage
   - Optimize slow queries if found

2. **Scaling Considerations:**
   - If high traffic, consider connection pooling (ProxySQL, etc.)
   - Monitor MySQL resource usage
   - Plan for read replicas if needed

3. **Security:**
   - Rotate database passwords regularly
   - Use SSL for database connections
   - Restrict database access to application servers only
   - Enable audit logging in production

4. **Backup Strategy:**
   - Automated daily backups
   - Point-in-time recovery enabled
   - Test restore procedure
   - Document disaster recovery process
