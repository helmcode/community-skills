---
name: remotion-product-video
description: Adds voiceover, music, subtitles, camera moves and well-timed motion to a product or launch video made with Remotion, using launch-video-kit. Use whenever the task is generating a voiceover with a TTS, choosing or extending a music track, mixing voice over music, adding subtitles that stay legible over a UI, moving a virtual camera over a UI rebuilt in HTML, or making animation in a Remotion video feel less mechanical (easing, anticipation, staggering, follow-through, timing).
license: MIT
---

# Remotion product video

Six pieces for the part of a video that is not the design: audio, subtitles and camera, plus the motion
principles that make what moves on screen feel authored. The kit's README explains the reasoning and the
numbers behind every decision (it is written in Spanish). This file is the working order and the rules
that are easy to skip.

**Paths.** `references/…` is relative to the folder of this SKILL.md. `scripts/…`, `src/…` and
`voiceover.json` are relative to the launch-video-kit checkout.

## 0 · Get the kit

The scripts and helpers live in [launch-video-kit](https://github.com/borjaperfra/launch-video-kit). If the
project does not have them yet:

```bash
npx degit borjaperfra/launch-video-kit launch-video-kit
cd launch-video-kit
cp .env.example .env                       # add your TTS key
cp voiceover.example.json voiceover.json   # one entry per line of voiceover
```

| Piece | What it solves |
|---|---|
| `scripts/gen-voice.mjs` | voiceover line by line against an OpenAI-compatible TTS endpoint, normalized |
| `scripts/analyze-music.mjs` | BPM, phase, energy arc and bar grid of a track |
| `scripts/loop-music.mjs` | extends a track to any length without an audible seam |
| `src/audio.ts` | ducks the music under the voice, with a measurable criterion |
| `src/Subtitle.tsx` | subtitles that stay legible over an interface, for watching muted |
| `src/camera.ts` | a virtual camera over a UI rebuilt in HTML |

The scripts run on their own, without installing the repo. Copy the `src/` helpers into the Remotion
project or import them.

## The order matters

1. **Voice first, edit second.** `gen-voice.mjs` prints the real length of every line in frames. Those
   numbers set the scene timings. If you edit first, every real take throws the edit off and you redo it.
2. **Then the music**, choosing the window by its energy arc (`analyze-music.mjs`) and extending it if
   needed (`loop-music.mjs`).
3. **Then the cuts and the motion**, locked to the bar grid the analysis prints. This is where the
   animation principles come in: read [references/motion-principles.md](references/motion-principles.md)
   before animating anything.
4. **Subtitles last**, once you know which lines already have on-screen text and which do not.

## Rules that get skipped

- **One voice file per line.** Never one long take.
- **Subtitle only what is not written on screen.** If the line already appears as on-screen text,
  subtitling it duplicates the words in the same place.
- **A subtitle's `dur` is reading time**, which has nothing to do with how long the line is spoken. Trim
  the ranges by hand so none overlaps the next subtitle or the on-screen text that follows.
- **`DUCK.base` stays at or below ~0.85.** Library tracks come mastered at 0 dB and any effect on top clips.
- **Camera: still, with short trips.** No continuous drift (it distracts) and no hard cuts (they jump).
  Every scene starts at the framing where the previous one ended.
- **Same interface on both sides of a seam, same component.** If one scene draws it by hand, it will
  differ in some detail, and that detail pops when the block changes.
- **Loop over the timings, never over the content.** A loop over the content array that indexes a
  shorter timings array crashes the render the moment you remove a beat.

## Verify with numbers

An agent cannot watch or hear the video. Everything that looks subjective has a number:

```bash
# the mix, section by section: voice over music should measure like voice alone
ffmpeg -hide_banner -ss 27 -t 4 -i out.mp4 -af volumedetect -f null /dev/null

# the global peak must stay below -1 dB
ffmpeg -hide_banner -i out.mp4 -af volumedetect -f null /dev/null

# a contact sheet instead of checking frames one by one
ffmpeg -v error -i f_%d.png -vf "scale=560:-1,tile=5x4" -frames:v 1 sheet.png
```

For motion, render stills at the frames around each move (`npx remotion still … --frame=N`) and check the
spacing: frames bunched at the ends of a move mean it eases; evenly spaced frames mean it is linear.

`loop-music.mjs` already checks its own seams: if any of them does not land in silence, it says so and you
pick two other gaps.

## Environment

- `ffmpeg` and `ffprobe` on the PATH, and Node 20 or later.
- In Git Bash on Windows: `ffmpeg` does not support `glob` patterns and `drawtext` fails for lack of
  fontconfig. Use numbered sequences. There is no `bc` either.
- Remotion's `staticFile()` does not handle spaces or accents well: rename assets to ASCII.
- `useVideoConfig().durationInFrames` inside a `<Sequence>` is the length of the SEQUENCE, not of the
  video. Do not mix it with absolute frames.

## What it cannot do

A TTS produces speech and nothing else. Sighs, laughs and sound effects have to be recorded. Synthetic
blips can come out of ffmpeg; the README has an example.
