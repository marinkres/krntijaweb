# Demo audio

The A/B player on the page loads two files from this folder:

- `before.mp3` — the dry loop, straight out of the DAW
- `after.mp3` — the same loop rendered through KRNTIJA.VI

`before.wav` / `after.wav` are the masters. The MP3s are encoded from them at 192 kbps
with gain applied so the two sides are loudness-matched:

| | before | after |
|---|---|---|
| gain applied at encode | −1.7 dB | −0.5 dB |
| integrated loudness | −9.8 LUFS | −9.9 LUFS |
| true peak | −1.2 dBFS | −1.0 dBFS |

The 1.2 dB difference between the two gain figures is the level difference the plugin
introduces on its own. Taking it back out is the whole point: an A/B where one side is
louder always makes the louder side sound better, which would hide what the plugin
actually does.

**Recalculate these two numbers on every new bounce.** They are not a fixed property of
the plugin — an earlier `after.wav` sat 3.2 LU *above* `before`, this one sits 1.2 LU
*below* it. Reusing old gain values silently re-introduces the bias in the other
direction.

To re-encode after a new bounce:

```sh
ffmpeg -y -i before.wav -af "volume=-1.7dB" -c:a libmp3lame -b:a 192k before.mp3
ffmpeg -y -i after.wav  -af "volume=-0.5dB" -c:a libmp3lame -b:a 192k after.mp3
```

Re-measure with `ffmpeg -i FILE -af ebur128=peak=true -f null -` and adjust the two gain
values until the integrated loudness matches and neither true peak goes above −0.5 dBFS.
MP3 encoding adds inter-sample peaks, so a WAV that sits at −0.2 dBFS can come out of the
encoder above zero.

Both files must be the same loop, same length, same start. The player keeps them locked
to one another and only swaps which one is audible, so any offset shows up as a jump when
you hit the switch.
