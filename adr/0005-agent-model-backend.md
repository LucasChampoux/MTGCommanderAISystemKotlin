# ADR 0005: Agent Model Backend — Claude API

**Status:** Accepted
**Date:** 2026-08-18

## Context

Once an agent overrides a Forge decision point (starting at M6), that
override needs to be backed by something that can actually make a
judgment call — an LLM. The realistic options are a hosted API or a
locally-run model. A local model would avoid both cost and the network
dependency for decision calls, but requires hardware capable of running
a genuinely useful model, which isn't currently available.

## Decision

Use the Claude API as the model backend for agent decision-making.

## Alternatives considered

- **Local models (e.g., via Ollama).** The preferred long-term direction
  on cost and independence grounds, but not adopted now — current
  hardware isn't there yet. Not rejected in principle; worth revisiting
  if that changes.
  
## Consequences

- Real, ongoing per-call cost, unlike a free local model — worth
  watching as decision coverage expands from one override (M6) to
  several (M7).
- A hard network dependency for any agent-driven decision, which NFR-2
  already anticipated in general terms; this makes it concrete.
- API credentials need standard hygiene — environment variable or local
  secrets file, never hardcoded or committed.
- Latency per decision becomes a real gameplay factor once more than one
  decision point is agent-driven; worth watching from M7 onward.
- Not a one-way door: a `PlayerController` override calls out to
  "whatever makes this decision" and doesn't care what's on the other
  end. Swapping to a local backend later, if hardware allows, is an
  implementation change behind the same interface, not an architecture
  change.
