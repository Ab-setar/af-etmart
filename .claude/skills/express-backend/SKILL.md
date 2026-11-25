---
name: express-backend
description: Node.js + Express.js API conventions for AF-EtMart, covering modular structure, validation, centralized error handling and API design. Use when creating or changing endpoints, middleware, services or anything under server/.
---

# Express Backend

Stack: Node.js (a supported LTS release), Express, JavaScript, PostgreSQL. Pick one module system (ES modules preferred) and use it throughout.

## Related skills

Use `postgresql-database` for schema and queries, `authentication-security` for anything touching identity or user input, and `testing-code-review` before calling work done.

## Structure (modular by domain)

```
server/src/
  app.js        builds the Express app (no listen, so tests can import it)
  server.js     starts and gracefully stops the process
  config/       reads and validates environment variables
  modules/      auth, users, stores, products, cart, orders, payments, admin
    <module>/   routes.js, controller.js, service.js, repository.js, schema.js
  middleware/   authenticate, authorize, validate, errorHandler, notFound
  db/           pool, transaction helper, migrations/
  lib/          errors, logger, small shared utilities
```

Create only the files a module needs.

## Layer responsibilities

- **Routes**: URL, method, middleware order.
- **Controller**: read validated input, call a service, send the response. No SQL, no business rules.
- **Service**: business rules, ownership checks, transactions. No `req` or `res`.
- **Repository**: parameterized SQL only.

Modules talk to each other through services, never another module's repository.

## API design

- Prefix with `/api/v1`. Plural nouns: `/products`, `/orders/:orderId`.
- Correct status codes: 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500.
- Choose 400 or 422 for validation failures once and stay consistent.
- Success: `{ "data": ..., "meta": ... }`. Error: `{ "error": { "code", "message", "details", "requestId" } }`.
- Paginate every list, with a default and maximum page size. Allow-list sort and filter fields.
- Creating orders and payments must accept an idempotency key so retries do not duplicate them.

## Validation

- Validate `params`, `query` and `body` with a schema before the controller runs. Use one validation library, approved by the owner.
- Reject unknown fields and pass only the validated result onward. Never spread `req.body` into a query.
- Never trust client-supplied price, total, role, user ID, seller ID or order status. Derive them on the server.

## Errors

- Define a small `AppError` family (validation, unauthorized, forbidden, not found, conflict).
- One error handler, registered last, formats every error.
- Express 5 forwards rejected promises to it; on Express 4 wrap async handlers.
- Map known database errors (unique violation `23505` to 409).
- Clients never receive stack traces, SQL or internal messages.
- Never swallow errors. Handle process-level failures by logging and exiting cleanly.

## Cross-cutting

- Validate configuration at startup and fail fast. Commit `.env.example`, never `.env`.
- Structured logs with a request ID. Never log passwords, tokens, cookies or full payment data.
- Security headers, explicit CORS origin list, body size limits, rate limits on sensitive routes.
- Timeouts on outbound calls. Close the server and database pool on shutdown.
- Health endpoint that reports database connectivity without exposing details.

## Payments and external services

Put each provider behind a small interface. Verify webhook signatures, confirm status with the provider, and process each event once. A simulated payment must be labelled as simulated in code, UI and reports.

## Done means

- Success, validation, 401, 403, 404 and conflict cases are tested for the endpoint.
- Server starts from a clean setup; lint and tests pass.
- Endpoint contract documented. Stubs return an explicit error such as 501, never fake success.
