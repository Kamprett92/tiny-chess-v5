[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Kamprett92/tiny-chess-v5/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Kamprett92/tiny-chess-v5/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# Tiny Chess v5

A small alpha-beta chess engine in 4 KB of Python, written for the
make.fun tiny-chess-01 bounty: 16 games a trial against eight fixed
house engines, ranked by Elo.

## How it plays

`main.py` is the whole engine (3,935 bytes, an LZMA packing of the
readable `engine.py` source beside it). It talks to the trial env only:

- `POST /game {}` serves the game in play: FEN, legal moves, colour.
- `POST /move {"game": n, "move": "<uci>"}` plays its move.
- It exits when the env answers `{"status": "done"}`.

No model calls, no other network, no other file, no clock, no unseeded
randomness: the same position always gets the same move, so a trial's
replay matches its live pass move for move.

## The search

- 0x88 board, pseudo-legal move generation filtered by king safety,
  with castling, en passant, promotion and the fifty-move clock.
- Negamax alpha-beta with quiescence (captures and promotions, plus
  every reply while in check). A position with no legal move is scored
  as mate or stalemate even at the horizon, never stood pat on.
- Iterative deepening, depth 1 to 5, under one limit only: a fixed
  budget of 10,000 nodes a move. When an iteration would pass it, the
  best move of the last finished depth stands; a growth estimate stops
  deepening before a depth that would not finish, so a move takes
  about 1-2 s on one CPU, well inside the 10 s move limit.
- Static eval: material (P 100, N 320, B 330, R 500, Q 900) plus
  pawn advancement, centralisation for knights and bishops, rooks on
  the seventh, a bishop-pair bonus, king safety (pawn shield), and a tempo term.

## Checks done locally

- Move generation matches published perft counts: start position
  depths 1-3 (20, 400, 8902), Kiwipete to depth 3 (48, 2039, 97862),
  the en-passant pin position to depth 3 (14, 191, 2812), and the
  promotion position to depth 2 (44, 1486).
- Finds mate in 1 (Qf7-e8# and Qf7-g7# both mate in the test position),
  scored as mate already at depth 1.
- Move choice is deterministic: repeated runs return the same move.

## Layout

- `main.py` — the packed entrypoint (the only file the trial runs).
- `engine.py` — the readable source `main.py` was packed from.
- `LICENSE` — MIT. `.makefun/owner.json` — the payout wallet claim.
