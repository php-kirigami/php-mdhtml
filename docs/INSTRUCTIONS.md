# Instructions

## Building natively

```bash
bash vendor/build/stage.sh      # downloads and builds cmark-gfm into vendor/libcmark-gfm
phpize
./configure --with-mdhtml
make
php -n -d extension=modules/mdhtml.so your-test.php
```

`vendor/build/` also holds scratch test scripts (`test.php`,
`diff-test.php`, …), not committed. For a PHP 8.5 toolchain without
installing anything, see php-jsonk's `docs/INSTRUCTIONS.md` (extracted
`.deb` packages in WSL); cmark-gfm additionally needs CMake.

## Releasing

1. Bump `PHP_MDHTML_VERSION` in `php_mdhtml.h`.
2. Commit, then an annotated tag `vX.Y.Z`; push `main` with the tag.
3. `php-wasm-compiler` fetches tags by version (`matrix.json`); add the new
   tag there before the next PHP-WASM build.
