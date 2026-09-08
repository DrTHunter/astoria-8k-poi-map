# Astoria 8K — POI Map

An interactive map of every POI added to my **Astoria 8K** world for *7 Days to Die* (V 3.2.0).

### **[Open the map →](https://drthunter.github.io/astoria-8k-poi-map/)**

Hover a marker to see what it is and what it replaced. Click one to copy its `teleport` command.
Filter by layer or by POI pack, search by name, and switch between seven full-world base renders.

> ### Want to actually play this map?
> This repo is only the map page. The world itself — the map patch, the starter bases, and a
> step-by-step setup guide written for people who have never modded before — lives here:
>
> ### **[→ DrTHunter/7d2d-astoria-my-world](https://github.com/DrTHunter/7d2d-astoria-my-world)**

---

## What's plotted

**2,106 markers**, everything in the world that isn't stock Astoria, plus every trader. The world
holds **13,105** decorations against 13,011 in the stock map — only +94 net, because almost every
addition *replaced* something rather than piling on top of it.

| Layer | Count | What it is |
|---|--:|---|
| **Traders** | 38 | One within a short walk of every town — 14 added, 3 given a better building |
| **Starter bases** | 10 | My eight builds on levelled pads in two walled plots, plus two horde bunkers. No sleeper volumes |
| **Compopack fill** | 1,812 | Tier‑0 ruins and 7th+ duplicates swapped for Compopack POIs |
| **Local fill** | 124 | Extra pack copies packed into the home cities |
| **Town lots** | 73 | First pass, dropped into cleared city lots |
| **Wilderness** | 49 | Rural, water and oversized POIs out in the open |

Markers are coloured by pack. The Compopack fill draws as small dots so the map stays readable at
full zoom; everything else keeps its numbered ring. The dashed rectangle is the walled compound,
and the ringed dot is the map spawn at `2430, -800`.

The full write-up of how all of it was placed — the rules, how they were derived from the map's own
data, and every swap as a CSV — is in the other repo:
[`docs/REPOPULATED.md`](https://github.com/DrTHunter/7d2d-astoria-my-world/blob/main/docs/REPOPULATED.md)
and [`docs/MY_PREFABS.md`](https://github.com/DrTHunter/7d2d-astoria-my-world/blob/main/docs/MY_PREFABS.md).

---

## How the page works

The whole thing is **one file**. `index.html` has no build step, no dependencies and no network
calls beyond the base map images — open it directly off disk or serve the folder, either works.

```
index.html      the page: styles, data and logic
maps/*.webp     seven full-world renders, 8192 m square, north up
```

### The data

Six `const` declarations near the top of the `<script>` hold everything. Regenerating the map means
rewriting those and nothing else.

| | |
|---|---|
| `POIS` | one object per marker: `{name, mod, x, y, z, w, d, where, note, n}` |
| `PACKS` | pack code → `{n: display name, c: colour}` |
| `CATS` | layer key → `{n: name, d: description}`; the order here is the order in the sidebar |
| `SPAWN`, `COMPOUND` | the spawn marker and the compound outline |
| `MAPS` | the base map list, `[filename, label]` |

`x` and `z` are the POI's **north-west corner** in world coordinates, exactly as `prefabs.xml`
stores them; `w`/`d` are its footprint after rotation. The page maps them to the world square with

```js
left = (x + 4096) / 8192          top = (4096 - z - d) / 8192
```

so anything using the same 8192 m square lines up without registering, which is why the base maps
can be swapped freely. `where` picks the layer, `mod` picks the colour, and `n` is the number drawn
in the pin.

### Base maps

Only the render currently on screen is ever fetched, and the choice is remembered in
`localStorage`. Adding one is two steps: drop the file in `maps/`, add a `[filename, label]` pair
to `MAPS`.

### Notes

- Pan and zoom are a CSS transform on one container, so 2,000-odd pins stay smooth. Pins are
  counter-scaled by `--inv` to keep a constant screen size.
- Clipboard falls back to `document.execCommand('copy')` — `file://` isn't a secure context, so the
  async clipboard API is unavailable when the page is opened straight off disk.
- Marker geometry is generated from the live `prefabs.xml`, so the page can't drift from the world:
  if a POI moves, the pin moves with it.
