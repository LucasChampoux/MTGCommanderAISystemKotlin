# Requirements & Roadmap

This document translates the [charter](README.md) and [ADRs](adr/) into
concrete, checkable requirements, plus a milestone sequence for tracking
progress. Hobby project, no dates — each milestone is done or it isn't;
the order reflects dependency and "prove the risky thing early," not a
schedule.

## Functional Requirements

### Deck import
- **FR-1** — Import a full Commander decklist, including the designated
  commander(s), from a manually-exported Moxfield decklist.
  *(ADR 0008)*
- **FR-2** — Correctly identify the designated commander(s) during
  import regardless of which Moxfield export format (plain text or CSV)
  is used. *(ADR 0008)*
- **FR-3** — Persist an imported deck locally so it doesn't need
  re-importing every session.

### Card data
- **FR-4** — Resolve card data (oracle text, mana cost, type line,
  image) for every card in an imported deck via the Scryfall API,
  matching the exact printing (set + collector number) captured at
  import. *(ADR 0003, ADR 0008)*
- **FR-5** — Cache resolved card data locally, keyed by Oracle ID; only
  fetch cards not already cached. *(ADR 0003)*
- **FR-6** — Support refreshing a cached card's data on demand, for
  errata or ban-list changes.

### Game engine
- **FR-7** — Run a full Commander-legal game between the human player
  and one AI-controlled opponent, with rules enforced by Forge.
  *(ADR 0001)*
- **FR-8** — Provide the human player a way to play via Forge's existing
  desktop GUI in Phase 1; a custom interface is deferred to a later
  phase. *(ADR 0004)*

### Agents
- **FR-9** — Bind exactly one agent to exactly one imported deck; an
  agent never plays a deck other than the one it's bound to.
- **FR-10** — An agent can override Forge's default decisions for at
  least: attack declarations, block declarations, spell/ability
  targeting, and mulligan decisions.
- **FR-11** — Every agent decision is constrained to Forge's actual
  legal-options collection for that decision point — never a freely
  generated choice. *(ADR 0001)*
- **FR-12** — Adding a new deck + agent pairing doesn't require
  modifying the code or config of any existing deck/agent.
- **FR-13** — Support flagging a game session as *testing*. Agent
  reasoning is always logged; it is only displayed live when the session
  is flagged as testing. *(ADR 0006)*
- **FR-14** — Construct each agent's context from three layers: a
  hand-written preface, a decklist-derived self-assessment, and a
  periodically-synthesized played-experience memory. *(ADR 0007)*

## Non-Functional Requirements

- **NFR-1 — Local-first.** Game logic and state run on the local
  machine; the game is playable without any cloud service hosting game
  state.
- **NFR-2 — Scoped network dependency.** Internet is required for
  Scryfall lookups on a cache miss and Claude API calls for agent
  decisions from M6 onward. Deck import (ADR 0008) and the game loop
  itself need no network access. *(ADR 0005)*
- **NFR-3 — No persistent full card mirror.** No local copy of the
  entire MTG card database; only cards actually seen get cached.
  *(ADR 0003)*
- **NFR-4 — Kotlin-first.** All original code is Kotlin. Third-party
  dependencies (Forge) may be JVM/Java, consumed via interop rather than
  rewritten. *(ADR 0001)*
- **NFR-5 — Correctness over performance.** Rules accuracy is inherited
  from Forge and shouldn't be traded away for speed; performance work is
  deferred until something is actually slow.
- **NFR-6 — Extensible by construction.** New decks/agents are additive,
  never require touching prior ones. Same requirement as FR-12, called
  out here as a quality attribute of the architecture, not just a
  feature.

## Roadmap — Milestones

No dates. Each milestone has a concrete "done when," and later ones
assume earlier ones are genuinely done, not approximately done.

**M1 — Toolchain proof.** Forge builds and runs from a Kotlin/Gradle
project; a trivial Kotlin call into Forge's Java API succeeds. *Done
when:* a Kotlin `main()` can instantiate a Forge `Game` object without
errors.

**M2 — First game, no custom code.** Play one full Commander game,
human vs. Forge's own built-in AI, using Forge's existing GUI and a deck
built directly in Forge's own format. *Done when:* a full game reaches a
win/loss with Lucas's actual commander deck, no import pipeline involved
yet. *(ADR 0004)*

**M3 — Moxfield import.** Deck Importer turns an exported Moxfield
decklist into a Forge-loadable deck. *Done when:* the same deck from M2
loads via the exported decklist instead of manual entry.
*(FR-1, FR-2, FR-3, ADR 0008)*

**M4 — Scryfall card cache.** Card data for that deck resolves via
Scryfall and caches locally. *Done when:* card data for the full deck is
available offline after one successful online resolution.
*(FR-4, FR-5, FR-6)*

**M5 — Walking-skeleton controller.** `ClaudeBackedController extends
PlayerControllerAi` is wired in for the AI seat, overriding nothing (or
one no-op logging override) — plays identically to M2, just through the
new controller. *Done when:* a full game completes with the custom
controller active, indistinguishable from stock `PlayerControllerAi`.
Proves the interception point from ADR 0001 before any intelligence is
added.

**M6 — First real agent decision.** Override `declareAttackers` with an
actual Claude API call, constrained to Forge's legal-attacker list.
*Done when:* a full game completes where attack decisions visibly come
from the model, everything else unchanged; decisions are logged, with
live display available when the session is flagged as testing.
*(FR-10, FR-11, FR-13, ADR 0005, ADR 0006)*

**M7 — Expand decision coverage.** Extend model-backed decisions to
blocks, targeting, and mulligan. *Done when:* all four decision types in
FR-10 are agent-driven for one full game.

**M8 — Deck-specific agent identity.** Agent context is built from all
three layers defined in ADR 0007. *Done when:* an agent plays using the
preface, decklist self-assessment, and at least one round of
played-experience synthesis from prior games — attack/block choices are
visibly different, and defensibly better, than with legal-move-only
context. *(FR-14, ADR 0007)*

**M9 — Second deck, second agent.** Onboard a second Moxfield deck with
its own independent agent. *Done when:* both agents can play in the same
session without either touching the other's code or config.
*(FR-9, FR-12, NFR-6)*

**Phase 2 (not yet scoped)** — full 4-player pod, table talk /
negotiation between agents, cross-session memory and reputation across
players. Explicit non-goals of Phase 1 per the charter; worth its own
requirements doc once M9 is real.
