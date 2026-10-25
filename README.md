# Engineering Notes for ricercar on Debian

*A technical notebook documenting the installation and DAC setup of the
ricercar bit-perfect music player.*

> **Understanding before copying.**

---

This repository documents how [ricercar](https://github.com/ricercar-player/ricercar)
was installed on a Debian 13 desktop and set up for bit-perfect playback
through two R2R DACs, including the problems found along the way, their
causes and their fixes.

## Reference Configuration

| Item | Value |
|------|-------|
| OS | Debian GNU/Linux 13 (trixie), amd64 |
| Sound server | PipeWire + WirePlumber |
| Player | ricercar 0.10.0-1 (`.deb` package) |
| DAC 1 | FiiO K11 R2R (USB) |
| DAC 2 | HiFiMAN EF499 (USB) |
| Headphones | Sennheiser HD 650 (EF499), Audio-Technica ATH-R50x (K11) |
| Speakers | K11 line out → AIYIMA T20 tube amplifier → TOMATE 2.0 soundbar |
| Streaming | Qobuz Studio Duo, through the unofficial Qobuz plugin |
| Qobuz app | Qobuzify (standalone, third-party) on the K11 |

## Current Technical Notes

| ID | Title | Status |
|----|-------|--------|
| TN-001 | [Installation](docs/TN-001-Installation.md) | Working |
| TN-002 | [DAC Setup](docs/TN-002-DAC_Setup.md) | Working |
| TN-003 | [Quick Reference](docs/TN-003-Quick_Reference.md) | Working |
| TN-004 | [Qobuz Streaming](docs/TN-004-Qobuz_Streaming.md) | Working |
| TN-005 | [Qobuzify](docs/TN-005-Qobuzify.md) | Working |

---

## Philosophy

Each Technical Note aims to:

- Explain the problem.
- Describe the assumptions.
- Present the solution.
- Discuss the limitations.
- Document future improvements.

> **Understanding always comes before copying.**

---

*Started 2026-10-24.*

---

MIT License — see [LICENSE](LICENSE).
