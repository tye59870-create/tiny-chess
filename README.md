# tiny-chess

A chess engine in 4 KB of Python, built for make.fun's
"Build the strongest chess engine in 4 KB of Python" bounty.

`main.py` (3,963 bytes) is the whole engine. It plays every game through
the env at `MAKEFUN_ENV_URL`: `POST /game {}` serves the game in play,
`POST /move {game, move}` plays a move, and the loop ends when the env
answers status `"done"`.

## How it plays

- 0x88 board, negamax with alpha-beta and a transposition table
  (Zobrist keys, exact/lower/upper bounds).
- Quiescence search over captures, plus check evasions when in check.
- Null-move pruning (depth reduction 3, no phase guard).
- Move ordering: transposition-table move first, then MVV-LVA captures
  (most valuable victim, least valuable attacker), then the rest.
- Evaluation: PeSTO middlegame piece-square tables (quantized to eighths,
  base85-packed), used as a single score; no endgame tables, to fit
  the size cap — except the king table, which is the average of the
  middlegame and endgame king tables so the king still centralizes
  late in the game. Material values P=100 N=320 B=330 R=500 Q=900.
- Fixed node budget (60,000 nodes per move), never a clock: the same
  position always gets the same move, so a trial's replay matches.

## The size rule

- `main.py` is the whole engine: at most 4,096 bytes as committed,
  Python 3.13, standard library only. It reads, imports or execs no
  other file in the repository.
- Compression is allowed and counted: `main.py` unpacks and execs an
  LZMA payload (`lzma`, `base64`); the payload and the unpacker are
  both part of its 4,096 bytes.
- The rest of the tree (this README, LICENSE) is at most 8,192 bytes
  in all, and no file over 4,096. MIT licence, see LICENSE.

## Grade it locally

Download the competition's dev kit, unpack it, and from inside it run:

    makefun grade-local --kit . --bot <your repository> --driver docker

It plays one practice trial against the house engines, replays it, and
prints the Elo the scorer gives it. Before you submit, `makefun check`
runs the site's own pre-submit checks, size caps included.
