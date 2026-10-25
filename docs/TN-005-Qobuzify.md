# TN-005 — Qobuzify

## Reference Configuration

| Item | Value |
|------|-------|
| App | Qobuzify standalone app (Linux), [matthewprince/qobuzify](https://github.com/matthewprince/qobuzify) |
| Version | 0.3.9 in the repository when this note was written (installed version not recorded) |
| Subscription | Qobuz Studio Duo |
| DAC | FiiO K11 R2R |
| Chain | K11 → AIYIMA T20 → TOMATE soundbar |
| Other player | ricercar on the HiFiMAN EF499 (TN-002, TN-004) |

---

## Purpose

Qobuzify is a third-party app, not made by Qobuz. It wraps the Qobuz web
player in Electron and adds themes and extensions (synced lyrics,
Last.fm scrobbling, a sleep timer, keyboard shortcuts and others). On
Linux it also bundles an `mpv` player for **bit-perfect output with
exclusive access to the DAC**. macOS and Windows builds play through the
web player instead.

It needs your own paid Qobuz subscription. Check its README for what it
modifies and for its own terms before relying on it.

---

## How it is used here

| App | DAC | Used for |
|---|---|---|
| Qobuzify | FiiO K11 R2R | Qobuz with the official catalogue UI, through the T20 to the soundbar |
| ricercar | HiFiMAN EF499 | Qobuz and Spotify plugins, bit-perfect to the HD 650 |
| VLC | whichever is the system output | local files |

Both Qobuzify's `mpv` and ricercar's `hw:` output take a DAC exclusively.
Only one of them can use a given DAC at a time. With one DAC each there
is no conflict. If one of them reports "busy" or stays silent, stop
playback in the other, then check who holds the DAC:

```sh
fuser -v /dev/snd/pcmC*D0p
```

---

## Problem: generic icon in the taskbar and menu

The packaged build ships an icon (`wrapper/build/icon.png`, a white "q"
on blue), but the window sets none of its own. Linux takes the icon from
a launcher (`.desktop` file). An AppImage installs no launcher, so the
desktop showed a generic icon.

### Solution

A launcher and a plain "q" icon of our own (an original drawing, not the
Qobuz logo), written to the home folder only:

```sh
set -e
APP=$(find ~ -maxdepth 3 -iname 'Qobuzify*.AppImage' 2>/dev/null | head -1)
[ -n "$APP" ] || { echo "No Qobuzify AppImage found under ~ (3 levels)."; exit 1; }
chmod +x "$APP"
mkdir -p ~/.local/share/icons/hicolor/scalable/apps ~/.local/share/applications
cat > ~/.local/share/icons/hicolor/scalable/apps/qobuzify.svg <<'SVG'
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256" width="256" height="256">
  <defs>
    <linearGradient id="bg" x1="0" y1="0" x2="0" y2="1">
      <stop offset="0" stop-color="#3a3f4b"/>
      <stop offset="1" stop-color="#1a1d24"/>
    </linearGradient>
  </defs>
  <rect width="256" height="256" rx="56" fill="url(#bg)"/>
  <circle cx="118" cy="112" r="48" fill="none" stroke="#ffffff" stroke-width="22"/>
  <rect x="155" y="64" width="22" height="136" fill="#ffffff"/>
</svg>
SVG
cat > ~/.local/share/applications/qobuzify.desktop <<DESK
[Desktop Entry]
Type=Application
Name=Qobuzify
Comment=Qobuz web player with Qobuzify themes + extensions
Exec="$APP" %U
Icon=qobuzify
Categories=AudioVideo;Audio;
StartupWMClass=Qobuzify
DESK
update-desktop-database ~/.local/share/applications 2>/dev/null || true
gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor 2>/dev/null || true
```

The icon appeared without restarting Qobuzify: the running window
already matched `StartupWMClass=Qobuzify`.

### Checks

- If the menu shows the icon but the taskbar does not, the window class
  differs. Run `xprop WM_CLASS`, click the Qobuzify window, and put that
  class on the `StartupWMClass=` line.
- If the AppImage moves (an update, a new folder), run the block again:
  it rewrites the launcher with the new path.

### Undo

Delete `~/.local/share/applications/qobuzify.desktop` and
`~/.local/share/icons/hicolor/scalable/apps/qobuzify.svg`.

---

## Assumptions

- Linux x86_64 with the standalone Qobuzify AppImage under the home
  folder (the block searches three levels deep).
- A paid Qobuz subscription.

## Limitations

- Third-party software that wraps the Qobuz web player; Qobuz may change
  the player and break it.
- The launcher is per user and does not follow the AppImage if it moves.

## Future Improvements

- Record the installed Qobuzify version here.
- Try its themes and extensions (Marketplace in the account menu).
