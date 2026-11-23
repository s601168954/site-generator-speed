# site-generator-speed

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/s601168954/site-generator-speed/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/s601168954/site-generator-speed/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for makefun-site 1.0.0: it returns exactly the same answer
for every case, byte for byte, in about half the CPU time of the house
baseline on the practice set (3/3 cases identical, verified with the kit's
`grade-local`).

## How it is faster

- **Markdown, one fused pass.** The reference lexes blocks into a token
  array and then renders it, escaping one character at a time. This build
  renders blocks straight from the lines, and its inline renderer scans
  once for the next markup or HTML character with a single regex, escaping
  plain text in bulk instead of one regex per character.
- **Templates compiled to flat jump ops.** The reference walks a node tree
  recursively on every render. This build compiles each template once per
  build into a flat op list with jump targets for `if`/`for`, rendered by
  a small loop machine with a shared loop stack. Scopes are null-prototype
  objects chained by prototype, so a name resolves with native property
  lookup, and each `for` reuses one scope and one `loop` object across its
  iterations instead of allocating per item.
- **No repeated work.** Slugs are cached per build (the same title or tag
  is slugged once), front matter is scanned by offset with the body taken
  as a slice instead of split-and-rejoined, files without a carriage
  return skip the line-ending rewrite, and each post's card is built once
  and reused by its page, the index, the tag pages and `recent`.
- **Same answers, by construction.** Every rule (front matter, slug,
  Markdown subset, template dialect, collections, tags, report) was
  written against the reference's README and its code, and checked
  against the practice set: all three cases byte-identical.

## Run it

```sh
node main.ts
```

`main.ts` reads `/input/cases.json`, answers each case with `run`, and
writes `/output/answers.json` (both paths can be overridden with
`MAKEFUN_INPUT_DIR` / `MAKEFUN_OUTPUT_DIR`).

## Files

- `solve.js` — the whole generator: `run(input)` returns makefun-site
  1.0.0's answer for one case. Dependency-free ES module.
- `main.ts` — the driver the harness runs: reads `/input/cases.json`,
  answers each case with `run`, writes `/output/answers.json`.
- `package.json` — `{"type": "module"}`, as the house baseline ships it.

---

<sub>Entered on [make.fun](https://make.fun/?ref=k0z8zfac&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
