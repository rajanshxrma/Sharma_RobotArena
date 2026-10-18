# QA notes - IRONCLASH

Tested sun 18 oct in Chrome. Every page was checked at 375px (phone), 768px (tablet) and 1280px (desktop).

What I checked on each page:
- layout at all three widths, and no sideways scrolling
- keyboard only (tab, shift+tab, arrow keys in radio groups, space on switches, enter on links)
- every filter, switch, radio and popup actually does what it should
- links, anchors and images all work
- every input has a label and every image has alt text, headings in order, no `<script>` anywhere
- contrast of the text colors

## results by page

| page | 375 | 768 | 1280 | keyboard | notes |
|---|---|---|---|---|---|
| home | ok | ok | ok | ok | hero stacks on phones, menu toggle opens and closes, ticker loops with no gap |
| arena roster | ok | ok | ok | ok | tank shows Titan-07 + Crusher, speed shows Volt Runner + Blink, support shows Medic-Bay + Patch, all shows all 6 |
| battle log | ok (after fix 5) | ok | ok | ok | replay popups open and close. arena cam goes live feed > scan > results with the mouse or arrow keys |
| control panel | ok | ok | ok | ok | tab to the bot cards and the arrow keys change the bot, focus outline shows on the card, space flips the switches |
| team | ok | ok | ok | ok | time table scrolls inside its box on phones and can be focused |

## control panel math

| what I did | expected | got |
|---|---|---|
| default (Titan-07) | ATK 84, HULL 98 | 84 / 98 |
| picked Crusher | HULL 22, ATK 97, hull CRIT chip | yes |
| + repair drone + overdrive | HULL 62, ATK 100 (capped), HEAT 75, CRIT goes away, battery drain chip shows | yes |
| + lava floor | HEAT 90, lava chip shows, loadout says "Lava Floor" | yes |

## contrast (AA needs 4.5:1)

- body text on the background: 16.4:1
- muted text on the background: 8.2:1, and on panels: 7.4:1
- cyan on the background: 12.6:1
- orange on the background: 6.7:1
- dark text on the orange button: 6.7:1
- red on a panel (the lowest): 5.4:1

All of them pass.

## bugs found and fixed

1. **reduced motion countdown:** with reduced motion on, the launch countdown showed an empty box for 3 seconds. The reduced motion rule shortened the durations but not the delays. Fixed with `.countdown span { animation-delay: 0s !important; }`.
2. **time table on phones:** you couldn't scroll it with the keyboard, because a scrolling div can't be focused by default. Added `tabindex="0"`, `role="region"` and a label.
3. **replay headings:** the replay popups went from h2 straight to h4. Changed the round titles to h3 and updated the css selector.
4. **power-up bars:** the bars were wrong when two power-ups were on, because the first version had one fixed width per power-up. Switched to 0/1 flags and one formula per bar.
5. **battle log on phones:** the page was 473px wide on a 375px phone. The results table's no-wrap cells stretched the grid column, since grid items default to `min-width: auto`. Added `.grid > * { min-width: 0; }`, so the table scrolls in its own box now. Re-checked all 5 pages at all 3 widths and none of them scroll sideways.

## links + assets

All the local links and image paths on the 5 pages work. Every `#` link has a matching id (`#main`, `#replay-1` to `#replay-3`, `#launch`, `#ai-pilot`, `#arena-cam`). No missing files in the css either.
