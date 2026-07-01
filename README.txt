pycdc-wasm — Corresponding Source mirror
==================================================

This repository publishes the Corresponding Source for the WebAssembly
build of pycdc (license: GPL-3.0-only) used in edgetools.io.

Contents
  build/      our build recipe: Dockerfile + helper scripts/config/patches.
              Rebuild with:  docker build build/
  upstream/   the exact upstream source archive(s) the build fetched,
              byte-identical and sha256-verified (see below).

Upstream sources:
  pycdc.tar.gz
    https://codeload.github.com/zrax/pycdc/tar.gz/b4289760970dbc399684f1e155ec6d1ea1cc787e
    sha256 d7d8c53d37a1a586097145b532fca02535db2f2f0999b3b1e910133c9941bf31
