# ADR 0001: Rules Engine — Forge, driven via a custom `PlayerController`

**Status:** Accepted
**Date:** 2026-08-18

## Context

Commander needs a full Magic: The Gathering rules implementation: the
stack, priority passing, the layers system for continuous effects,
replacement effects, triggered abilities, and so on. Writing this from
scratch is a genuinely massive undertaking — it's one of the hardest parts
of any MTG software project. We also need a way for an external
decision-maker (an agent, eventually LLM-backed) to control a seat instead
of a human or the engine's own AI.

## Decision

Use **Forge** (open-source, Java/JVM, GPLv3, actively maintained, 99%+ card
coverage, explicit Commander support) as the rules engine, run in-process
and driven from Kotlin via JVM interop.

Forge separates game state (`Player`) from decision-making
(`PlayerController`, an abstract class of 100+ decision-point methods —
`declareAttackers`, `chooseSpellAbilityToPlay`, `chooseTargetsFor`,
`mulliganKeepHand`, `vote`, etc.). Its built-in heuristic AI
(`PlayerControllerAi`) is a concrete, non-`final` implementation of that
class. Our agents subclass `PlayerControllerAi` directly, inherit its
heuristics for free, and override only the decision points where an
agent's judgment should apply.

## Alternatives considered

- **Build a custom Kotlin engine, scoped to only the cards in our own
  decks.** Attractive early on, since starting with one deck bounds the
  card-implementation surface to roughly 100 cards instead of all of Magic.
  Rejected once we confirmed Forge's `PlayerController` split gives us the
  decision-hook we actually needed — the thing that made custom-build
  attractive was avoiding a fight with someone else's internals, and that
  fight turns out not to be necessary.
- **Extend the abstract `PlayerController` directly**, the way Forge's own
  test suite does (`PlayerControllerForTests`), reusing AI logic via
  composition. Rejected in favor of subclassing the concrete
  `PlayerControllerAi` — same end result, less code, since we inherit
  rather than manually re-wire each fallback decision.

## Consequences

- We inherit Forge's full card-rules coverage instead of re-implementing
  it — this is the whole point.
- Adds a JVM/Java dependency inside an otherwise-Kotlin codebase. Mitigated
  by full Kotlin/Java interop; no separate process or IPC required.
- Forge's own code comments note `PlayerController` implementations are
  "theoretically capable of making illegal choices" that the core engine
  won't always catch. Our controller must only ever return choices drawn
  from the exact legal-options collection Forge provides at each decision
  point — never a freely generated choice.
- We're depending on a third-party open-source project's internal API
  (`PlayerController`'s method surface) staying reasonably stable across
  Forge versions. Some maintenance burden on Forge upgrades should be
  expected.
- **Forge is GPLv3.** For personal, local use this isn't a practical
  concern, but if any part of this project is ever distributed, published,
  or its source shared, GPLv3's copyleft terms are worth a real review
  before doing so.
