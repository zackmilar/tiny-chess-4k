# Tiny Chess Engine — make.fun tiny-chess-01 entry

A 4 KB chess engine. Original code, MIT licensed.
`main.py` is 4,066 bytes (cap 4,096).

## How it works

- **Board:** 0x88, integer pieces (white +1..+6, black −1..−6).
  Moves are packed ints: `from | to<<7 | promo<<14`.
- **Search:** negamax + alpha-beta, iterative deepening, fixed
  **50,000 nodes/move** — never a wall clock, no unseeded randomness,
  so replays reproduce the exact same moves. Transposition table
  (dict, fixed-seed Zobrist), killer moves, MVV-LVA + promotion ordering.
- **Quiescence:** stand-pat if not in check; captures/promotions only;
  all evasions in check; mate if none.
- **Eval:** material (P100 N320 B330 R500 Q900) + formula piece-square
  tables + bishop-pair bonus (+30) + passed-pawn bonus + king
  middlegame/endgame taper. Fifty-move rule scores 0.
- **Protocol:** HTTP/JSON env at `MAKEFUN_ENV_URL` — POST `/game {}`,
  POST `/move {"game": n, "move": "<uci>"}`, POST `/resign`.
  Castling is the king move (e1g1); promotions use `q/r/b/n`.
- **Packaging:** the engine is lzma-compressed and base85-encoded into
  `main.py` (a compressed payload + unpacker is explicitly allowed;
  both count toward the 4,096 bytes).

## Results (practice, official scorer, real kit env)

trial-0 **1,809**, trial-1 **1,545**, trial-2 **1,451** — mean **1,602**.
Clears the 1,154 final floor by 448; beats the house baseline (~1,325)
by 277. Determinism confirmed (trial-0 run twice, move lists
byte-identical). ~22k nps on the test VM; mean 2.27 s per 50k-node move,
well under the 10 s/move limit.
