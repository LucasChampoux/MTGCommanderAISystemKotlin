# ADR 0007: Deck-Specific Agent Identity — Layered Context with Memory-Style Refinement

**Status:** Accepted
**Date:** 2026-08-18

## Context

The premise of dedicated per-deck agents (vs. one generalist agent) only
pays off if each agent's understanding of its deck is substantive, not
just "here are the legal moves." Two static sources of that
understanding: how the deck's pilot would explain it to someone else,
and what the decklist itself structurally implies. Neither captures what
actually happens at the table — which lines work, which don't, patterns
only visible in hindsight across multiple games.

## Decision

An agent's context is built from three layers:

1. **Preface** — a short, hand-written description of the deck's game
   plan, win conditions, and style. Authored once per deck, revised by
   hand, never auto-generated.
2. **Decklist self-assessment** — a structural read of the current
   decklist (curve, ramp/removal/draw density, apparent win conditions),
   regenerated automatically whenever the decklist changes. Depends on
   card data from the Scryfall cache (ADR 0003).
3. **Played-experience memory** — a periodically-synthesized summary
   drawn from the agent's own game history, using the reasoning log from
   ADR 0006 as raw material. Refreshed after a batch of games rather
   than after every single one, mirroring how Claude's own memory system
   periodically distills conversation history into durable, selectively
   applied memory rather than replaying raw transcripts each time.
   Synthesis itself is a Claude API call (ADR 0005), consistent with the
   rest of the agent's decision-making.

All three combine into what the agent actually knows about its deck at
decision time. Only layer 3 changes on its own.

## Alternatives considered

- **A single static document per deck.** Rejected — misses exactly the
  empirical, only-visible-in-hindsight patterns that motivated wanting
  persistent agent memory in the first place.
- **Re-derive everything from raw game history at decision time, no
  synthesis step.** Rejected — expensive per decision, and risks
  flooding the agent's context with noise instead of durable signal;
  loses the actual "memory" framing in favor of just a longer
  transcript.

## Consequences

- Introduces a new component: a per-deck agent identity/memory store,
  distinct from the Card Cache (ADR 0003) and the reasoning log
  (ADR 0006).
- This is the least-precedented piece of the design so far. Its scope
  belongs at M8 — M1 through M7 don't need any of it, and it shouldn't
  be allowed to creep earlier.
- Synthesis cadence (after how many games, on what trigger) is a
  parameter to tune once there's real game history to look at — not
  worth deciding from first principles now.
- Directly depends on ADR 0006's log existing before layer 3 can be
  built at all.
