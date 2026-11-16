# tiny-chess-01 — a 4 KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/theyschool/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/theyschool/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A complete chess engine in a single `main.py` of 4,071 bytes (Python 3.13,
standard library only). It plays the make.fun gauntlet through the env API
(`MAKEFUN_ENV_URL`: `POST /game`, `/move`, `/resign`, UCI moves) and answers
every move inside the 10 s limit with fixed, deterministic budgets — no clock,
no randomness, so replays reproduce the same moves.

## Design

- **Board** — 0x88, 128 ints; moves packed into one int (from/to/promotion/
  flags plus an ordering score in the high bits, so ordering is a plain
  sort). Legal move generation with castling, en passant, promotion,
  check/checkmate/stalemate, repetition and the fifty-move rule
  (perft-verified on the standard positions).
- **Search** — iterative deepening with a transposition table (131,072
  entries, mate scores ply-adjusted), alpha-beta with quiescence over
  captures, promotions and check evasions, null-move pruning (R=2) and
  delta pruning in quiescence. Separate fixed budgets: 50,000 alpha-beta
  nodes and 60,000 quiescence nodes per move. The budgets are fixed node
  counts (never a clock), chosen so a move stays inside the 10 s limit even
  on a heavily loaded box.
- **Move ordering** — MVV-LVA capture scores (plus promotion bonus) encoded
  in the move int, root moves re-sorted by the previous iteration's scores.
- **Evaluation** — incremental (maintained in make/unmake, so it is O(1)):
  material plus formula piece-square tables, with a tapered king table that
  walks the king toward the centre in the endgame.
- **Packing** — the source is golfed to 7,327 bytes, then compressed with
  `lzma` (FORMAT_ALONE, preset 9) and base85-encoded into the 4,071-byte
  `main.py`, which unpacks and execs itself at startup.

## Grade it locally

From the competition kit directory:

    makefun grade-local --kit . --bot <path to this repo> --driver docker

(`--driver process` also works if the kit's env server is started by hand;
see the kit's RULES.txt.)
