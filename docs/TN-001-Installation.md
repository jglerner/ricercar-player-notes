# TN-001 — Installation

## Reference Configuration

| Item | Value |
|------|-------|
| OS | Debian GNU/Linux 13 (trixie) |
| Architecture | amd64 (x86_64) |
| Method | `.deb` package from the ricercar releases page |

---

## Problem

The first install attempt failed with:

```
ricercar: Depends: libasound2 (>= 1.0.16) but it is not installable
          Depends: libc6 (>= 2.34) but it is not installable
          Depends: libc6 (>= 2.35) but it is not installable
          Depends: libfontconfig1 (>= 2.12.6) but it is not installable
          Depends: libxkbcommon0 but it is not installable
```

## Cause

`libc6` is the core C library and is present on every Debian system.
When apt calls it "not installable", the package is almost always built
for a different CPU architecture than the system.

```sh
dpkg --print-architecture            # amd64  (the PC)
dpkg-deb -f ricercar_*.deb Architecture   # arm64  (the download)
```

The `arm64` package (for ARM boards) had been downloaded on an `amd64` PC.
apt looked for the arm64 versions of every library and found none. The
other four errors were side effects of the same mismatch.

## Solution

1. Delete the wrong package.
2. Download the file ending in `_amd64.deb` from
   <https://github.com/ricercar-player/ricercar/releases/latest\>.
3. Check it, then install it. The `./` tells apt it is a local file.

```sh
dpkg-deb -f ricercar_*_amd64.deb Architecture   # must print amd64
sudo apt update
sudo apt install ./ricercar_*_amd64.deb
```

## Assumptions

- Debian 12+ or Ubuntu 22.04+. The packages need glibc 2.35 or newer.
- On trixie, `libasound2` is provided by `libasound2t64`; apt accepts it.

## Limitations

- Packages exist only for x86_64 and aarch64. A 32-bit ARM system
  (`armhf`) has no ready-made package.

## Future Improvements

- Verify downloads with the signed `SHA256SUMS` and `minisign`, as
  described in the ricercar README.
