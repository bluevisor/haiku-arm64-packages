# haiku-arm64-packages

A `pkgman` repository of extra packages for **Haiku arm64**. The arm64 nightly
only knows the ~25 HaikuPorts bootstrap packages baked into the image, and
upstream publishes no arm64 HaikuPorts index. These packages are cross-built by
[haiku-arm64-bootstrap](https://github.com/bluevisor/haiku-arm64-bootstrap)
(`scripts/container/build-{core-tools,dev-tools,stack}.sh`, Perl natively,
index from `make-package-repo.sh`), following the HaikuPorts recipes.

```sh
pkgman add-repo https://bluevisor.github.io/haiku-arm64-packages
pkgman install ncurses6                            # fixes the bootstrap terminfo path
pkgman install diffutils patch tar gzip xz_utils zstd pkgconf
pkgman install bzip2 zip unzip nano ninja cmake    # cmake pulls in libiconv
pkgman install curl git openssh python3.12 perl   # + openssl3, nghttp2, sqlite, ca_root_certificates
```

Replacing a hand-installed `non-packaged` copy of curl/git/OpenSSH/Python/Perl?
Install these first, then move the old files away: they shadow the packages
(see [guide step 13](https://github.com/bluevisor/haiku-arm64-bootstrap/blob/main/docs/guide.md#13-from-non-packaged-to-packages)).

**Install `ncurses6` from here first** (`pkgman install ncurses6`, version
`6.6_release`). It replaces the HaikuPorts arm64 bootstrap `ncurses6`, whose
built-in terminfo path is `/packages/ncurses6_bootstrap-6.6-1/.self/data/terminfo`.
That package mounts as `ncurses6-6.6_bootstrap-1`, so nano, `clear` and Python
curses fail with `ncurses: cannot initialize terminal type`. The version is
`6.6_release` because Haiku's version order puts `6.6` below `6.6_bootstrap`.

`pkgman` needs HTTPS for this, which stock Haiku arm64 does not have:
install the `openssl3` + revision-4 `haiku`/`haiku_devel` packages from the
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
| bzip2 | 1.0.8 | bzip2 | bzip2, bzcat, bzgrep, libbz2 + headers; `tar -j` |
| zip | 3.0 | Info-ZIP | stores Haiku file attributes; `-Z bzip2` |
| unzip | 6.10c23 | Info-ZIP | Info-ZIP beta, as HaikuPorts ships; restores attributes; no iconv mapping |
| nano | 9.1 | GNU GPL v3 | needs our `ncurses6`, see above |
| ninja | 1.13.2 | Apache v2 | |
| cmake | 4.4.4 | CMake (BSD 3-clause) | ctest, cpack; bundled curl/libarchive/libuv; HTTPS downloads use `CARootCertificates.pem` |
| libiconv | 1.18 | GNU LGPL v2.1, GNU GPL v3 (iconv tool) | Haiku's libroot has no iconv |
| ca_root_certificates | 2026_10_20 | MPL v2.0 | curl.se Mozilla bundle as `data/ssl/CARootCertificates.pem`; zlib (HaikuPorts' is zstd) |
| openssl3 | 3.5.9 rev 2 | Apache v2 | HaikuPorts patchset; `OPENSSLDIR=/boot/system/data/ssl`, default CA file `CARootCertificates.pem`; replaces the release's rev 1 |
| nghttp2 | 1.63.0 | MIT | library only |
| curl | 8.23.0 | Curl | OpenSSL, HTTP/2, zlib; replaces the bootstrap curl (no TLS) |
| git | 2.56.0 | GNU GPL v2 | HTTPS; no Perl/Python/Tcl extras |
| openssh | 10.6p1 | BSD | with OpenSSL (RSA/ECDSA); creates the `sshd` privsep user on install; not autostarted |
| sqlite | 3.53.4.0 | Public Domain | lib + `sqlite3` shell |
| python3.12 | 3.12.15 | Python | ssl, ctypes, curses, sqlite3, bz2, lzma; `python3 -m ensurepip` → non-packaged |
| perl | 5.42.2 | GNU GPL v1 / Artistic | threads, shared libperl; built natively |
| ncurses6 | 6.6_release | MIT | replaces the bootstrap ncurses6 (wrong terminfo path); terminfo database in `data/terminfo` |

Built for hrev60290, arm64 only. Packages are zlib-compressed: the arm64
packagefs cannot read zstd-compressed packages. Libraries and headers ship in
the main package (there are no separate `_devel` packages).

Exact upstream sources are in [`sources/`](sources), except openssl, curl,
git, openssh, Python and Perl, whose identical tarballs are `sources-*` assets of the
[bootstrap release](https://github.com/bluevisor/haiku-arm64-bootstrap/releases/tag/v2026.10.22-hrev60290).
The HaikuPorts patchsets
are in the bootstrap repo under `patches/haikuports/`, plus `patches/cmake-4.4.4-*.patch`.

Layout: `repo.info`, `repo` (binary index with each package's checksum),
`repo.sha256`, `packages/*.hpkg`, and an `index.html` package list, served by
GitHub Pages.
