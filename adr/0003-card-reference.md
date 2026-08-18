# ADR 0003: Card Reference — Scryfall API + local cache

**Status:** Accepted
**Date:** 2026-08-18

## Context

The system needs authoritative card data — oracle text, mana cost, type
line, images, format legality — without keeping a full local mirror of the
~30,000+ card database, per an explicit project requirement: reference
card data via API, don't store everything locally indefinitely.

## Decision

Use the **Scryfall API** as the source of truth for card data. Resolve an
entire decklist in one or two calls via `POST /cards/collection`, which
accepts up to 75 card identifiers per request. Cache resolved cards
locally in **SQLite (via SQLDelight)**, keyed by Scryfall Oracle ID,
populated lazily as decks are imported. Refresh cache entries on a TTL or
an explicit "check for updates" action, rather than continuously syncing.

## Alternatives considered

- **Mirror Scryfall's full bulk-data dump locally.** Rejected — directly
  conflicts with the "API, not local mirror" requirement, and is
  unnecessary: our decks touch a few hundred unique cards at most, not
  thirty thousand.
- **Rely only on Forge's own bundled card data.** Considered as a
  supplement rather than a replacement — Forge's data is oriented toward
  rules/gameplay scripting, not the richer oracle text/rulings/image/
  pricing data Scryfall provides.

## Consequences

- Card lookups require network access — consistent with the project's
  stated internet-connection requirement.
- Need to respect Scryfall's rate-limit etiquette even though
  `/cards/collection` batching minimizes call volume.
- The local cache needs a real invalidation story for errata and
  banned/restricted-list changes — a card resolved once shouldn't be
  treated as permanently correct.
