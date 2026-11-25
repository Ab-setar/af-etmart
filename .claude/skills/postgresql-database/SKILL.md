---
name: postgresql-database
description: PostgreSQL schema, migration, query and transaction standards for AF-EtMart. Use when designing tables, writing migrations, constraints, indexes or SQL, or handling money, stock and orders.
---

# PostgreSQL Database

## Naming and types

- `snake_case`; plural table names; foreign keys as `<singular>_id`.
- Primary keys: `BIGINT GENERATED ALWAYS AS IDENTITY`, or UUID where IDs must not be guessable. Decide once per kind of table. Hard-to-guess IDs never replace authorization.
- Money: integer minor units (`price_santim BIGINT`, currency ETB). Never `REAL` or `DOUBLE PRECISION`. The `pg` driver returns `BIGINT` as a string; convert deliberately and safely.
- Time: `TIMESTAMPTZ`, stored as instants, shown in `Africa/Addis_Ababa`.
- Text: `TEXT` with `CHECK` limits where needed. Phone numbers normalized to `+251...`. UTF-8 for Amharic.
- Small stable value sets: `CHECK` constraint or lookup table. Use native enums only when values rarely change.

## Core model (starting point, confirm before migrating)

- `users` (role: customer, seller, admin)
- `stores` owned by a seller, with approval status
- `categories`, `products` (belong to a store), `product_images`
- `addresses`, `carts`, `cart_items`
- `orders` (customer), `seller_orders` (one per store per order), `order_items`
- `payments`, `payment_events`, `reviews`, `refresh_tokens`, `audit_logs`

## Integrity

- Enforce rules in the database as well as in code: `NOT NULL`, `UNIQUE`, `CHECK`, foreign keys.
- Choose `ON DELETE` deliberately. Do not cascade away orders, payments or audit history; deactivate instead.
- Examples: price and stock `>= 0`, quantity `> 0`, rating between 1 and 5, one cart line per cart and product.
- Case-insensitive unique email: unique index on `lower(email)`.
- Order items snapshot product name and unit price at purchase time.
- `created_at` and `updated_at` on mutable tables, with one consistent way to maintain `updated_at`.
- Normalize first. Denormalize only for a measured reason and document how it stays correct.

## Indexes

- PostgreSQL does not index foreign keys automatically; add indexes for those used in joins, filters or deletes.
- Design from real queries: composite column order, partial indexes for active rows.
- For product search start with PostgreSQL full-text search or `pg_trgm` before any search service. Test Amharic text; English stemming does not apply to it.
- Verify important queries with `EXPLAIN (ANALYZE, BUFFERS)` on realistic data. `ANALYZE` executes the statement, so never run it on writes against real data.

## Queries

- Always parameterize values (`$1`). Allow-list any identifier such as a sort column.
- Select needed columns, not `SELECT *`. Avoid N+1 queries.
- Paginate with a stable order and unique tie-breaker; use keyset pagination for large lists.
- Scope seller and customer queries by owner in the `WHERE` clause.

## Transactions and concurrency

- Wrap multi-step changes in a transaction on a single pooled client: `BEGIN`, `COMMIT`, `ROLLBACK`, always release.
- Prevent overselling: `UPDATE ... SET stock = stock - $1 WHERE id = $2 AND stock >= $1`, or lock rows with `SELECT ... FOR UPDATE` in a consistent order.
- Keep transactions short; no network calls inside them.
- Unique constraints on idempotency keys and provider event IDs.

## Migrations

- Every schema change is a versioned migration in Git, run by one approved tool.
- Never edit a migration that has been applied to a shared database; add a new one.
- Provide a rollback or state why it is irreversible. Plan data backfills; build large indexes with `CONCURRENTLY` outside a transaction.
- Ask before any destructive operation (drop, truncate, bulk delete).
- Seed data is clearly fake and never contains real credentials or personal data.
- The application's database role has least privilege, not superuser.

## Done means

- Migrations apply to an empty database; rollback tried where provided.
- Constraint violations and concurrent stock updates are tested.
- If no database was available, say the SQL is unverified.
