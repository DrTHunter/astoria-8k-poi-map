# Astoria 8K — POI Map

An interactive map of everything added to the **Astoria 8K** world for *7 Days to Die* (V 3.2.0).

**[Open the map →](https://drthunter.github.io/astoria-8k-poi-map/)**

Hover any marker to see what it is and what it replaced. Click to copy a `teleport` command.
Filter by layer or by POI pack, or search by name.

---

## What's in the world

`prefabs.xml` went from **13,011 → 13,064** decorations. That is only +53 net, because most
additions *replaced* something rather than piling on top of it.

| Layer | Count | What it is |
|---|--:|---|
| **My builds** | 13 | Own prefabs from `LocalPrefabs`, on levelled terrain pads |
| **Local fill** | 124 | Extra pack copies packed into the home cities |
| **Town lots** | 73 | First pass, dropped into cleared city lots |
| **Wilderness** | 49 | Rural, water and oversized POIs out in the open |
| | **259** | total markers on the map |

197 of those took over a Tier‑0 filler lot — `remnant_*`, `rubble_*`, `lot_vacant_*` and friends,
the non-enterable rubble that RWG scatters through every town. Astoria had **1,070** of them.
Replacing filler instead of adding POIs means the town keeps its shape and its lot count, but
the dead lots become buildings you can actually loot and quest.

POIs come from eight packs: MPLogue, Voltralux, Zeebark, WinterDawn, Svarii, Caleseche,
ShadowModernHouse and Cog's POIs.

## How the placement was done

Nothing was hand-placed. Sites were solved against the world's own data files.

**Town lots.** Every RWG street tile declares its POI slots in `POIMarkerStart` /
`POIMarkerSize` / `POIMarkerPartRotations`. Those markers were resolved to world space through
each tile's own rotation, which gives the real lot rectangles — so a swapped-in POI lands
exactly where the tile intended a building, with the road, driveway and streetlights already
built and the ground already flat.

**Rotation.** A POI's facing is not its raw `rotation` value; it depends on how the prefab was
authored. The law used here was derived from 8,758 matched placements across Navezgane, the
four shipped Pregen worlds and Astoria itself:

```
rotation = (marker_rotation + tile_rotation + RotationToFaceNorth) mod 4
```

**Wilderness.** `dtm.raw` (8192², 16-bit) for terrain, `splat3.png` for roads, and a footprint
mask built from all 13,011 existing decorations using each prefab's real `PrefabSize`. Sites had
to be flat, clear with 8 m of margin, above the water plane and near a road. Every placement was
then re-checked at full 1 m resolution — **zero overlaps**.

**Repetition control.** In the local fill, no POI appears twice in the same town, copies are at
least 300 m apart, and nothing is used more than twice in the area.

## Terrain edits

Astoria has **no** empty flat ground big enough for a 103×103 build — RWG levelled every large
plateau and then built a town on it. So `dtm.raw` was edited to level a pad under each of the 13
own builds, with a 24 m cosine ramp blending back into natural terrain.

- 0.27% of the map surface modified, cut/fill ≤ 10 m per build
- No existing POI footprint was touched (asserted, not assumed)
- Land steeper than 45° in the modified area went **down**, 15,872 m² → 7,401 m²
- `dtm.raw.ORIGINAL-BACKUP` holds the original

## Spawn

`spawnpoints.xml` now has a single point at **2415, −817**, the centre of `Prison-perimiter`.
The original ten are kept in `spawnpoints.xml.ORIGINAL-BACKUP`.

A Land Claim Block **cannot** be baked into the world — it needs an owner, and an unowned one
claims nothing. Place one yourself on arrival:

```
giveself keystoneBlock
```

Per the game's own description: *"LCBs will also prevent Sleeper and Biome respawns, but allow
Blood Moon or Screamer spawns."* So it stops POI sleepers and wandering zombies inside the
claim, but not blood moons or screamers. Clear the prison once; the LCB keeps them from
coming back.

## Applying it to a save

New POIs only appear in chunks the save has not generated yet. For an existing save, rebuild
the affected chunks from the F1 console:

```
chunkreset <x1> <z1> <x2> <z2>
```

The player-relative form (`chunkreset` with no arguments) explicitly *does not reload POI data*,
so the coordinate form is required. It also destroys anything built in that box and resets loot
containers there.

On a fresh save none of this is needed — everything is present from the start.

## Layout of the world folder

Generated alongside `%APPDATA%/7DaysToDie/GeneratedWorlds/Astoria 8K/`:

| File | Contents |
|---|---|
| `ADDED_POIS.md` | the first 122, with coordinates |
| `ADDED_POIS_local.md` | the 124 local copies, by town, with reset commands |
| `MY_PREFABS.md` | the 13 own builds, spawn and claim-block notes |
| `TELEPORTS*.txt` | teleport commands for everything |

Backups kept in place: `prefabs.xml.ORIGINAL-BACKUP`, `.BEFORE-ORPHAN-FIX`,
`.BEFORE-DENSIFY`, `.BEFORE-MYPREFABS`, plus `dtm.raw.ORIGINAL-BACKUP` and
`spawnpoints.xml.ORIGINAL-BACKUP`.

## Known gaps

- **La Saignerie medieval pack (34 POIs) is not loaded.** It ships without a `ModInfo.xml` and
  its prefabs are not under a `Prefabs/` folder, so the game never sees it. It needs
  repackaging before those POIs can be placed.
- **Own builds load from `LocalPrefabs`.** That is one of the game's prefab search paths, so it
  resolves in single player, but a dedicated server would need them moved into a mod folder.
- **Traders were deliberately skipped** — the packs ship eight, and extra traders risk
  conflicting with Astoria's existing trader and quest routing.
- Six wilderness POIs sit on 6–8.7 m of ground relief. Nothing flatter exists at their size.

## The map page

The base map is switchable: seven full-world renders live in `maps/`, and the **Map** chips
above Layer and Pack swap between them. Each is north up over the same 8192 m square as the
pins, so nothing needs registering, and only the one on screen is ever fetched — the page
loads about 1.2 MB rather than the 1.5 MB that the old embedded JPEG cost every visitor.

`index.html` is otherwise unchanged and still has no build step; serve the folder, or open
the file directly.
