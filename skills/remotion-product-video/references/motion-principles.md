# Motion principles for product video

In 1987 John Lasseter published *Principles of Traditional Animation Applied to 3D Computer Animation*
(SIGGRAPH '87). He took the principles Disney's animators had worked out by hand, later collected by
Frank Thomas and Ollie Johnston in *The Illusion of Life*, and showed that a computer does not apply them
by itself: left alone, software moves things at constant speed, in straight lines, all at once. That is
exactly what makes a product video look like a screen recording.

Lasseter's paper keeps eleven of the twelve classic principles (he drops solid drawing, which belongs to
pencil work). Below, each one translated to a product video in Remotion, where what moves is mostly an
interface: cards, panels, cursors, numbers, text and the camera.

## The one rule above all

**Motion explains something or it does not happen.** Every move should answer "what changed?" or "look
here". If a move answers neither, cut it. The camera rule in SKILL.md (still, with short trips) is this
rule applied to the camera.

## The eleven principles

### 1. Timing

How many frames a move takes decides what it means. Few frames feel light and confident; many frames
feel heavy or important. In a product video, timing is mostly set by the voice: the line's frame count
(from `gen-voice.mjs`) is the budget the move has to fit.

- UI elements entering: 12 to 20 frames at 30 fps.
- Something the viewer must read: hold it still at least as long as it takes to read it aloud.
- Vary timing on purpose. If everything takes 15 frames, the video has no rhythm.

### 2. Slow in and slow out

Nothing in the real world starts or stops at full speed. Never leave `interpolate` linear for anything
visible:

```tsx
import { interpolate, Easing, useCurrentFrame } from "remotion";

const frame = useCurrentFrame();
const x = interpolate(frame, [0, 18], [-40, 0], {
  extrapolateLeft: "clamp",
  extrapolateRight: "clamp",
  easing: Easing.bezier(0.16, 1, 0.3, 1), // fast start, long soft landing
});
```

- Entering: ease out (it arrives and settles).
- Leaving: ease in (it gathers speed and goes).
- Moving from A to B on screen: ease in and out (`Easing.inOut(Easing.cubic)`).
- Always clamp, or values keep going past the range before and after the move.

### 3. Anticipation

A small move in the opposite direction before the main one tells the viewer something is about to
happen. In UI: a button dips a few percent before the click, a card nudges back before it slides
forward, the cursor slows down before reaching its target. Keep it short (3 to 6 frames) and small.

### 4. Staging

One idea on screen at a time, and the eye goes where you want. Before something important appears, clear
or dim what is around it. Use the camera, scale and contrast to point; never two things moving for
attention in the same moment. If the voice is talking about the sidebar, nothing else on screen moves.

### 5. Follow through and overlapping action

Parts of a thing do not all stop at the same frame. When a panel lands, its contents can arrive a few
frames later; a list appears one row after another instead of as a block. Stagger by a fixed number of
frames:

```tsx
const rows = items.map((item, i) => {
  const local = frame - i * 3; // 3-frame stagger
  const y = interpolate(local, [0, 14], [12, 0], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
    easing: Easing.out(Easing.cubic),
  });
  return { item, y };
});
```

Remember the rule in SKILL.md: loop over the timings, so removing an item never breaks the render.

### 6. Squash and stretch

Things that deform a little look alive; rigid things look like diagrams. In an interface keep it subtle:
a 2 to 4% scale on press, a toggle knob that stretches as it travels. A `spring` with some bounce gives
the settle for free:

```tsx
import { spring, useVideoConfig } from "remotion";

const { fps } = useVideoConfig();
const s = spring({ frame: frame - pressAt, fps, config: { damping: 12, stiffness: 180 } });
const scale = interpolate(s, [0, 1], [0.96, 1]);
```

`damping` around 200 removes the overshoot entirely; lower values bounce more. For a serious product,
bounce once at most.

### 7. Arcs

Living things move in curves; machines move in straight lines. A cursor travelling across the screen
should follow a gentle arc, which you get by easing x and y with different curves or adding a small
offset that peaks midway. The camera is the exception: its trips are short and straight, because a
curving camera reads as drift.

### 8. Secondary action

A small move that supports the main one without competing with it. The card slides in (primary) and its
shadow softens as it lands (secondary); a number counts up and its label fades in after. If the secondary
action draws the eye away from the primary, it is too big.

### 9. Exaggeration

Push a move a little past reality so it reads at video speed and on a phone screen. A highlight a bit
brighter than the real one, a zoom to 1.1 instead of 1.03. Product videos need much less exaggeration
than cartoons: enough to be noticed, never enough to look like a toy.

### 10. Straight ahead and pose to pose

Pose to pose means defining the key positions first and filling the frames between them. That is how
`interpolate` and `camera.ts` framings already work, and it is the right default: you control where
things are on the frames that matter (when the voice says the word). Straight ahead, frame by frame, is
for organic effects such as particles or noise; drive those from `frame` with a formula.

### 11. Appeal

The sum of the rest: clear shapes, readable text, motion with a consistent personality. Pick one easing
family and one stagger value for the whole video and stick to them, so every scene feels made by the same
hand.

## Checklist before rendering

- [ ] Every move explains a change or points somewhere. Nothing moves just to move.
- [ ] No visible `interpolate` is linear, and every one is clamped.
- [ ] Groups stagger instead of arriving as a block.
- [ ] Only one thing competes for attention at any moment, and it matches what the voice is saying.
- [ ] Springs overshoot once at most.
- [ ] The same easing and stagger values are used across the whole video.

## Further reading

- John Lasseter, *Principles of Traditional Animation Applied to 3D Computer Animation*, SIGGRAPH '87.
- Frank Thomas and Ollie Johnston, *The Illusion of Life: Disney Animation* (1981).
- [dylantarre/animation-principles](https://github.com/dylantarre/animation-principles): a set of skills
  that apply the twelve principles by role and medium (`npx skills add dylantarre/animation-principles
  --skill motion-designer`).
