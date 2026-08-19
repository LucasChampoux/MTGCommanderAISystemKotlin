# ADR 0008: Deck Import — Moxfield Export (Manual)

**Status:** Accepted
**Date:** 2026-08-19
**Supersedes:** ADR 0002

## Context

ADR 0002 treated Moxfield's unofficial API as the primary import path,
with manual plaintext export as a fallback for when the API broke. That
got the relationship backwards. Lucas builds and maintains decklists in
Moxfield because that's the tool he already uses — not because KotPod
should treat Moxfield as a live system it depends on. The actual intent
is simpler: export a decklist from Moxfield, import that export into
KotPod once, and from then on the deck is KotPod's own data. Card
enrichment (oracle text, images, legality) still comes from Scryfall
(ADR 0003) regardless of how the decklist itself arrived.

## Decision

Parse a Moxfield export file, provided manually by Lucas, as the sole
import path — no live calls to Moxfield at all. Corrected against a real
export Lucas provided: Moxfield's plain-text format is richer than
generic guides suggest — each line is
`<qty> <card name> (<set code>) <collector number>`, with an optional
trailing `*F*` (foil) or `*E*` (etched, best inference) marker, and
double-faced/modal cards appear as `Front Name / Back Name` on one line.
So plain text alone already carries exact-printing identity, not just
card name — CSV isn't needed for that. CSV remains available separately
for Moxfield's own category/tag data, which plain text does not carry;
worth adding only if that tag data turns out useful for ADR 0007's
decklist self-assessment. Commander designation survives Moxfield's own
exports, so the parser needs to specifically identify and separate it
rather than treat every line as a mainboard card, and needs to
explicitly exclude the maybeboard/"considering" pile if present.

## Alternatives considered

- **Moxfield's unofficial API**, called live per import — ADR 0002's
  original choice. Rejected on reflection: it makes KotPod depend on an
  undocumented, Cloudflare-protected, reverse-engineered endpoint for
  something that only ever needs to happen once per deck. The
  export-based approach sidesteps that entire risk category — no
  scraping, no bot-detection workaround, nothing that can silently
  change shape — and fits the project's broader instinct toward not
  taking on live dependencies that aren't earning their keep. Not closed
  off permanently: worth revisiting as a convenience layer later if
  manual export becomes genuinely tedious across many decks.
- **Scrape the rendered deck page directly.** Rejected — client-side
  rendering means the useful content isn't present in the initial HTML
  response (confirmed directly, ADR 0002).
- **Manual re-entry of decklists.** Rejected — error-prone and defeats
  the purpose of "import."

## Consequences

- Removes an item from NFR-2's internet-required list entirely, not just
  changes how it's satisfied — deck import needs no network access at
  all.
- The parser owns real format-handling work: locating the commander
  line(s), splitting `Front / Back` names for double-faced cards, and
  excluding maybeboard/considering cards if present. Set code +
  collector number, once parsed, resolve to an exact Scryfall printing
  via `/cards/collection` using `{set, collector_number}` identifiers —
  no separate CSV path needed just to get exact-printing/art data.
- Re-importing an updated decklist (Lucas edits the deck later) is a
  manual "export again, re-run the importer" step, not something the
  system detects or pulls on its own. Fine for a single-user hobby
  project; worth revisiting only if that manual step becomes real
  friction.
- ADR 0002's `moxfield_public_id`-as-live-reference framing doesn't
  carry forward — see the data model update.
