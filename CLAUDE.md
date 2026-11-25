# AF-EtMart

A multi-vendor e-commerce marketplace for Ethiopia.

## Current status

Environment setup. No application code exists yet. Do not start implementation until the owner approves it.

## Roles

- **Customer**: browses, buys, tracks orders, reviews.
- **Seller**: manages a store, products, stock and fulfilment.
- **Administrator**: moderates sellers, products and the platform.

## Technology

React + Vite, JavaScript, Tailwind CSS, Node.js + Express, PostgreSQL, JWT authentication, Git and GitHub.

## Planned layout (not yet created)

```
client/   React app
server/   Express API and database migrations
docs/     decisions and API notes
.claude/  project skills
```

## Architecture principles

- One frontend and one modular API. No microservices or extra infrastructure without a demonstrated need.
- Organize by feature or domain. Layers: route, controller, service, repository.
- The server is the authority for prices, stock, permissions and order status.
- PostgreSQL enforces data integrity; multi-step changes use transactions.
- One order may include several sellers; each seller sees and fulfils only their own part.
- Configuration from environment variables; providers (payment, SMS, storage) behind small interfaces.

## Development rules

1. Ask before installing a dependency, changing the stack, running destructive operations, committing or pushing.
2. Make small, reviewable changes. Reuse before creating.
3. Validate input, handle errors centrally, enforce authorization on the server.
4. Test and verify before reporting completion.
5. Never present unfinished, mocked or untested work as complete.
6. Never commit secrets. Keep `.env.example` current.
7. If a requirement is unclear or undecided, ask.

## Frontend quality bar

- Mobile-first and usable from about 360px upward.
- Accessible (target WCAG 2.2 AA): semantic HTML, keyboard use, visible focus, labels, contrast, reduced motion.
- Visually polished and consistent, with shared design tokens.
- Considerate of slow or expensive mobile data: optimized images, small bundles.
- Prices in ETB; text kept easy to translate; fonts that render Ethiopic script where Amharic appears.

## Using the skills

Skills live in `.claude/skills/`. Load every skill that fits the task.

| Task | Skills |
|---|---|
| Visual design | `frontend-design` |
| React code | `react-development`, `react-best-practices` |
| API code | `express-backend` |
| Schema, SQL, migrations | `postgresql-database` |
| Login, permissions, input, uploads, payments | `authentication-security` |
| Tests, review, completion reports | `testing-code-review` |

If guidance conflicts: the owner's instructions come first, then this file, then project skills, then third-party skills. Security requirements are not relaxed for performance or looks.

`frontend-design` and `react-best-practices` are third-party skills. Do not edit them; see `.claude/skills/ATTRIBUTION.md`. Skip Next.js and server-component advice, which does not apply here.

## Open decisions

Do not assume these:

- Payment providers (for example Telebirr, Chapa) and cash on delivery
- Seller approval process, commissions and payouts
- Interface languages and calendar display
- Email and SMS verification
- Image storage and hosting
- Delivery and shipping model
- Libraries for validation, migrations, server data and testing
