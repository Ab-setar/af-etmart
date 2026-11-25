---
name: authentication-security
description: Security rules for AF-EtMart covering passwords, JWT and refresh tokens, role and ownership authorization, input validation, uploads, payments and secrets. Use for any login, permission, user input, file, payment or sensitive data work, and when reviewing it.
---

# Authentication and Security

Use maintained libraries for cryptography and tokens. Never invent your own. Deny by default.

## Passwords

- Argon2id, at least 19 MiB memory, 2 iterations, parallelism 1. If bcrypt is chosen instead: cost 10 or higher and enforce its 72-byte input limit.
- Minimum length 8 or more, allow long passphrases and paste, block common passwords, no forced composition rules.
- Never log, return or store plaintext passwords. Exclude the hash from API responses.

## Tokens

- Access token: short-lived (about 15 minutes), kept in memory on the client, sent as `Authorization: Bearer`.
- Refresh token: random or signed, in an `HttpOnly`, `Secure`, `SameSite` cookie with a narrow path. Store only its hash in the database.
- Rotate on each use. If a rotated token is reused, revoke that token family.
- Never keep refresh tokens in `localStorage` or in URLs.
- When verifying JWTs, fix the allowed algorithm and check signature, expiry, issuer and audience. Decoding is not verifying.
- Strong secrets from the environment, different per environment and token type.
- Put no sensitive data in a JWT payload; it is readable.
- Logout, password change and suspension revoke refresh tokens. Account for access tokens staying valid until expiry; check current status on sensitive actions.
- Protect cookie-authenticated endpoints against CSRF (`SameSite` plus origin check or CSRF token). Deployment domains affect cookie settings, so confirm them.

## Authorization

Check on the server for every protected request:

1. Authenticated?
2. Role allowed?
3. Owns this resource? (customer: own orders; seller: own store, products and seller orders)
4. Is the action valid in the current state?

| Role | Scope |
|---|---|
| Customer | Own profile, addresses, cart, orders, reviews |
| Seller | Own store and its products and fulfilment, only after approval if approval is adopted |
| Administrator | Platform moderation; actions are audit-logged |

- Public registration never creates an administrator; role is never taken from unchecked input.
- Return 404 where revealing existence would leak information.
- Test that user A cannot read or change user B's data.

## Input and output

- Schema-validate all input, including webhooks and file metadata.
- Parameterized SQL only. Escape output; sanitize any permitted rich text.
- Prevent mass assignment with explicit field lists.
- Validate redirect targets and server-fetched URLs.

## Abuse protection

- Rate limit login, registration, password reset, OTP and checkout.
- Generic messages that do not reveal whether an account exists.
- Reset and verification tokens: random, hashed, single-use, expiring. OTPs: attempt limits.

## File uploads

Allow-list types, verify content as well as extension, limit size, generate server file names, store outside executable paths. Treat SVG as active content.

## Payments

Compute amounts on the server. Verify webhook signatures, confirm with the provider, process once. Never store card numbers or wallet PINs.

## Operations

- No secrets in Git. If one is committed, rotate it; deleting the file is not enough.
- This repository may be public: check diffs before every commit.
- HTTPS in production, security headers, strict CORS (no wildcard with credentials).
- Review dependencies before adding them and audit them regularly.
- Collect only needed personal data. Flag privacy or payment regulation questions for the owner to confirm with a qualified adviser.

## Done means

- Negative tests: no token, expired, tampered, wrong role, wrong owner, reused refresh token.
- Findings reported with severity. Never claim the system is "secure"; state what was checked and what was not.
