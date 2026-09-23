# TODO

- **`{% plugin %}` inside inline code** (point 12): a `{% %}` inside a
  single-backtick code span is still treated as a live plugin invocation;
  only fenced blocks are protected.
- **Non-ASCII heading anchors** (point 10): `php_mdhtml_slugify()` is
  ASCII-only.
- **Windows build** (point 14): `config.w32` exists, but
  `vendor/build/stage.sh` only produces Unix `.a` archives; a Windows build
  needs its own MSVC staging step.
