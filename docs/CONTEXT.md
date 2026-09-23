# Context

Part of the Kirigami ecosystem. Companion to `php-wasm-compiler`
(`../php-wasm-compiler`, github.com/php-kirigami/php-wasm-compiler), which
builds the `php.wasm` runtime and vendors third-party PHP extensions for it.

Purpose: replace the CommonMark/GFM *structural* parsing currently done by
hand-rolled regexes in `kirigami/packages/php-prepros/src/libraries/md.class.php`
(the `MD` class — see "Relationship to other repos" below) with a real, fast
C implementation, while keeping `MD::toHtml()`'s output identical.

This repo exists because of a specific dead end found in `php-wasm-compiler`
(2026-09-12, see its CLAUDE.md decisions 30-35 for the full investigation):
the only PECL extension providing CommonMark parsing to PHP,
`krakjoe/cmark`, is unmaintained since 2019, wraps plain `cmark` (no GFM
tables/tasklists/strikethrough/autolink), and hand-mimics PHP's internal
`zend_object` struct with hardcoded offsets — a pattern that silently
breaks (heap corruption, crash on object destruction) whenever PHP's real
`zend_object` layout changes, which it did between 2019 and PHP 8.5 (a new
`extra_flags` field). Patching that fork further would mean repeating the
same fragile hand-rolled-struct trick for every new GFM node type
(Table, TableCell, TaskItem, Strikethrough) we'd want to add. Writing a
small, purpose-built extension against `cmark-gfm` directly, with an
idiomatic (not hand-rolled) object model, was judged less total work and
much less fragile.

## Relationship to other repos

- **`php-wasm-compiler`** (github.com/php-kirigami/php-wasm-compiler,
  sibling checkout at `../php-wasm-compiler`): builds the core `php.wasm`
  (JSPI, Node-only) and vendors third-party PHP extensions for it. Its
  `cli.mjs compile-extension` subcommand is a fast, disposable smoke-test
  harness (compiles just one extension against already-built PHP headers,
  no full `php.wasm` rebuild) — used there to validate `krakjoe/cmark` and
  `sodium` during development, and the natural mechanism to validate
  php-mdhtml's WASM build too, once we get there (point 6). Its CLAUDE.md
  decisions 30-35 document the full `krakjoe/cmark` object-layout crash
  investigation that motivated this repo.
- **`kirigami`** (github.com/php-kirigami/kirigami, sibling checkout at
  `../kirigami`): the framework itself.
  `packages/php-prepros/src/libraries/md.class.php` is the `MD` class this
  extension accelerates — the single source of truth for exactly what
  output is expected. Any behavior change on either side must be checked
  against it. `packages/php-prepros/src/libraries/md.plugins.php` has the
  default plugins (`codepen`, `checklist`, `callout`, `img-asset`)
  registered through `MD::registerPlugin()` — unaffected by this
  extension (point 3).
