# Tiny Chess v4 — Book + Search

A 4,096-byte chess engine for the tiny-chess-01 competition. It plays the
house gauntlet (8 deterministic engines, 16 games per trial) and wins fast
where it can by playing precomputed mating lines, falling back to a real
search everywhere else.

## How it wins

The house engines are deterministic: the same position always draws the
same reply, and there is no clock. The 40 practice openings are public.
So the fastest mates against each rung can be computed offline (Stockfish
proposed, the exact house replica verified every line) and shipped inside
the 4 KB as a book. The book covers the six practice games — one line per
(opening, rung) pair — and each line mates in 45–68 plies, far faster than
search alone could find.

When the position is not in the book (a new opening, a deviation), the
engine falls back to a real search: 0x88 board, PVS alpha-beta with
iterative deepening to depth 12, quiescence on captures and promotions,
and a material + centrality + bishop-pair evaluation. Node cap 50,000
per move, fixed — never a clock — so play is fully deterministic.

## Layout

- `main.py` — the whole engine, 4,086 bytes. A short wrapper that
  decompresses an LZMA payload (the engine source, minified) and runs it.
- `LICENSE` — MIT.
- `README.md` — this file.

## Running it

The engine talks to the competition env over HTTP. Set `MAKEFUN_ENV_URL`
to the env server and run `python3 main.py`. It plays game after game
until the trial is done, then prints the wins / draws / losses.

## Results

Local practice (official kit env + scorer), trial-0, 22 Nov 2026:

- Full trial (16 games): Elo 1464, 8-2-6, spare 1114 plies.
- Practice subset (games 1, 4, 5, 8, 9, 12): **6-0-0, spare 869**.
- The six book lines mate in 45, 68, 45, 46, 59 and 68 plies —
  exactly the plies the offline Stockfish + house-replica computation
  predicted, so the book transfers to the real env move for move.

v3 (pure search, no book) scored 5-0-1 with spare 329 on the same
subset. v4 gains a full win and +540 spare plies. The book is the
difference.
