# Status

## Current state (2026-09-23, version 0.1.4)

- Statically linked into `@kirigami/php-wasm` by `php-wasm-compiler`
  (`mdhtml: { mode: static }`), and `kirigami/php-prepros`'s `MD::toHtml()`
  delegates to it; the old pure-PHP renderer is kept there as `MD_LEGACY`.
- 0.1.4 made the emoji/plugin tables request-scoped again (point 16): the
  0.1.3 process-lifetime tables double-freed request-allocated Closures and
  crashed PHP-WASM after a few renders.
- Output diff-tested against the previous `MD::toHtml()` (point 11); the
  deliberate gaps are listed in [TODO.md](TODO.md).

## Build and verification history

**✅ First native build succeeds and produces correct output.** Built and
tested end-to-end natively (WSL Ubuntu 26.04, PHP 8.5.4, cc 15.2.0,
cmake 4.2.3 — no Emscripten/Docker involved, per point 6):

- `vendor/build/stage.sh` (not committed, regenerated on demand — see
  `.gitignore`): downloads `github/cmark-gfm` tag `0.29.0.gfm.13`, builds
  `libcmark-gfm_static` + `libcmark-gfm-extensions_static` via CMake
  (needed `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` — the upstream
  `CMakeLists.txt`'s own minimum predates CMake 4.x, which dropped that
  compatibility), and stages the two `.a` files plus the public headers
  into `vendor/libcmark-gfm/{include,lib}`.
- `config.m4` links against the staged `vendor/libcmark-gfm` via
  `PHP_ADD_INCLUDE`/`PHP_ADD_LIBRARY_WITH_PATH` (relative paths — an
  earlier attempt using `$ext_srcdir` failed the `test -f` guard, since
  that variable isn't set yet at the point `PHP_ARG_ENABLE` runs in a
  standalone `phpize` build; config.m4 already executes with cwd at the
  extension root, so plain relative paths are correct here).
- `md.c` now implements the real `MDHtml\Render()`: registers the GFM
  extensions once (`cmark_gfm_core_extensions_ensure_registered()`),
  attaches `table`/`strikethrough`/`autolink`/`tagfilter`/`tasklist` to a
  `cmark_parser_new(CMARK_OPT_UNSAFE | CMARK_OPT_FOOTNOTES)`, feeds it the
  input, and renders via `cmark_render_html()` — passing
  `cmark_parser_get_syntax_extensions(parser)` so extension node types
  (tables, tasklists, strikethrough) actually get rendered rather than
  silently dropped. `CMARK_OPT_UNSAFE` is deliberate: raw HTML passes
  through untouched for `MD::`'s own stricter whitelist sanitizer to
  handle afterward, matching what `MD::toHtml()` already does today.
  cmark's returned buffer is freed with plain `free()` (not `efree()`) —
  it came from the default libc-based allocator, since `cmark_parser_new()`
  (not `_with_mem()`) was used.
- Smoke-tested with a single Markdown sample covering every structural
  feature in scope (headings, bold/italic, links, nested lists, task
  list checkboxes, a table, strikethrough, an autolink, a blockquote, a
  fenced code block with a language class, and a footnote
  definition/reference): every feature rendered correctly, footnotes
  included (see point 2's correction — no PHP-side handling needed for
  those after all).

**✅ Round 2 (2026-09-12, points 9-11 above): renamed to php-mdhtml, moved
heading ids/`{#id}`/link+image attrs/task-list shape/raw-HTML whitelist/
emoji into the extension, diff-tested against `MD::toHtml()`, decided the
footnote-markup question.** `mdhtml.c` is ~1300 lines now. Verified with
both the feature smoke test (`vendor/build/test.php`) and the full diff
corpus (`vendor/build/diff-test.php`, both scratch files, not committed)
against a real checkout of `../kirigami`.

**✅ Round 3 (2026-09-12, point 12 above): the `{% plugin %}` system moved
into the extension.** `MDHtml\RegisterPlugin()`/`UnregisterPlugin()`/
`GetRegisteredPlugins()`, a raw-Markdown-string pre-scan calling
registered callbacks via `call_user_function()`, and a post-render
placeholder substitution with a paragraph-unwrap fix-up for block-style
plugins. Verified: inline usage, block usage (correctly unwrapped from
its `<p>`), an unregistered name (stays literal), register/unregister
round-trip, and the ```` ``` ```` fenced-code-block protection.

**✅ Round 4 (2026-09-12, point 13 above): `==highlight==`/`^sup^`/`~sub~`,
GFM alerts, and definition lists moved into the extension.** This closes
out point 10's entire "still PHP-side" migration list — every
`MD::toHtml()` post-processing step now has a C-side equivalent in
`mdhtml.c` (~1850 lines) except the two deliberate, documented gaps below.
Full diff-test corpus re-run, no regressions.

**✅ Round 5 (2026-09-17, point 15 above): fixed `emoji_custom`/`plugins`
being wrongly request-scoped**, found while starting the `kirigami`
integration below — verified in Docker that registrations now survive
across separate request lifecycles on the same process, matching
`MD::`'s real behavior. `v0.1.3`.
