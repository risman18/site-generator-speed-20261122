[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/risman18/site-generator-speed-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/risman18/site-generator-speed-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

# A faster drop-in for makefun-site 1.0.0

`run(input)` in `solve.js` returns exactly the output tree makefun-site 1.0.0's
`build` returns for the same site — every page's `index.html`, the index and
tag pages, `build-report.txt` and each static file, byte for byte — but faster.
It is a from-scratch reimplementation of makefun-site's own dialect (its front
matter, template language, Markdown subset, slug rule and report), written
against the reference program in the competition kit.

## How it is faster

- **Inline Markdown, batched:** the inline renderer scans for the next
  construct with one native regex pass and escapes whole plain runs at once,
  instead of escaping character by character.
- **Fused Markdown pass:** lexing and rendering run in one pass over the
  source's lines, with first-character gating before any regex, so no token
  objects are ever built.
- **Fast slugs and dates:** slugs take an ASCII fast path (NFKD is the
  identity on ASCII) and are memoised per case, as are long dates.
- **Cheap word counts:** reading minutes are counted with one manual pass
  over the body using a whitespace lookup table, with no word array allocated.
- **Lean templates:** the template tree is rendered with string
  concatenation, and each `for` reuses one scope object across its
  iterations instead of allocating one per element.
- **No wasted work:** a post's card is built once and reused for its page;
  tag slugs are computed once per name.

## Layout

- `solve.js` — the entry: a dependency-free ES module exporting `run(input)`.
- `main.ts` — the harness driver: reads every case from `/input/cases.json`
  (or `$MAKEFUN_INPUT_DIR`), answers each with `run`, and writes
  `/output/answers.json` (or `$MAKEFUN_OUTPUT_DIR`). It turns V8's
  incremental marking off before importing `solve.js`, as the house baseline
  does. Node 22 or later, nothing to install.

## Correctness

Checked against the kit's practice set (3 cases) and 12 further cases drawn
with the kit's own generator from fresh seeds: every answer byte-identical
to makefun-site 1.0.0's. Locally, `run()` over the practice set measures
about 4× the speed of the house baseline on this machine.

## Entry

Built for the make.fun bounty
[Build a static site faster, 22 November 2026](https://make.fun/bounty/site-generator-speed-01-run-20261122).
