# ADR 0002: Deck Import — Moxfield

**Status:** Superseded by ADR 0008
**Date:** 2026-08-18

## Context

Decks need to come in from Moxfield, where they're already built and
maintained. Moxfield's website is Cloudflare-protected and renders
client-side — a plain page fetch returns metadata but not the actual card
list (confirmed directly). Moxfield does not publish an official public
API.

## Decision

**Primary:** call Moxfield's unofficial API —
`GET https://api2.moxfield.com/v2/decks/all/{publicId}` — where
`publicId` is the identifier segment of the deck's URL. This returns full
deck JSON: mainboard, commander(s), categories.

**Fallback:** parse Moxfield's manual plaintext export (available from the
deck's own Export menu) if the API endpoint is unavailable, rate-limited,
or changes shape without notice.

## Alternatives considered

- **Scrape the rendered deck page directly.** Rejected — client-side
  rendering means the useful content isn't present in the initial HTML
  response.
- **Manual re-entry of decklists.** Rejected — error-prone and defeats the
  purpose of "import."

## Consequences

- Dependency on an undocumented, reverse-engineered endpoint. It could
  change, get more aggressively rate-limited, or get pulled behind
  stricter bot protection without warning — this is not a stable contract.
- Some routes may require handling a Cloudflare JS-challenge rather than a
  plain HTTP request; a naive Ktor client call may get blocked outright.
  Needs verification against the real endpoint before relying on it, and a
  realistic `User-Agent`/retry strategy.
- The plaintext-export fallback parser should be built as a real, working
  path — not a "just in case" stub — since it's the only route immune to
  the API changing shape.
