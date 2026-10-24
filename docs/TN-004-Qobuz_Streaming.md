# TN-004 — Qobuz Streaming

## Reference Configuration

| Item | Value |
|------|-------|
| Player | ricercar 0.10.0-1 |
| Plugin | Qobuz (unofficial), `ricercar-qconnect` 0.2.1 by anon7627 |
| Subscription | Qobuz Studio Duo (hi-res up to 24/192) |
| DAC | FiiO K11 R2R (`hw:2,0`), or HiFiMAN EF499 (`hw:5,0`) |
| Local music | Played in VLC, not in ricercar |

---

## Purpose

ricercar has no streaming service built in. Qobuz reaches it through a
**source plugin**: a separate program, installed from the community hub,
that signs in to Qobuz, browses the catalogue and hands ricercar a stream
URL for each track. ricercar reads the stream itself and sends it to the
DAC untouched.

This note covers the plugin setup, how to confirm that what plays is
really Qobuz in hi-res, and the two problems found on the way.

---

## Setup

1. **Settings → Setup assistant → Plugins** (or the Plugins page): turn
   on **Qobuz (unofficial)**.
2. Sign in. The official Qobuz page opens in the browser; the password
   goes to Qobuz, not to the plugin. When the page says the tab can be
   closed, the plugin shows **Signed in**.
3. **Settings → Library**: remove `~/Music` (see Problem 1).
4. **Settings → Audio output**: select the DAC's `hw:` entry, then
   **Refresh**. No "busy" message may show under it (see Problem 2).

The Spotify plugin (unofficial, librespot, Ogg Vorbis 320 kbit/s, needs
Premium) is set up the same way. It is lossy; Qobuz is the main source.

### Quality

The plugin picks the quality from ricercar's selected output:

| Output accepts | Qobuz quality requested |
|---|---|
| 176.4 kHz or more | up to 24/192 |
| 88.2 / 96 kHz | up to 24/96 |
| less, or 16-bit only | CD (16/44.1) |

Both DACs accept 192 kHz, so both get the full Studio quality. The
subscription is the other limit. Select a `hw:` output, not `pipewire`.

---

## Verification

Play a Qobuz album and read the chain line under the seek bar:

```
● Qobuz  >  FLAC 24/96  >  S24_3LE  >  FiiO K11 R2R
```

| Item | Meaning |
|---|---|
| Green dot | bit-perfect, nothing alters the samples |
| Qobuz | the source is the stream, not a `file:///` path |
| FLAC 24/96 | the album's own resolution |
| S24_3LE | 24-bit container the DAC accepts natively |
| FiiO K11 R2R | the `hw:` output |

Album covers in the library carry **QOBUZ** and **HI-RES** badges.

---

## Problem 1: a "Qobuz" playlist was a local copy

The playlist "O som da orquestra" played from
`~/Music/Playlists/O som da orquestra/*.mp3`, not from Qobuz, and failed:

```
ERROR symphonia_core::formats::probe: reached probe limit of 1048576 bytes
WARN  ricercar_audio::player: file:///…/105 - Der Wildschütz_ Overture.mp3:
      decode error: probe …: unsupported feature: core (probe):
      no suitable format reader found
```

### Cause

Two things together:

- ricercar indexed `~/Music`, so local playlists appeared next to the
  Qobuz ones and looked the same.
- These MP3s (320 kbit/s, 44.1 kHz, valid: `file` and `ffprobe` read
  them) carry a **3000×3000 cover (about 2.5 MB)** in the ID3v2 tag,
  before the audio. ricercar's decoder (Symphonia 0.6.1) reads at most
  1 MiB looking for the audio, gives up, and reports "no suitable format
  reader". Reproduced with Symphonia 0.6.1: the original fails, the same
  file with the cover shrunk to 1000×1000 opens.

### Solution

- Local music is played in **VLC**. `~/Music` was removed from ricercar's
  library, so everything in ricercar is now Qobuz or Spotify.
- The decoder limit is reported to the ricercar developers:
  [ricercar-player/ricercar#5](https://github.com/ricercar-player/ricercar/issues/5)
  (opened 2026-10-24).

If local MP3s are ever wanted in ricercar again, shrinking the embedded
cover fixes them without touching the audio:

```sh
ffmpeg -i in.mp3 -map 0:a -map 0:v:0 -c:a copy \
  -c:v mjpeg -q:v 3 -vf "scale='min(1000,iw)':-2" \
  -disposition:v:0 attached_pic -map_metadata 0 -id3v2_version 3 out.mp3
```

---

## Problem 2: "Device or resource busy" while setting up

Under the selected DAC in **Settings → Audio output**:

```
Could not read the device: ALSA function 'snd_pcm_open' failed with
error 'Device or resource busy (16)'
```

`fuser -v /dev/snd/pcmC2D0p` showed `pipewire` holding the DAC.

### Cause

The **GNOME Sound settings window** was open. It runs level meters on
the default output, visible in `wpctl status` as:

```
GNOME Settings
    input_FL  < FiiO K11 R2R:monitor_FL  [active]
    input_FR  < FiiO K11 R2R:monitor_FR  [active]
```

That keeps PipeWire playing to the DAC, so it never goes idle and never
releases it. This was also the real cause of the earlier "busy" error on
the EF499 (TN-002).

### Solution

1. Close the GNOME Settings window.
2. Pause other sound (browser, video).
3. Wait about 10 seconds: `fuser -v /dev/snd/pcmC2D0p` prints nothing.
4. In ricercar, **Refresh** under Audio output, then play.

PipeWire releases an idle DAC by itself, so the K11 can stay the
system's default output and still be used bit-perfect by ricercar. While
ricercar plays, other apps' sound waits until it stops.

---

## Known issue: Qobuz Connect does not play from the phone

Qobuz Connect lets the Qobuz app list ricercar as a device
(`ricercar (trixie)`) and control it. Registration and selection work;
playback started from the phone does not.

```
qconnect::session: registered with Qobuz Connect as renderer Some(1)
qconnect::session: selected in the Qobuz app      (22:03:31)
qconnect::session: selected in the Qobuz app      (22:05:40)
qconnect::session: selected in the Qobuz app      (22:10:55)
(nothing after any of them)
```

- The iPhone app (Qobuz iOS 10.2.0, build 2610015094500) shows the track
  playing normally, but nothing plays on the computer, and the plugin
  logs no `loading track`, warning or error.
- Playing the same track from inside ricercar works (FLAC 24/96,
  bit-perfect).
- Signing out and in on both the app and the plugin changes nothing.
- The device is free (`fuser -v /dev/snd/pcmC2D0p` prints nothing), so
  this is not the "busy" problem above.
- ricercar starts plugins with a cleaned environment and does not pass
  `RUST_LOG`, so the plugin's debug output (`SetState` messages) cannot
  be enabled the way its README describes.

Reported to the plugin author:
[anon7627/ricercar-qconnect#1](https://github.com/anon7627/ricercar-qconnect/issues/1)
(opened 2026-10-24). Untested: the same selection from the web player at
play.qobuz.com, which would show whether the problem is the iOS app.

Workarounds: play from inside ricercar, or control it from the phone
with a UPnP app (mconnect Player, BubbleUPnP), which does not use the
plugin.

---

## Listening Chains

| Chain | Notes |
|---|---|
| K11 → headphone out (PO) → Audio-Technica ATH-R50x | bit-perfect to the K11 |
| K11 → line out (LO) → AIYIMA T20 (two valves) → RCA-to-P2 (3.5 mm) cable → TOMATE soundbar (2.0) | the speaker chain; the T20 clearly improved the sound. The tubes colour the analog signal after the DAC, so playback stays bit-perfect up to the K11 |
| EF499 → Sennheiser HD 650 | bit-perfect to the EF499 |

---

## Assumptions

- A Qobuz Studio (or Sublime) subscription for hi-res.
- Third-party plugin: it uses the Qobuz web API outside its terms of use.
  Qobuz may break it or restrict accounts that use it.

## Limitations

- Qobuz tops out at 24/192; the K11's 352.8/384 kHz is not used by it.
- Card numbers (`hw:2,0`, `hw:5,0`) can change after a reboot or
  replugging. Select the DAC again if it is missing.

## Future Improvements

- Qobuz Connect from the phone: retest when
  [anon7627/ricercar-qconnect#1](https://github.com/anon7627/ricercar-qconnect/issues/1)
  is answered or fixed, and move it out of Known issue.
- Try the web player (play.qobuz.com) as the Connect controller.
- Optionally make WirePlumber ignore the EF499 so only ricercar can use it.
- Replace the TOMATE soundbar with better speakers (planned upgrade).
- Follow [ricercar-player/ricercar#5](https://github.com/ricercar-player/ricercar/issues/5)
  (large ID3v2 tags); update this note when it is fixed.
