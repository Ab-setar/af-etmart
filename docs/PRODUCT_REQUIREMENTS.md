# AF-EtMart: Product Requirements

> Status: **draft for owner review**. Nothing here is implemented. Items marked *(proposed)* are recommendations, not approved decisions.
> Open decisions are listed in [DEVELOPMENT_ROADMAP.md](DEVELOPMENT_ROADMAP.md#open-project-decisions).

## 1. Problem

*These are working hypotheses. Confirm them with real sellers and buyers.*

- Many small sellers promote goods through social media and chat, where stock, prices and order status are tracked by hand.
- Buyers have limited ways to compare sellers, trust them, or follow an order.
- Delivery addresses are often described by area and landmark, and online payment is not universal.

Suggested validation before or during week 1: interview 5 sellers and 5 buyers about how they sell, pay, deliver and handle returns.

## 2. Target users *(proposed)*

| Group | Initial target |
|---|---|
| Customers | Smartphone users in the launch city, often on limited mobile data, who want clear prices in ETB and cash on delivery |
| Sellers | Licensed small and medium retailers and producers who can arrange fulfilment in the launch city |
| Administrator | A small internal operations team |

Starting with one city (for example Addis Ababa) and a few categories is recommended but not decided.

## 3. Business objectives

1. Let approved sellers list products and manage stock and orders without technical help.
2. Let customers find products and place trustworthy orders.
3. Give the operator control over who sells and what is listed.
4. Keep accurate order, price and payment records for later commission, tax and payout work.

Proposed MVP acceptance targets (adjust as needed):

- All critical journeys in section 5 pass end to end on a phone.
- No test of concurrent buying results in negative stock.
- A pilot seller can add a first product in under 10 minutes.

## 4. Value proposition

- **Customers:** many verified sellers in one place, clear ETB prices, pay on delivery, visible order status.
- **Sellers:** an online storefront with stock and order management.
- **Operator:** a moderated marketplace with an audit trail.

## 5. Main user journeys

**Customer:** browse or search → product page (with seller details) → register or log in → add to cart → choose address and payment method → review the full order → place order → track each seller's part → cancel or request a return where allowed.

**Seller:** create account → submit seller application with business details → approval → complete store profile → add products and stock → receive seller order → confirm or reject → dispatch → record delivery and cash collected → view sales.

**Administrator:** log in → review applications → approve or reject with reason → manage categories → suspend products, sellers or users → monitor orders → handle disputes and returns.

## 6. Roles and permissions

One account has one role *(proposed)*. A seller keeps customer abilities but cannot buy their own products. Pending applicants stay customers. Administrators are created only by a controlled server-side script.

Legend: ✓ allowed, own = only their own records, – denied.

| Action | Guest | Customer | Seller | Admin |
|---|---|---|---|---|
| Register, log in | ✓ | ✓ | ✓ | log in only |
| Browse, search products | ✓ | ✓ | ✓ | ✓ |
| Manage profile, addresses | – | own | own | own |
| Manage cart, place orders | – | own | own | – |
| View order history, cancel, request return | – | own | own (as buyer) | view all, resolve |
| Submit seller application | – | own | – | review |
| Manage seller profile | – | – | own | view, suspend |
| Create, update, delete products | – | – | own, when approved | moderate |
| Manage inventory | – | – | own | view |
| View and process seller orders | – | – | own | view, intervene with reason |
| Sales summary | – | – | own | all |
| Manage categories | – | – | – | ✓ |
| Manage users | – | – | – | ✓ |

"Delete" means archive: products referenced by orders are never erased.

### Security risks and required controls

| Role | Main risks | Controls |
|---|---|---|
| Customer | Account takeover; reading others' orders or addresses; price tampering; fake cash-on-delivery orders | Rate limits; ownership checks; server-side pricing; order limits and admin cancellation; phone confirmation by seller |
| Seller | Fraudulent seller; editing another seller's products or orders; seeing excess customer data; malicious uploads; marking unpaid orders paid | Verification and approval; ownership scoping; only delivery data needed for their own seller order; upload checks; status history |
| Administrator | Stolen admin account; abuse of power; self-escalation | Strong passwords, second factor (post-MVP, or earlier if possible), no public registration, audit log, reason required for sensitive actions |

## 7. Functional requirements

Priority: **M** = MVP, **P** = postponed.

| ID | Requirement | P |
|---|---|---|
| FR-01 | Register, log in, log out, refresh session, change password | M |
| FR-02 | Manage profile and delivery addresses (area, subcity, woreda, landmark, phone) | M |
| FR-03 | Seller application, admin approval or rejection with reason | M |
| FR-04 | Seller profile with the legal and contact details shown to buyers | M |
| FR-05 | Categories managed by admin | M |
| FR-06 | Seller product create, edit, archive, with images and stock | M |
| FR-07 | Public listing with search, category, price filter, sort, pagination | M |
| FR-08 | Persistent cart for logged-in users | M |
| FR-09 | Checkout with full order review before placing | M |
| FR-10 | Multi-seller order split into seller orders | M |
| FR-11 | Cash on delivery | M (proposed) |
| FR-12 | Order status tracking and history | M |
| FR-13 | Cancellation and return requests, resolved by admin | M (basic) |
| FR-14 | Seller sales summary | M (basic) |
| FR-15 | Admin: users, sellers, products, orders, audit log | M (basic) |
| FR-16 | Online payment through a provider | P unless decided early |
| FR-17 | Ratings and reviews | P |
| FR-18 | Product variants (size, colour) | P (decide in week 1) |
| FR-19 | Automated commissions and seller payouts | P |
| FR-20 | Email or SMS notifications and self-service password reset | P unless a provider is chosen |
| FR-21 | Coupons, wishlists, chat, recommendations, courier tracking, native apps | P |

## 8. Non-functional requirements *(proposed targets)*

| Area | Requirement |
|---|---|
| Performance | Typical API reads under 500 ms at p95 on the pilot data set; initial compressed JavaScript under 200 KB; resized, lazy-loaded images |
| Mobile and data | Usable from 360 px; usable on slow 3G-like connections |
| Accessibility | WCAG 2.2 AA target |
| Security | As in [SYSTEM_ARCHITECTURE.md](SYSTEM_ARCHITECTURE.md#7-security) |
| Reliability | Daily database backups with one restore rehearsed before launch |
| Integrity | Transactional checkout; no overselling; price snapshots |
| Privacy | Minimum personal data; see section 9 |
| Maintainability | Modular code, tests for critical flows, documented API |
| Localization | UTF-8, text easy to translate, ETB, East Africa Time |
| Scale | Pilot size to be agreed (suggested: 50 sellers, 5,000 products, 100 concurrent users) |

## 9. Legal and market constraints to verify

From web research on 2026-11-26. **This is not legal advice.** Have a qualified Ethiopian lawyer confirm what applies, including later amendments and directives.

| Source | What it says | Product effect |
|---|---|---|
| Electronic Transaction Proclamation 1205/2020, Art. 28 | Suppliers publish identity, contacts, registration numbers, full price, terms, delivery period and return policy; buyers can review and correct before ordering | Seller information, policy pages, review step |
| Art. 29 | Cancellation without reason within 7 days of receiving goods; refund within 30 days | Return requests in MVP |
| Art. 31 | Fulfil within 30 days unless agreed otherwise; refund within 7 days of notifying unavailability | Order age monitoring |
| Art. 41–42 | Platform operators verify seller identity, contacts and licences, keep records, and provide dispute settlement | Seller verification, complaint handling |
| Personal Data Protection Proclamation 1321/2024, Art. 22 | Locally collected personal data must be stored on a server or data centre in Ethiopia | **Constrains hosting, backups, image storage and third-party services** |
| Art. 33, 43–44 | Register with the authority before processing; notify breaches within 72 hours | Pre-launch registration, incident procedure |

Also confirm: platform business licence and registration (one law firm summary lists TIN, domain, logistics and recognised payment system evidence), VAT and receipt obligations, and rules on holding money on behalf of sellers.

Payments: providers found include Chapa, SantimPay and ArifPay, and wallets such as telebirr. A third-party guide says Chapa allows test mode immediately but requires business documents before live payments. **No provider is selected.**

## 10. Sources

- [Electronic Transaction Proclamation 1205/2020 (PDF)](https://chilot.wordpress.com/wp-content/uploads/2023/02/90d5d-electronic-transaction-proclamation-no.-1205-2020.pdf)
- [Proclamation 1321/2024 text (unofficial copy)](https://www.metaappz.com/References/ethiopian_laws/federal/pr_1321_2024/en/txt)
- [Digital Policy Alert: data localisation](https://digitalpolicyalert.org/event/24922-implemented-personal-data-protection-proclamation-proclamation-no-13212024-including-data-localisation-requirements)
- [Dablo Law Firm: platform operator registration](https://dablolawfirm.com/e-commerce-companies-can-now-register-as-an-electronic-platform-operator-code-85125-or-as-an-electronics-intra-platform-operator-code-85126/)
- [Chapa integration guide (third party)](https://dreamtech.et/guides/chapa-payment-integration-ethiopia)
- [Gateway comparison (published by a competing vendor)](https://payment.et/compare)
