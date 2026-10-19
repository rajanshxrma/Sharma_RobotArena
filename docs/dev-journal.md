# dev journal - IRONCLASH

Rajan Sharma, CSC 4370 Project 1. Each entry starts with my stand-up (done / now / blockers), then what I decided or ran into.

## sun 18 oct, 3:30 pm - planning + base styles

done: read through the project page, showcase, rubric and Q&A guide
now: pick a track, set up the repo, build the home page
blockers: none

- Went with Track E, Robot Arena Dashboard. It has the most room for animation that actually means something (power-ups, alerts, scans). The Combat HUD pack fits it best, since status bars, alert chips and overlays are basically the whole look.
- Picked a dark "arena at night" hud style. Orange is for actions and warnings, cyan is for hud lines and data. I thought about a light theme but put it in the backlog, because the glow and scan line effects need a dark background to show up.
- Put every color, font, spacing step and timing in `:root` variables before building anything.
- Three fonts: Chakra Petch for headings, Inter for text, JetBrains Mono for the numbers and hud tags.
- `style.css` goes in a fixed order (variables > base > layout > components > pages > utilities > media queries). All the keyframes go in `animations.css` so the main file stays readable.
- ticker: with one copy of the alerts there'd be a gap before it loops, so I doubled the items and slide the track by -50%. The second copy is `aria-hidden` so screen readers don't read it twice.
- mobile menu with no js: a checkbox and a label (`.nav-toggle:checked ~ .main-nav`). The checkbox is hidden but still focusable, so the menu works with the keyboard.

## sun 18 oct, 3:40 pm - roster + battle log

done: home page
now: roster with filters, then the battle log
blockers: how to filter with no js

- roster filter: radio buttons + the `~` selector. The catch is that `~` only reaches later siblings, so the radios have to come before `.filter-bar` and `.roster-grid` at the same level. I left a comment in the html because that's easy to break by moving markup around.
- Each stat bar takes its value from one variable (`style="--val:72%"`), so one `.fill` rule styles every bar.
- Status is a dot plus a word (Online / Repair / Offline), not just a color.
- battle log: the timeline is an ordered list, then a results table and replay popups that open with `:target`. Each popup has a close link back to the page.

## sun 18 oct, 6:40 pm - control panel

done: roster and battle log
now: control panel, the most interactive page
blockers: getting the stats to react to the switches without js

- My first version had a hard coded width for each power-up (like `width: 96%`). That broke as soon as two power-ups were on at the same time, because the rules fought each other.
- Fixed it by making each switch only set a flag: `.hud:has(#pu-overdrive:checked) { --od: 1; }`. Each bar then works out `min(100, max(0, base + bonus))`, the width is `calc(var(--now) * 1%)`, and a transition slides it. Now any mix of power-ups works.
- The numbers next to the bars use `counter-reset: stat var(--now)` and `content: counter(stat)`, so the number always matches the bar.
- launch countdown: a `:target` popup where 3, 2, 1 and FIGHT! are stacked in one grid cell with staggered delays. The animation only gets added while the popup is open, so it plays again every launch.
- bug: with reduced motion turned on, the countdown sat on an empty box for 3 seconds before FIGHT! showed up. The reduced motion rule shortens the animation durations but not the delays. Fixed it with `.countdown span { animation-delay: 0s !important; }` in the reduced motion block.
- Alert chips (OK / WARN / CRIT / LOCK) always have an icon and a word, and they sit in an `aria-live` area.

## sun 18 oct, 7:20 pm - team page

done: control panel
now: team intro page
blockers: none

- Split the work into four roles (designer, layout, animation, QA). That kept my kanban cards small.
- Time tracking is by phase, the way the course asks for it.
- On phones the time table scrolls sideways inside its box. You can't scroll a box from the keyboard unless it can be focused, so I gave it `tabindex="0"`, `role="region"` and a label.
- Renamed the AI pilot cards on the control panel to "step 05" so it's clear the arena cam is my one bonus feature.

## sun 18 oct, 7:30 pm - proposal, readme, arena cam, qa

done: team page, kanban, journal
now: proposal with wireframes, readme, the arena cam bonus, then a full QA pass
blockers: none

- Wrote the proposal doc with wireframes for all five pages plus the mobile home layout.
- arena cam: three radio buttons (live feed / scan / results) and `:has()` on the camera section. Each box sits inside the same wrapper as its robot and uses percentage positions, so it stays on the bot at any screen size. I kept one low confidence detection (0.41) with a dashed box on purpose, so the UI shows that detection isn't always sure.
- The scan beam's bright edge was off the frame half the time at first. I changed the sweep to go from -100% to 5% so it stays visible.
- QA found the battle log was 473px wide on a 375px phone. Grid items have `min-width: auto` by default, so the results table's no-wrap cells stretched the column. Fixed it with `.grid > * { min-width: 0; }`, and now the table scrolls inside its own box.
- QA also found the replay popups jumping from h2 to h4, so I changed the round titles to h3.
- Checked contrast on all the colors. The lowest is red on a panel at 5.4:1, which still passes AA.
