# TN-002 — DAC Setup

## Reference Configuration

| Item | Value |
|------|-------|
| DAC 1 | FiiO K11 R2R, USB high speed, asynchronous |
| DAC 2 | HiFiMAN EF499, USB high speed, asynchronous |
| Headphones | Sennheiser HD 650 on the EF499 |
| Sound server | PipeWire + WirePlumber |

---

## What the DACs Report to Linux

Read from `/proc/asound/card<N>/stream0`:

| | FiiO K11 R2R | HiFiMAN EF499 |
|---|---|---|
| PCM formats | S16_LE, S24_3LE, S32_LE | S16_LE, S24_3LE, S32_LE |
| PCM rates | 44.1 kHz – **384 kHz** | 44.1 kHz – **192 kHz** |
| Native DSD (`DSD_U32_BE`) | up to **DSD256** | up to **DSD128** |
| Files above 192 kHz | play natively | refused on `hw:` |

DSD rate = ALSA rate × 32: 88.2 kHz → DSD64, 176.4 kHz → DSD128,
352.8 kHz → DSD256.

---

## Roles

| DAC | Role | Path |
|---|---|---|
| FiiO K11 R2R | System output (browser, video, everything else), and ricercar when idle. SE line out (LO) or headphone out (PO). | PipeWire, shared; `hw:` exclusive while ricercar plays |
| HiFiMAN EF499 | ricercar, bit-perfect, to the HD 650 | `hw:` exclusive |

A DAC can be the system output and still be used by ricercar: PipeWire
releases it a few seconds after the last sound, as long as the GNOME
Sound settings window is closed (TN-004).

Second chain: K11 → GR70 → ADC → optical → JBL Cinema 180. The ADC turns
the K11's analog output back into digital, so bit-perfect ends at the K11.
That is an intended sound choice.

---

## ricercar Settings

1. **Settings → Audio output**: choose the `hw:` entry marked
   **BIT-PERFECT** for the DAC.
2. ricercar volume at **100 %**; use the knob on the DAC.
3. **ReplayGain off** (the default).
4. Check the **green dot** on the chain line under the seek bar.
   Clicking the chain line shows every step that alters the samples.

The NOS/OS filter is set on the DAC itself. ricercar sends the original
samples either way; it is a sound preference.

---

## Problem: ALSA error when playing

ricercar showed an ALSA error on the EF499.

## Cause

`hw:` output needs the DAC exclusively. PipeWire had the DAC open
because it was the system's default output while the GNOME Sound
settings window was open; its level meters keep the default output busy
(found later, see TN-004):

```
$ fuser -v /dev/snd/pcmC*D0p
/dev/snd/pcmC5D0p:   jglerner  2710 F...m pipewire
```

`pcmC5D0p` = card 5, device 0, playback = the EF499.

## Solution

Close the GNOME Sound settings window and pause other sound; keeping the
system default on another device (here, the K11) also works. After that,
`fuser` shows ricercar holding the EF499:

```
/dev/snd/pcmC2D0p:   jglerner   2710 F...m pipewire   # K11, system
/dev/snd/pcmC5D0p:   jglerner 623426 F...m ricercar   # EF499, bit-perfect
```

To play bit-perfect on the K11 (for files above 192 kHz or DSD256),
temporarily switch the system output to another device in Sound
settings, then select the K11's `hw:` entry in ricercar. Selecting
`pipewire` in ricercar also reaches the K11, but not bit-perfect.

## Limitations

- **Card numbers change.** The K11 was `hw:2,0` and the EF499 `hw:5,0`
  in this setup, but numbers depend on the order devices appear. After
  a reboot or replugging, select the DAC again if it is missing.
- If the desktop makes the EF499 the default output (for example after
  replugging it), the ALSA "busy" error returns until the default is
  changed back.
- ricercar's native DSD / DoP output is marked experimental and untested
  on DSD hardware. Use DSD → 24-bit PCM until tested, and test native DSD
  at low volume.

## Future Improvements

- Optionally make WirePlumber ignore the EF499 so only ricercar can use
  it (a rule in `~/.config/wireplumber/wireplumber.conf.d/`).
- Test native DSD on the K11.
- Planned upgrade: replace the TOMATE soundbar (K11 → AIYIMA T20 →
  TOMATE) with better speakers.
