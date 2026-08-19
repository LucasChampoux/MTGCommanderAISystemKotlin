# ADR 0006: Agent Reasoning Visibility — Logged Always, Shown Only When Testing

**Status:** Accepted
**Date:** 2026-08-18

## Context

An agent backed by an LLM (ADR 0005) naturally produces some reasoning
alongside each decision. Whether to surface that during a game is a real
trade-off: visibility helps validate and debug the agent's decision
quality, but during ordinary play it lets the human player see what the
"opponent" is planning — undermining the point of playing against a
genuine opponent rather than a transparent one.

## Decision

Agent reasoning is always logged, regardless of session type. It is only
**displayed live** when a game session is explicitly flagged as
*testing* rather than ordinary play. Logging independent of display
means nothing is lost if something worth reviewing happens during
ordinary play — it just isn't surfaced in the moment.

## Alternatives considered

- **Always show reasoning.** Rejected — enables metagaming and removes
  the point of playing against an opponent rather than a script.
- **Never log reasoning at all.** Rejected — forecloses debugging and
  the testing use case entirely, and throws away the raw material ADR
  0007's played-experience memory depends on.
- **Show reasoning only in post-game review, never live, even when
  testing.** A reasonable middle ground, but rejected as the default —
  live visibility during an explicitly-flagged testing session is more
  useful for validating in-the-moment decisions than reconstructing them
  afterward. Nothing prevents building post-game review as well later.

## Consequences

- Requires a session-level "testing vs. ordinary" flag — a new, small
  piece of game-setup state.
- The always-on log is the raw material for ADR 0007's periodic
  played-experience synthesis — this decision isn't just about
  visibility, it's also what makes that later design possible.
- Log storage/rotation is an implementation detail to settle when it's
  actually being built, not a concern now.
