# TODO

- **Non-ASCII heading anchors** (point 10): `php_mdhtml_slugify()` is
  ASCII-only.
- **Windows build** (point 14): `config.w32` exists, but
  `vendor/build/stage.sh` only produces Unix `.a` archives; a Windows build
  needs its own MSVC staging step.
