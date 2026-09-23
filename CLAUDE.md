# php-mdhtml

A PHP extension rendering CommonMark + GFM through `cmark-gfm`, plus every
`MD::toHtml()` extension (heading anchors, `{% plugin %}` shortcodes,
emoji, GFM alerts, definition lists, `==highlight==`/`^sup^`/`~sub~`).
Backs `kirigami/php-prepros`'s `MD` class; shipped statically in
`@kirigami/php-wasm`.

This file is the entry point only. Detailed, evolving content lives under
[docs/](docs/), split by topic:

- [docs/CONTEXT.md](docs/CONTEXT.md) — why this repo exists (the
  `krakjoe/cmark` dead end) and how it relates to `kirigami` and
  `php-wasm-compiler`.
- [docs/DECISIONS.md](docs/DECISIONS.md) — the numbered decision log
  ("points" 1-16); source comments cite points by number.
- [docs/STATUS.md](docs/STATUS.md) — current state and build history.
- [docs/TODO.md](docs/TODO.md) — known gaps and next actions.
- [docs/INSTRUCTIONS.md](docs/INSTRUCTIONS.md) — building and releasing.

## Core conventions

- **Language**: conversation with the user is in French; everything
  committed to this repo (code, comments, docs, commit messages) is in
  English.
- **Output parity**: `MD::toHtml()`'s output is the spec (point 3); check
  any behavior change against `../kirigami/packages/php-prepros`.
- **Request-scoped state**: plugin and emoji tables live per request
  (point 16); don't keep request-allocated zvals across requests.
- **Target**: PHP 8.5, native and Emscripten (`php-wasm-compiler`).
- **License**: GPL-2.0-or-later.
- Keep this file at or under ~200 lines. New durable content goes into
  the matching `docs/*.md` file.
