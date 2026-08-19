# ADR 0004: Human Interface — Reuse Forge's GUI for Phase 1

**Status:** Accepted
**Date:** 2026-08-18

## Context

The human player needs a way to actually play a game (FR-8). Building a
custom interface — battlefield, hand, stack, zones, drag-and-drop — is a
large effort that's orthogonal to what Phase 1 is actually trying to
prove: that the engine and agent pipeline work. Forge already ships a
working desktop GUI.

## Decision

Use Forge's existing desktop GUI for the human seat through Phase 1. A
custom interface remains a real goal, deliberately deferred to a later
phase once the engine/agent pipeline is proven out.

## Alternatives considered

- **Build a minimal custom UI now.** Rejected — unnecessary work for
  what Phase 1 needs to prove, and premature: better informed once
  there's an actual agent to build a UI around.
- **Command-line-only interaction.** Rejected — Forge already has a
  working graphical option; no reason to go backward to something less
  usable in the meantime.

## Consequences

- Saves significant Phase 1 effort; keeps focus on the engine/agent
  pipeline, which is the actually novel part of this project.
- Forge's GUI is a general MTG client, not tailored to this project.
  Agent-specific observability — reasoning visibility (ADR 0006) in
  particular — likely needs its own surface (console/log output) rather
  than integrating into Forge's own UI, which isn't built for that.
- Custom interface stays an explicit later-phase item, motivated by
  wanting a purpose-built experience rather than a generic MTG client —
  not dropped, just sequenced after the parts that are actually
  uncertain.
