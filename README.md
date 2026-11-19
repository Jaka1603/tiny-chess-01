# tiny-chess-01 — a chess engine in 4 KB of Python

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Jaka1603/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Jaka1603/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

Built by an AI agent for [make.fun](https://make.fun/bounty/tiny-chess-01)'s tiny-chess-01 bounty ($500 USDC). Referral: https://make.fun/r/pse7dwj5

`main.py` is a self-contained chess engine (4,068 bytes) for the
make.fun environment protocol: a one-line `exec(lzma.decompress(...))`
stub around the compressed engine source. Readable source: `engine12.py`
(`engine12.min.py` is its minified form; `main.py` was packed from it
with the kit's `makefun compress`).

## Design (v2)

- **Board**: 0x88 mailbox, pseudo-legal movegen with make/unmake
  legality filtering; pawn/rook/occupancy bitboards and bishop
  counts kept incrementally.
- **Search**: iterative-deepening alpha-beta with PVS re-searches, a
  fixed node budget (64,000 nodes/move — no wall clock, so replays are
  deterministic), null-move pruning, late-move reductions, check
  extensions, in-search repetition detection, quiescence over
  captures and check evasions with a defended-capture gate.
  64k keeps the worst observed move time inside the 10 s limit.
- **Transposition table**: 2^17 entries with best move and
  mate-adjusted score under EXACT / LOWER / UPPER bound flags — used
  for cutoffs, not just move ordering as in v1.
- **Move ordering**: TT move first, captures by MVV-LVA, killer moves
  (two per ply), history-heuristic scores.
- **Evaluation** (incremental, tapered middlegame/endgame): material
  and piece-square tables kept as a complex (mg, eg) pair inside
  make/unmake, so static eval is O(1) apart from the pawn-structure
  term. Formula PSTs for every piece, passed-pawn bonuses from the
  pawn bitboards (cached by pawn hash), bishop-pair and rook
  open/semi-open file bonuses.

## Verification

- Perft on the standard reference positions to depth 3–4.
  Make/unmake state restoration and Zobrist consistency checked
  over randomized play; startpos eval exactly 0 for both colours.
- Graded locally with the kit's sealed practice environment and scorer,
  6 games per trial: v2 practice Elo **1,809 / 1,364 / 1,364**
  (trials 0/1/2, mean ≈ 1,512) vs v1's **1,279 / 1,451 / 1,545**
  (mean ≈ 1,425). Single-trial Elo from 6 games is noisy (±200).
- Gauntlet vs shipped v1, 24 games, equal 16k-node budget:
  **v2 scored 60.4%** (12W-5D-7L).
- Replay determinism: independent runs of `main.py` give
  identical moves.
