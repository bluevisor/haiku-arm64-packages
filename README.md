# haiku-arm64-packages

A `pkgman` repository of extra packages for **Haiku arm64**. The arm64 nightly
only knows the ~25 HaikuPorts bootstrap packages baked into the image, and
upstream publishes no arm64 HaikuPorts index. These packages are cross-built by
[haiku-arm64-bootstrap](https://github.com/bluevisor/haiku-arm64-bootstrap)
(`scripts/container/build-core-tools.sh`, index from `make-package-repo.sh`),
following the HaikuPorts recipes.

```sh
pkgman add-repo https://bluevisor.github.io/haiku-arm64-packages
pkgman install diffutils patch tar gzip xz_utils zstd pkgconf
```

`pkgman` needs HTTPS for this, which stock Haiku arm64 does not have:
install the `openssl3` + revision-3 `haiku`/`haiku_devel` packages from the
[bootstrap release](https://github.com/bluevisor/haiku-arm64-bootstrap/releases/tag/v2026.10.22-hrev60290)
first ([guide step 11](https://github.com/bluevisor/haiku-arm64-bootstrap/blob/main/docs/guide.md#11-pkgman-over-https)).

| Package | Version | License | Notes |
|---|---|---|---|
| diffutils | 3.12 | GNU GPL v3 | diff, cmp, diff3, sdiff |
| patch | 2.8 | GNU GPL v3 | |
| tar | 1.35 | GNU GPL v3 | HaikuPorts `--replace-hardlinks` patch; `-z`/`-J`/`--zstd` |
| gzip | 1.15 | GNU GPL v3 | gunzip, zcat, zgrep, … |
| xz_utils | 5.8.4 | 0BSD, GNU GPL v2 (scripts) | xz, lzma, liblzma + headers |
| zstd | 1.5.7 | BSD (3-clause) / GNU GPL v2 | zstd CLI, libzstd + headers (zlib, lzma support) |
| pkgconf | 3.0.10 | ISC | `pkg-config` symlink; searches `develop/lib/pkgconfig` |

Built for hrev60290, arm64 only. Packages are zlib-compressed: the arm64
packagefs cannot read zstd-compressed packages. Libraries and headers ship in
the main package (there are no separate `_devel` packages).

Exact upstream sources are in [`sources/`](sources); the HaikuPorts patchsets
are in the bootstrap repo under `patches/haikuports/`.

Layout: `repo.info`, `repo` (binary index with each package's checksum),
`repo.sha256`, `packages/*.hpkg`, served by GitHub Pages.
