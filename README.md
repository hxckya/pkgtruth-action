# pkgtruth Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-pkgtruth-blue?logo=github)](https://github.com/marketplace/actions/pkgtruth)
[![release](https://img.shields.io/github/v/release/hxckya/pkgtruth-action)](https://github.com/hxckya/pkgtruth-action/releases)

**Block hallucinated and slopsquatted npm dependencies in pull requests.**

Coding agents invent package names. Attackers register the ones that repeat.
This action runs [pkgtruth](https://github.com/hxckya/pkgtruth) against your
`package.json` on every pull request, posts a sticky comment with the evidence,
and fails the check when something must not be installed.

```yaml
name: Dependency gate
on:
  pull_request:
    paths: ['package.json', '**/package.json']

permissions:
  contents: read
  pull-requests: write   # for the sticky comment; drop it and set comment: false if you only want the check

jobs:
  pkgtruth:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: hxckya/pkgtruth-action@v1
```

## What it catches

| Verdict | Meaning |
|---|---|
| `HALLUCINATED` | The name does not exist on npm. An agent made it up. |
| `DANGER` | npm purged this name for malware and left a `-security` placeholder; or it is deprecated; or it is a near-twin of a package with orders of magnitude more adoption. |
| `CAUTION` | New, tiny, no repository, or runs install scripts with low adoption. Not blocked by default. |
| `UNKNOWN` | The registry could not be reached. Never treated as safe. |

Live examples it blocks today: `crossenv` (purged, ~1,400 installs/week),
`supabase-js` (purged — a model dropped the scope from `@supabase/supabase-js`),
`types-node`. The weekly list is in
[SLOPSQUATS.md](https://github.com/hxckya/pkgtruth/blob/main/SLOPSQUATS.md).

## Inputs

| input | default | |
|---|---|---|
| `path` | `.` | Directory with `package.json`, or a path to one |
| `fail-on` | `danger` | `danger` fails on HALLUCINATED/DANGER; `caution` also fails on CAUTION and UNKNOWN |
| `comment` | `true` | Post/update a sticky PR comment (needs `pull-requests: write`) |
| `version` | `latest` | pkgtruth version |
| `node-version` | `22` | Node.js version |

## Outputs

`blocking`, `total`, `report` (path to the JSON).

## What it is not

It reads registry metadata only — it does not analyze package source, so a
legitimate-looking package carrying a payload still passes. It is not a
replacement for `npm audit`, Dependabot, or Socket; those answer "is this
trusted code vulnerable?". This answers the earlier question: should this
package be here at all?

A legitimate package that gets flagged is a bug. Please
[open an issue](https://github.com/hxckya/pkgtruth/issues) — that is the report
this project most wants.

## License

MIT © hxckya
