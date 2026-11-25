---
name: testing-code-review
description: Testing strategy, verification and code review checklist for AF-EtMart. Use when writing or running tests, fixing bugs, reviewing changes, preparing commits or pull requests, or reporting that work is complete.
---

# Testing and Code Review

## Tools

Proposed: Vitest, React Testing Library, Supertest, and Playwright for a few end-to-end flows. Confirm with the owner before installing. Use whatever the repository already has.

## What to test

| Level | Focus |
|---|---|
| Unit | Pricing, totals, order state transitions, validation, permissions |
| API integration | Each endpoint against a real test PostgreSQL database |
| Component | Forms, UI states, accessible interaction |
| End-to-end | The critical flows below |

Critical flows:

1. Customer registers, logs in, browses, adds to cart, checks out, sees the order.
2. Seller sets up a store, creates a product, fulfils their part of an order.
3. Administrator approves or suspends a seller and moderates a product.
4. A multi-seller order splits correctly.
5. Failures: out of stock, concurrent purchase, repeated submission, failed or duplicate payment event, expired session.

Every API endpoint covers success, invalid input, unauthenticated, wrong role, wrong owner, not found and conflict, where applicable.

## How to write tests

- Test behaviour, not implementation. Query the UI by role and label.
- Each test is independent and deterministic; reset data and control time.
- Mock only external boundaries such as payment, SMS and email.
- Never point tests at a production or shared database.
- For a bug, first write a test that fails, then fix it.
- Do not weaken, skip or delete a test just to make the suite pass. Explain any change to expected behaviour.

## Verification

Run the checks that exist: lint, tests, production build, migrations on an empty database, and a manual check of the changed flow. Report each command and its actual result.

## Review checklist

- Does it do what was asked, with no unrelated changes?
- Edge cases and error paths handled?
- Server-side validation, authorization and ownership present?
- Queries parameterized, indexed, transactional where needed?
- UI states, responsiveness and accessibility covered?
- Duplication, needless abstraction or unjustified dependency?
- Secrets, debug logs or commented-out code in the diff?
- Tests meaningful and documentation updated?

Label findings **blocker**, **should fix** or **suggestion**, with file, line, impact and a fix.

## Reporting

State separately:

- **Done and verified** (how)
- **Done but not verified** (why)
- **Not done** (including stubs, mocks and TODOs)
- **Known problems**

Never say tests pass unless they were run and passed. No tests found or none run is not a pass. Coverage percentage is a hint, not proof.

## Git

Small focused commits with clear messages, on a feature branch after the initial setup. Review the diff first. Commit or push only with the owner's approval.
