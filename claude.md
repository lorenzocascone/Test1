# Claude Guidelines for Test1

## Project Overview
A single-file premium chess game (`chess.html`) — open it directly in any modern
browser, no build step or server required.

## Architecture
Everything lives in `chess.html`, organised as three sections:

1. **CSS + markup** — dark glassmorphism UI, four board themes via
   `body[data-board]`, responsive grid layout (desktop 3-column → mobile stack).
2. **`<script id="engine-code">`** — the chess engine (`createEngine()` factory):
   - 0x88 board representation with incremental make/unmake and Zobrist hashing
   - Negamax alpha-beta with transposition table, quiescence search, null-move
     pruning, killer/history move ordering, check extensions
   - PeSTO-style tapered evaluation (midgame/endgame piece-square tables)
   - Iterative deepening with time management; skill levels via softmax move
     selection over fully-scored root moves
   - This script's *text content* is also used to spawn two Web Workers
     (one for the engine opponent, one for live analysis/hints), so it must
     stay self-contained and free of DOM references
3. **UI script** — game state (same engine factory on the main thread),
   drag & drop + click-to-move, transform-based piece animation, history
   navigation, clocks, WebAudio sound synthesis, confetti, localStorage settings.

## Working with This Codebase
- The engine is validated with perft tests (standard positions, depths 1–4).
  If you touch move generation or make/unmake, re-run perft before shipping.
- The engine section must keep working in three contexts: main thread (window),
  Web Worker, and Node (`module.exports` guard) — that's how it gets tested.
- Moves are packed 28-bit ints (from | to<<7 | piece<<14 | captured<<18 |
  promo<<22 | flags); the encoding is shared between main thread and workers.
- Board squares are built once; per-move updates only touch CSS classes and
  piece transforms (keep it that way — no innerHTML churn in the hot path).

## Verification
- Syntax: `node --check` on extracted script sections.
- Engine: perft + search smoke tests in Node.
- UI: Playwright smoke test (click move, drag move, engine reply, modals,
  theme switch, mobile viewport).
