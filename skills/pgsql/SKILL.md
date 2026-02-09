---
name: pgsql
description: Use when the user wants to query a PostgreSQL database. Reads DATABASE_URL from the skill's .env file and executes read-only SQL queries using psql. Supports Prisma schema awareness - automatically translates Prisma model/field names to actual PostgreSQL table/column names. Triggers on keywords like "query database", "check DB", "select from", "pgsql".
argument-hint: <SQL query> - e.g., "SELECT * FROM studios LIMIT 5"
user-invocable: true
allowed-tools: Bash(/opt/homebrew/opt/libpq/bin/psql *), Bash(psql *), Read, Grep, Glob
---

# PostgreSQL Query Skill

Execute read-only SQL queries against a PostgreSQL database.

## Prerequisites

- `psql` must be installed. Preferred path: `/opt/homebrew/opt/libpq/bin/psql`. Fall back to `psql` on PATH.
- A `pgsql.env` file must exist in the project's `.claude/` directory.

## Configuration

The user must create a `pgsql.env` file at `<project-root>/.claude/pgsql.env` with:

```env
# Required: PostgreSQL connection string
DATABASE_URL="postgresql://user:password@host:5432/dbname"

# Optional: absolute path to a Prisma schema file for name translation
PRISMA_SCHEMA_PATH="/path/to/prisma/schema.prisma"
```

## Workflow

### 1. Read Configuration

Read `<project-root>/.claude/pgsql.env` where `<project-root>` is the current working directory. Extract:
- `DATABASE_URL` (required) — the PostgreSQL connection string
- `PRISMA_SCHEMA_PATH` (optional) — absolute path to a Prisma schema file

If the file or `DATABASE_URL` is missing, stop and tell the user to create `.claude/pgsql.env` with the required configuration.

### 2. Prisma Schema Awareness (if configured)

If `PRISMA_SCHEMA_PATH` is set, read the Prisma schema file to understand:
- `@@map("table_name")` — model-to-table mappings
- `@map("column_name")` — field-to-column mappings

When the user writes queries using Prisma model names (PascalCase) or field names (camelCase), translate them to the actual PostgreSQL names before executing. For example:
- `Studio` → `studios`
- `walletAddress` → `wallet_address`
- `studioId` → `studio_id`

If the user writes raw SQL with correct PostgreSQL names, run it as-is.

### 3. Execute Query

Run the query using psql:
```bash
/opt/homebrew/opt/libpq/bin/psql "$DATABASE_URL" -c "<translated query>"
```

### 4. Present Results

Display the query results clearly. If the result set is large, summarize or highlight key findings.

## Safety Rules

This is a **READ-ONLY** skill. Do NOT execute any of the following:
- `INSERT`, `UPDATE`, `DELETE`
- `DROP`, `ALTER`, `TRUNCATE`
- `CREATE`, `GRANT`, `REVOKE`
- Any statement that modifies data or schema

If the user's query would modify data, refuse and explain why.

## Error Handling

1. **`pgsql.env` not found or `DATABASE_URL` missing**: Tell the user to create `.claude/pgsql.env` in their project root with `DATABASE_URL`
2. **psql not installed**: Suggest `brew install libpq`
3. **Connection refused**: The database may not be running — suggest checking Docker or the DB service
4. **Relation does not exist**: The table name may be wrong — if Prisma schema is configured, check `@@map` for the correct name

## Examples

**Simple query:**
```
User: /pgsql SELECT count(*) FROM studios
```

**Using Prisma names (auto-translated when schema is configured):**
```
User: /pgsql SELECT * FROM Studio WHERE walletAddress = '0x...'
→ Translates to: SELECT * FROM studios WHERE wallet_address = '0x...'
```

**Describe a table:**
```
User: /pgsql \d studios
```

**List all tables:**
```
User: /pgsql \dt
```
