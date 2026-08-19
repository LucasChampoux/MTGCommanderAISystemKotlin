# KotPod
*(working title)*

## What this is

A locally-run system for playing Magic: The Gathering Commander (EDH) games against AI-controlled opponents. Each opponent is a dedicated agent bound to one specific deck, imported from Moxfield, so an agent's decisions stay grounded in the strategy of the single deck it knows rather than generalizing across every deck in existence.

## Why

Commander is a social, multiplayer format, and assembling a consistent pod —time, people, a shared table — is the actual bottleneck to playing, not knowledge of the game. This project removes that bottleneck for practice and deck-testing, and eventually for the social/political side of the format (see Non-goals for what's explicitly deferred).

## Essentials — Phase 1

- Play a full Commander game, locally, against one AI-controlled opponent running one imported deck.

- Import a decklist from Moxfield by URL.

- Resolve card data (oracle text, mana cost, types, images) via the Scryfall API, cached locally — no full local card database.

- Enforce rules via Forge (see ADR 0001).

- Human player plays through Forge's existing GUI (ADR 0004); a purpose-built interface is a later-phase goal, not Phase 1.

- One agent, bound to one deck, controlling combat, targeting, and other decision points in place of Forge's default heuristic AI — backed by the Claude API (ADR 0005), drawing on a layered, evolving understanding of its own deck (ADR 0007).

- Agent reasoning is always logged, and shown live only in sessions explicitly flagged as testing (ADR 0006).

## Non-Goals — Phase 1

- A full 4-player pod / multiple simultaneous AI opponents. The architecture shouldn't preclude this later, but v1 targets 1v1.

- Table talk, negotiation, or political play between agents.

- Agent-to-agent social memory — reputation or grudges shaped by how other players/agents have behaved across sessions. Real, separate, later phase, deserving its own design doc once Phase 1 actually plays a correct game. (Distinct from ADR 0007's per-deck played-experience memory, which is a deck's own strategic self-knowledge, not social dynamics between players.)

- A custom-built interface. Forge's own GUI covers Phase 1; a purpose-built UI is a deliberate later-phase goal (ADR 0004).

- Networked or remote play. Local only.

## Constraints

- Kotlin-first. New code is Kotlin; the rules engine (Forge) is a JVM/Java dependency consumed via Kotlin/JVM interop, not rewritten.

- Local execution. Gameplay runs on-machine, no cloud dependency to play a game.

- Internet required, but only for reference and decision calls — Moxfield import, Scryfall lookups, and Claude API calls for agent decisions (ADR 0005). No requirement to keep a full offline card mirror.

## Components (current thinking — expected to drift)

| Component        | Responsibility                                              |
|-------------------|--------------------------------------------------------------|
| Deck Importer     | Pull a decklist from Moxfield, normalize card names/counts   |
| Card Cache        | Resolve and locally cache card data from Scryfall            |
| Game Engine       | Forge, run in-process, enforcing all MTG/Commander rules      |
| Agent Controller  | Per-deck `PlayerController` subclass routing decisions to an agent, logging reasoning per session |
| Agent Identity Store | Per-deck layered context (preface, decklist self-assessment, played-experience memeory) an agent draws on |

## Status

Design/documentation stage. No code written yet. See adr/ for the specific technical decisions made so far and the reasoning behind each, and requirements-roadmap.md for detailed requirements and the milestone sequence.
