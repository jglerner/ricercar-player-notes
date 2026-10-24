# TN-003 — Quick Reference

## Check the system

```sh
dpkg --print-architecture                # amd64
dpkg-query -W -f='${Version}\n' ricercar # installed version
```

## List outputs

```sh
ricercar --print-devices     # what ricercar can use
aplay -l                     # ALSA playback devices
for c in /proc/asound/card*/stream0; do echo "== $c"; cat "$c"; done
```

## "ALSA busy" error

```sh
fuser -v /dev/snd/pcmC*D0p   # who holds each DAC
```

If `pipewire` holds the DAC ricercar should use:

1. Close the GNOME Sound settings window (its level meters hold the
   default output).
2. Stop other apps playing sound.
3. Wait a few seconds, then **Refresh** under Settings → Audio output
   and press play again.

See what holds the default output:

```sh
wpctl status | sed -n '/Audio/,/Video/p'
```

## Qobuz check

Chain line under the seek bar: green dot, `Qobuz`, `FLAC 24/…`, the DAC
name. A `file:///` source in the log means a local file, not Qobuz.

## Bit-perfect checklist

- `hw:` output marked BIT-PERFECT
- ricercar volume 100 %, volume on the DAC
- ReplayGain off, not muted
- Green dot on the chain line

## Which DAC for which file

| File | DAC |
|---|---|
| Up to 24/192 | EF499 (HD 650) or K11 |
| 352.8 / 384 kHz | K11 only |
| DSD64 / DSD128 | either |
| DSD256 | K11 (native), or converted to PCM |
