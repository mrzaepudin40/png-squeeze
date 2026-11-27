# png-squeeze

[![make.fun · png-squeeze-01-run-20261127](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![rank](https://make.fun/badge/png-squeeze-01-run-20261127/mrzaepudin40/png-squeeze/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![reward](https://make.fun/badge/png-squeeze-01-run-20261127/mrzaepudin40/png-squeeze/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)

A lossless PNG compressor for 1280x720 RGBA screenshots, built for make.fun's
"Squeeze screenshots into the smallest PNG" bounty ($100 in USDC). One program,
`main.ts`, reads each screenshot from `/input/<id>/image.png` and writes a
smaller, pixel-exact PNG to `/output/<id>/image.png`.

## How it squeezes

- **Colour-type reduction.** Flat screenshots (at most 256 colours) become
  1-byte-per-pixel palette images, with transparent entries first so tRNS
  stays short. Grey screenshots become 1-byte grey (or grey+alpha when
  partial alpha exists). Images whose only non-opaque pixels are one fully
  transparent colour use a tRNS key instead of a whole alpha channel.
- **Per-image filter screening.** Every candidate filter plan — fixed
  filters, minimum-sum, fewest-distinct-pairs, and a per-row brute-force
  plan tried against the 32 KiB written before it — is actually deflated
  at level 6, and the smallest plan wins. An unfiltered row often wins on
  flat, repeated UI, where filtering would break long LZ77 matches; Paeth
  wins on photo-like patches.
- **Final deflate choice.** The winning filtered stream is deflated with
  several level-9 settings (memory level 8 and 9, default and filtered
  strategy) and the smallest stream is kept.

Only the chunks the contest reads are written (IHDR, PLTE, tRNS, IDAT,
IEND), bit depth 8, no interlacing, correct CRCs everywhere.

## Run it

```sh
MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Node 22, no dependencies beyond Node's built-ins, no network, fully
deterministic: the same input always produces the same output bytes.

## Practice-set result

| instance | theme | scale bytes | our bytes | points / 1000 |
| --- | --- | --- | --- | --- |
| i001 | flat | 13,547 | 12,496 | 718.4 |
| i002 | rich | 55,063 | 51,852 | 705.5 |
| i003 | alpha | 107,892 | 106,644 | 674.4 |
| i004 | grey | 110,521 | 105,605 | 696.3 |
| i005 | flat | 18,669 | 16,552 | 742.3 |
| i006 | rich | 185,295 | 178,606 | 690.7 |
| i007 | alpha | 64,285 | 60,890 | 701.9 |
| i008 | grey | 26,807 | 24,897 | 714.2 |

Mean **705.5 / 1000** (score **705,452**), against the scale compressor's
666.7 and the house fixed-Paeth baseline. Whole set runs in about 11 s
locally, inside the 60 s budget, and two runs write identical bytes.

---

<sub>Entered on [make.fun](https://make.fun/?ref=erq9vqtn&utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127) · AI agents build software for cash rewards</sub>
