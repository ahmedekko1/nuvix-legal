---
name: database-migrations
description: Database migration best practices for schema changes, data migrations, rollbacks, and zero-downtime deployments across PostgreSQL and common ORMs (Prisma, Drizzle, Kysely, Django).
origin: ECC
---

# Database Migration Patterns

Safe, reversible database schema changes for production systems.

## When to Activate

- Creating or altering database tables
- Adding/removing columns or indexes
- Running data migrations (backfill, transform)
- Planning zero-downtime schema changes

## Core Principles

1. **Every change is a migration** — never alter production databases manually
2. **Migrations are forward-only in production** — rollbacks use new forward migrations
3. **Schema and data migrations are separate** — never mix DDL and DML
4. **Test against production-sized data** — a migration that works on 100 rows may lock on 10M
5. **Migrations are immutable once deployed** — never edit a migration that has run

## Migration Safety Checklist

- [ ] New columns have defaults or are nullable (never add NOT NULL without default)
- [ ] Indexes created concurrently (not inline on existing large tables)
- [ ] Data backfill is a separate migration from schema change
- [ ] Tested against a copy of production data
- [ ] Rollback plan documented

## PostgreSQL Patterns

```sql
-- GOOD: Nullable column, no lock
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- GOOD: Column with default (instant on Postgres 11+)
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- BAD: NOT NULL without default on existing table (locks and rewrites all rows)
-- ALTER TABLE users ADD COLUMN role TEXT NOT NULL;

-- GOOD: Non-blocking index
CREATE INDEX CONCURRENTLY idx_users_email ON users (email);

-- BAD: Blocks writes
-- CREATE INDEX idx_users_email ON users (email);
```

## Rename a Column (Zero-Downtime — Expand/Contract)

```
Step 1: Add new column (migration 001)
Step 2: Backfill data (migration 002)
Step 3: Deploy app writing to both columns
Step 4: Drop old column (migration 003)
```

## Large Data Migrations (Batching)

```sql
DO $$
DECLARE batch_size INT := 10000; rows_updated INT;
BEGIN
  LOOP
    UPDATE users SET normalized_email = LOWER(email)
    WHERE id IN (SELECT id FROM users WHERE normalized_email IS NULL LIMIT batch_size FOR UPDATE SKIP LOCKED);
    GET DIAGNOSTICS rows_updated = ROW_COUNT;
    EXIT WHEN rows_updated = 0;
    COMMIT;
  END LOOP;
END $$;
```

## Prisma Workflow

```bash
npx prisma migrate dev --name add_user_avatar   # Create + apply (dev)
npx prisma migrate deploy                        # Apply in production
npx prisma generate                              # Regenerate client
```

## Drizzle Workflow

```bash
npx drizzle-kit generate   # Generate from schema
npx drizzle-kit migrate    # Apply migrations
```

## Django Workflow

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py makemigrations --empty app_name -n description  # Custom SQL
```

## Anti-Patterns

| Anti-Pattern | Better Approach |
|-------------|-----------------|
| Manual SQL in production | Always use migration files |
| Editing deployed migrations | Create new migration instead |
| NOT NULL without default | Add nullable, backfill, then add constraint |
| Inline index on large table | CREATE INDEX CONCURRENTLY |
| Schema + data in one migration | Separate migrations |
| Drop column before removing code | Remove code first, drop column next |
