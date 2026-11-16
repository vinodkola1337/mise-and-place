# Project Charter — curated companion

<!--
Hand-authored, display-only companion to the authoritative `.kittify/charter/charter.yaml`.
Never a charter-resolving input; `charter generate` never overwrites it.
-->

## Purpose

Mise & Place is a modular, cloud, multi-tenant restaurant platform for small and mid-size Indian
restaurants. It starts as a portfolio project and must be able to grow into a paid product.
v1 is "done" when it is a demo the owner can show and explain, not a complete product.

## Scope

- **v1:** dine-in only. Users are guests (anonymous sessions, no login) and owners. Roles
  (guest, owner, staff, kitchen) and per-role pages are planned from the start.
- **Guest flow:** scan table QR (encodes outlet and table) → browse menu (ingredients, allergens,
  customizations) → cart → order → pay.
- **Payments v1:** UPI QR plus staff "mark as paid". No real gateway.
- **Per-outlet settings, not code paths:** pay-before vs pay-after, and what releases an order to
  the kitchen.
- **Out of scope:** GST invoicing, accounting, reviews.
- **v2:** kitchen staff, cloud kitchens, PWA.

## Architecture rules

- One Postgres database; every row scoped by restaurant and outlet, enforced with row-level security.
- Vue 3 + Vite + TypeScript frontend, NestJS backend, Postgres. Modular monolith — no premature
  microservices.
- Modules split by job: Core (menu, outlets, users and roles, settings), Ordering, Payments,
  Records (reports only); Kitchen in v2. Modules interact through order events, never by reading
  or writing each other's data.
- Features are switched on per outlet with feature flags. Code checks flags, never plan names.
- UI is built with the Axis design system. No one-off components that Axis should own; list gaps.
- Mobile-first. Guest screens phone-first; owner screens work on phones, designed for tablet and
  laptop. Load from cached data where possible and show a clear offline state, so the v2 PWA is an
  add-on, not a rewrite.

## Why these rules

The costliest failure is a **cross-tenant data leak**, so tenant isolation is an error-level rule
with mandatory tests and no exceptions. Payment and order-release state are the other mandatory
test areas because they are where wrong behaviour costs real money. Elsewhere testing is
deliberately light for a solo developer: typecheck and lint must pass, and other business rules are
tested where cheap. This is a doctrine-gap note: the "one developer, flag-driven, event-linked
modular monolith" policy has no exact built-in paradigm, so it lives as project directives
`MP-001` to `MP-010` in `charter.yaml`, selected alongside `domain-driven-design` and `atomic-design`.

## Process

- Short-lived feature branches per work package, Conventional Commits, zero required approvals.
- Merge only after the self-review checklist (MP-008) passes: tenant scoping and RLS, flags not plan
  names, event-based module boundaries, Axis reuse, ADR written where needed, typecheck and lint green.
- Big decisions get a short ADR. Hosting and CI are not chosen yet; that is the first ADR.
- Exceptions need a written reason and removal date, and never apply to tenant isolation or
  payment rules.
- Amend this charter by running `/spec-kitty.charter` with the new guidance.

## Terminology

Keep distinct: restaurant vs outlet, table, guest session, order vs order event, module,
feature flag vs plan. Avoid "tenant" in user-facing text.
