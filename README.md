[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Kamprett92/tiny-chess-v5/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Kamprett92/tiny-chess-v5/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# Tiny Chess v9

A small alpha-beta chess engine in 4 KB of Python for the make.fun
tiny-chess-01 bounty: 6 games a trial against six fixed house
engines, ranked by Elo.

Version 9 keeps the version 7 search exactly and raises its fixed
node budget. Version 7 scored 1,099 on the platform, under the
1,154 final floor; version 8 tried a transposition table,
principal variation search and check extensions and scored worse
in local trials, so version 9 drops those and bets on search
depth alone.

## How it plays

`main.py` is the whole engine (3,384 bytes, an LZMA packing of
`engine.py`). It talks to the trial env only (`POST /game`,
`POST /move`) and exits on `{"status": "done"}`.

No model calls, no other network or file, no clock, no
randomness: same position, same move.

## The search

- 0x88 board, pseudo-legal generation filtered by king safety,
  with castling, en passant, promotion and the fifty-move clock.
- Negamax alpha-beta with quiescence (captures, promotions, and
  every reply while in check). No legal move scores as mate or
  stalemate even at the horizon, never stood pat on.
- Ordering: captures by MVV-LVA, promotions, then two killer
  moves per ply and a history table, reset at every move.
- Null-move pruning (R = 2, depth 3+, never in check).
- Iterative deepening, depth 1 to 8, under one limit: 15,000
  nodes a move. The best move of the last finished depth
  stands; a growth estimate stops deepening before a depth
  that would not finish. About 1-2 s a move, inside the 10 s limit.
- Eval: material (P 100, N 320, B 330, R 500, Q 900) plus
  piece-square tables (one 32-char string per piece, mirrored,
  5 cp/step), passed-pawn bonuses, bishop-pair bonus, king
  safety (pawn shield), tempo.

## Checks done locally

- Move generation matches published perft counts: startpos
  1-3 (20, 400, 8902), Kiwipete to 3 (48, 2039, 97862),
  en-passant pin to 3 (14, 191, 2812), promotion to 2 (44, 1486).
- Finds mate in 1 (Qf7-e8#), scored as mate already at depth 1.
- Move choice is deterministic: repeated runs return the same move.
- Local practice trials (kit level 31, 18 games): 4 won, 5 drawn,
  9 lost — 6.5 points, up from 4.5 for version 7 over the same
  18 games. Trial-0: 1-3-2 (beat depth-4 as white); trial-1:
  1-2-3; trial-2: 2-0-4 (beat depth-2 and depth-3).

## Layout

- `main.py` — the packed entrypoint (the only file the trial runs).
- `engine.py` — the readable source `main.py` was packed from.
- `LICENSE` — MIT. `.makefun/owner.json` — the payout wallet claim.
