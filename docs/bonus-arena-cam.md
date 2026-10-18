# bonus feature - arena cam (computer vision overlay)

Rajan Sharma, CSC 4370 Project 1. This is bonus feature 4 from the course list.

![arena cam results at 1280px](screenshots/arena-cam-desktop-result.png)

## what it is

The arena cam is a fake computer vision overlay on one frame from the Titan-07 vs Volt Runner match. It's the last section of the battle log page (`battle-log.html#arena-cam`), since the battle log is where you go to look back at what happened in a match. You can get to it from the hint on the home page, from the "Scan a frame" button on the Titan-07 vs Volt Runner card in the timeline, or by scrolling down.

It has three modes, picked with three buttons:

- **live feed:** the arena floor and four bots, with a blinking LIVE chip
- **scan:** a beam sweeps down the frame, the grid flickers, the bots go grey, and a progress bar fills while the steps of the "analysis" show up one at a time
- **results:** a box locks onto each bot, one after another, and then its tag pops in (name, confidence, threat). The list next to the frame shows the same detections as text, and you can filter it by tank / speed / support.

The page says it's a simulation, since there's no real model or camera. Every box, label and score is typed into the html.

## the AI concept: object detection

Object detection means finding things in an image and saying what each one is. For every thing it finds, a detector gives back a box, a label and a confidence score. Here's how I faked each part:

- **bounding box:** a `.det` box inside each robot's `.cam-obj` wrapper. The wrapper is placed with `--x`, `--y` and `--w` percentages.
- **class label:** the tag text (like `TITAN-07 · tank`). Each class also gets its own color through `--c`.
- **confidence score:** the number in the tag (0.97), plus a confidence bar in the list. The bar reuses the site's `.bar` / `.fill` component.
- **cut-off:** a tick at 50% on every confidence bar. Anything under it isn't counted.
- **processing:** that's the scan mode.

Volt Runner is drawn a little blurry because it's moving fast, and its row says that's why it only scored 0.88.

## my design idea: show the unsure one

Most detection demos only show the confident boxes. I kept one low confidence detection on purpose: a robot half hidden behind the pit wall at **0.41**.

- It gets a dashed box that only covers the part of the bot you can see.
- The tag says `??? 0.41 uncertain` instead of a name.
- In the list it's "Unknown bot", marked not counted because it's under the cut-off, but it stays on screen for a person to check.

A real system that hides low scores can miss things, and one that shows them like facts can be wrong. Showing the "maybe" differently (a dashed line, a question mark, the word uncertain) makes the difference clear without relying on color.

## how it's built

- `battle-log.html`: the arena cam section, the "Scan a frame" button, and the link to `arena-cam.css`
- `css/arena-cam.css`: all the arena cam styles, with its own reduced motion part
- `css/animations.css`: 3 new keyframes (`cam-sweep`, `cam-flicker`, `box-lock`). It also reuses `blink`, `bar-grow`, `fade-up`, `pop-in` and `pulse`
- `index.html`: one link to the arena cam in the hero hint

The three mode buttons are radio inputs inside labels, so you can tab to them and use the arrow keys. Everything hangs off the camera section with `:has()`:

```css
.cam:has(#cam-scan:checked) .cam-beam { animation: cam-sweep 1.6s linear infinite; }
.cam:has(#cam-results:checked) .det { animation: box-lock 0.45s ... var(--d) both; }
```

Each box gets its own delay from an inline `--d`, so they lock on one at a time. The box is a child of the robot's wrapper and sized with percentages, so it can't slide off the robot when the screen size changes.

The filter fades things with `filter` instead of `opacity`, because the box animation already uses opacity and would win.

I couldn't reuse the `scan` keyframe from the home page, because it moves a fixed 480px and the camera frame changes height with the screen. `cam-sweep` uses percentages instead.

## trade-offs

- **no auto advance:** without js the page can't jump from scan to results by itself, so the results button pulses when the scan is done. It's one more click, but it works the same with a keyboard.
- **fixed scores:** the same frame always gives the same numbers, which is fine for a mockup but isn't how a real camera works.
- **side view:** the bots use the same side view svgs as the rest of the site, so it's more like a wide shot than a top down camera.
- **`:has()` support:** it needs a newer browser (chrome 105+, safari 15.4+, firefox 121+). The rest of the site already uses it.
- **short tags on phones:** under 640px the tags only show the name and score, because the full tags covered the bots. The rest is in the list right under the frame.

## test checks

All of these were done in Chrome at 375, 768 and 1280px.

### layout

| # | check | result |
|---|---|---|
| L1 | every box sits on its bot and stays inside the frame at all 3 widths | ok, 4/4 bots at every width. The bot on the right edge has its tag grow to the left so it stays in the frame |
| L2 | only one panel and one status chip show in each mode | ok |
| L3 | in results, the boxes lock on one at a time | ok, right after switching only the first box is in and the rest are waiting on their delay |
| L4 | the scan beam is visible while scanning | fixed. The beam first went from -100% to 100%, so its bright edge was off the frame half the time. Changed it to -100% to 5% |
| L5 | the rest of the battle log still works | ok, the replay popups still open and close, and the timeline and table look the same apart from the new button |

### accessibility

| # | check | result |
|---|---|---|
| A1 | keyboard only: tab to the mode buttons and change mode | ok, tab reaches the group and the arrow keys go live feed > scan > results |
| A2 | you can see where focus is | ok, the label gets the same yellow outline as the rest of the site |
| A3 | every input has a label and every image has alt text | ok, 8 inputs and 5 images on the page, none missing |
| A4 | not color alone | ok, the threat chips have an icon and a word (high / med / low), and the unsure detection has a dashed box, a "?" and the word uncertain |
| A5 | screen readers get the same info as the boxes | ok, the boxes are `aria-hidden` and the list has the name, class, confidence and threat as text |
| A6 | reduced motion | ok, with reduced motion emulated in devtools there's no beam or flicker, the progress bar is full right away, and all boxes and tags show at once. I set `animation-delay: 0s` for them, because I'd already hit that delay bug on the control panel countdown |
| A7 | contrast of the new tag colors | ok, between 6.0:1 and 13.3:1, so they all pass AA |

### responsive

| # | check | result |
|---|---|---|
| R1 | 1280px: frame and list side by side | ok |
| R2 | 768px: frame full width, list under it in 2 columns | ok |
| R3 | 375px: everything stacks, and the tags still fit | ok, the tags show only name + score and the list is 1 column |
| R4 | no sideways scrolling | fixed. At 375 the battle log was 473px wide, but that came from the results table, not the arena cam. Added `.grid > * { min-width: 0; }` in `style.css`, and now it's 375 at 375, 768 at 768 and 1280 at 1280 |

### screenshots

- `screenshots/arena-cam-desktop-result.png`: 1280, results
- `screenshots/arena-cam-desktop-scan.png`: 1280, scan
- `screenshots/arena-cam-tablet-scan.png`: 768, scan with the beam mid sweep
- `screenshots/arena-cam-tablet-filter-tank.png`: 768, results with the tank filter on
- `screenshots/arena-cam-mobile-idle.png`: 375, live feed
- `screenshots/arena-cam-mobile-result.png`: 375, results
- `screenshots/arena-cam-reduced-motion-scan.png`: 1280, scan with reduced motion on (no beam)
