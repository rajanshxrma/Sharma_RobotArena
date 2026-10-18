# Development Journal: IRONCLASH

Rajan Sharma · CSC 4370 · Project 1 · Track E Robot Arena Dashboard + Combat HUD pack

Each entry starts with the three stand-up questions (done / now / blockers) and then the decisions and problems from that block.

---

## Sun 18 Oct 2026, 3:30 PM: planning + base styles

**Done:** read the project brief, showcase, rubric and Q&A guide.
**Now:** pick a track, set up the repo, build the home page.
**Blockers:** none.

- Picked **Track E, Robot Arena Dashboard**. It has the most room for motion that actually means something (power-ups, alerts, scans). The **Combat HUD pack** is the natural pair: status bars, alert chips and overlays are the whole look of the site.
- Design direction: a dark "arena at night" HUD. Orange (`--accent`) is for actions and warnings, and cyan (`--cyan`) is for HUD lines and data. I considered a light theme and put it in the backlog instead, because the glow and scan-line effects depend on a dark background for contrast.
- Every color, font, spacing step, radius and timing value is a variable in `:root`, so the pages can't drift apart.
- Three font roles: Chakra Petch for headings (techy but readable), Inter for body text, JetBrains Mono for numbers and HUD tags.
- `style.css` has one fixed section order (variables > base > layout > components > pages > utilities > media queries). All `@keyframes` live in `animations.css` so the main file stays readable.
- **Live ticker:** a scrolling ticker with only one copy of the text would leave a gap before it loops. I doubled the items and animate the track by `-50%`, so the loop is seamless. The duplicate copy is `aria-hidden="true"` so screen readers don't read every alert twice.
- **Mobile menu without JS:** a checkbox plus a label (`.nav-toggle:checked ~ .main-nav`). The checkbox is visually hidden but still focusable, so the menu works from the keyboard.

## Sun 18 Oct 2026, 3:40 PM: Arena Roster + Battle Log

**Done:** home page committed.
**Now:** roster with filters, then the battle log.
**Blockers:** filtering with no JavaScript.

- **The filter:** radio buttons plus the `~` sibling selector. The catch is that `~` only reaches *later siblings*, so the radios can't sit inside the filter bar. They have to come before `.filter-bar` and `.roster-grid` at the same level. I left an HTML comment there because it's an easy thing to break when moving markup around.
- Stat bars use one custom property each (`style="--val:72%"`), so a single `.fill` rule styles every bar and only the number changes.
- Status uses a dot *and* a word (Online / Repair / Offline), so the meaning doesn't depend on color alone.
- Battle log: a match timeline (ordered list), a results table, and replay overlays that open with `:target` (the link's `#replay-1` becomes the URL hash). Each overlay has a close link back to the page.

## Sun 18 Oct 2026, 6:40 PM: Control Panel

**Done:** roster and battle log committed.
**Now:** the control panel, the most interactive page.
**Blockers:** making the stats react to the switches without JavaScript.

- **Key decision:** each power-up switch sets a 0/1 flag on the HUD with `:has()`, e.g. `.hud:has(#pu-overdrive:checked) { --od: 1; }`. Each bar computes its value as `min(100, max(0, base + bonus))`. The width is `calc(var(--now) * 1%)`, and a width transition makes it slide. One formula drives every bar, so I didn't need a separate rule for each combination of switches.
- **Live numbers:** `counter-reset: stat var(--now)` with `content: counter(stat)` prints the bar value as text, so the number and the bar always agree.
- **Considered and rejected:** a separate hard-coded rule for each power-up (the first version did this, e.g. `width: 96%`). It broke as soon as two power-ups were on together. The variable formula fixed that.
- **Launch countdown:** a `:target` overlay. 3-2-1-FIGHT! are four spans stacked in one grid cell with staggered `--delay` values. The animation is only applied while the overlay is open, so it replays on every launch.
- **Bug found + fixed:** with reduced motion turned on, the global rule shortens animation *durations* but not *delays*, so those users stared at an empty box for 3 seconds before FIGHT! appeared. I added `.countdown span { animation-delay: 0s !important; }` inside the reduced-motion block.
- Alert chips (OK / WARN / CRIT / LOCK) always have an icon and a word, not just a color. They sit in an `aria-live` region.

## Sun 18 Oct 2026, 7:20 PM: Team page

**Done:** control panel committed.
**Now:** team introduction page.
**Blockers:** none.

- Solo project, so I split the work into four roles (designer, layout, animation, QA). That kept the Kanban cards small and makes it clear what each part of the work covered.
- Time tracking is by phase (Proposal, Layout, Motion, Polish, Presentation), as the course asks.
- **Accessibility fix:** the time table sits in a horizontally scrolling `.table-wrap` on phones. A scroll box you can't focus can't be scrolled from the keyboard, so I gave it `tabindex="0"`, `role="region"` and a label.
- Relabelled the AI pilot cards on the control panel as "Step 05" of the setup flow, so the one declared bonus feature (Arena Cam) is clear.
