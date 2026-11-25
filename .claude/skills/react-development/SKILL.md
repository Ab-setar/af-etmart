---
name: react-development
description: React + Vite + JavaScript + Tailwind CSS conventions for AF-EtMart. Use when creating or changing components, pages, routing, forms, client state, API calls or anything under client/.
---

# React Development

Stack: React, Vite, JavaScript (no TypeScript), Tailwind CSS. This is a client-rendered single-page app.

## Related skills

- `frontend-design`: visual direction and polish.
- `react-best-practices`: performance rules. This project does not use Next.js or React Server Components, so skip the `server-*` rules and any `next/*` advice. Use `React.lazy` where a rule says `next/dynamic`. Its examples are TypeScript; write JavaScript.
- Check the installed React version before using newer APIs.

## Before writing code

1. Read the neighbouring code and reuse existing components, hooks and utilities.
2. Confirm the API contract the screen depends on. If the endpoint does not exist, say so.
3. Ask before adding a dependency. State what it replaces and why a few lines of our own code are not enough.

## Structure (feature-based)

```
client/src/
  app/         router, providers, layouts
  features/    auth, catalog, cart, checkout, orders, seller, admin
    <feature>/ components/, hooks/, api.js, pages/
  components/  shared UI primitives (Button, Input, Modal, Table)
  hooks/       shared hooks
  lib/         API client, formatting, constants
```

Features do not import each other's internals. Move code to a shared folder when a second feature needs it, not before.

## Components

- Function components. One main component per file, `PascalCase.jsx`; hooks are `useName.js`.
- Keep components focused. Split when one mixes data loading, business rules and large markup.
- Never define a component inside another component.
- Compose with `children` instead of adding many boolean props.
- Check props at feature boundaries and document non-obvious ones with JSDoc.

## State and data

- Keep state as close as possible to where it is used. Derive values during render instead of syncing them in effects.
- URL holds search, filters, sorting and page number so views can be shared and survive reload.
- Context is for low-frequency global data (session, theme). Add a state library only after a concrete need.
- All HTTP goes through one API client in `lib/`: base URL, credentials, error shape, token refresh. No scattered `fetch` calls.
- Choose one approach for server data (small shared hooks, or a query library approved by the owner) and use it everywhere.
- Cancel or ignore stale requests so older responses cannot overwrite newer ones.
- Effects are for synchronizing with external systems and must clean up. User actions belong in event handlers.

## Required UI states

Every data view handles loading, empty, error (with a retry or next step) and success. Submit buttons are disabled while pending to prevent duplicate orders or payments.

## Forms

- Use native form elements with visible labels.
- Validate on the client for usability and show messages next to fields. The server remains the authority; display its field errors.
- Keep the user's input after a failed submission.

## Routing and roles

- Separate route groups for customer, seller and admin, lazy-loaded.
- Route guards only improve the experience. They are not security; the API enforces authorization.
- Include not-found and forbidden pages and an error boundary.

## Styling

- Tailwind utilities with design tokens defined in one place. Follow the installed Tailwind version's configuration style.
- Do not build class names from fragments (`bg-${color}-500`); map to complete class strings.
- Mobile-first. Extract a component when a class combination repeats.

## Accessibility

- Semantic HTML first; ARIA only when HTML cannot express it.
- Full keyboard use, visible focus, focus management in dialogs and after navigation.
- Meaningful `alt` text, WCAG 2.2 AA contrast, announced form errors, reduced motion respected.

## Security and configuration

- Anything in a `VITE_` variable is public. Never put secrets in the client.
- Do not use `dangerouslySetInnerHTML` with user or seller content.
- Follow `authentication-security` for token handling.

## Done means

- Lint, tests and production build pass (report any that are not set up yet).
- Checked at about 360px, 768px and 1280px, with keyboard only.
- All required UI states exercised.
- Mock data and unfinished parts are named in the summary. A screen on mock data is not a finished feature.
