# Astoria 8K — POI Map

An interactive map of everything added to the **Astoria 8K** world for *7 Days to Die* (V 3.2.0).

**[Open the map →](https://drthunter.github.io/astoria-8k-poi-map/)**

Hover any marker to see what it is and what it replaced. Click to copy a `teleport` command.
Filter by layer or by POI pack, or search by name.

---

## What's in the world

`prefabs.xml` holds **13,062** decorations, against **13,011** in the stock map. Only +51 net,
because almost every addition *replaced* something rather than piling on top of it.

| Layer | Count | What it is |
|---|--:|---|
| **My builds** | 7 | Own prefabs from `LocalPrefabs`, on levelled terrain pads |
| **Compopack fill** | 1,812 | Junk ruins and 7th+ duplicates swapped for Compopack POIs |
| **Local fill** | 124 | Extra pack copies packed into the home cities |
| **Town lots** | 73 | First pass, dropped into cleared city lots |
| **Wilderness** | 49 | Rural, water and oversized POIs out in the open |
| | **2,065** | total markers on the map |

Markers are colour-coded by pack. The Compopack fill draws as small dots so the map stays
readable at full zoom; everything else keeps its numbered ring. Toggle any layer off with the
**Layer** chips.

## The Compopack pass

Astoria shipped with **1,070** Tier‑0 filler lots — `remnant_*`, `rubble_*`, `lot_vacant_*` and
friends, the non-enterable rubble RWG scatters through every town — plus a lot of repetition
(one downtown filler appeared **18** times). 1,812 of those slots now hold a Compopack POI.

| Reason | Count |
|---|--:|
| junk — Tier‑0 `remnant_*`, `rubble_*`, `lot_*`, `*_filler_*` | 1,416 |
| redundant — copies **beyond 6** of a vanilla POI, keeping the 6 most spread out | 396 |

- **820 distinct** Compopack POIs, drawn from a pool of **1,419 verified-unique** ones.
- Max **4 copies** of any one POI across the whole 8 km map.
- Closest same-POI pair **584 m**, median 4,221 m; never twice in one town unless that town had
  no alternative.
- **1,769 of 1,812** have sleeper volumes, so they are lootable and questable. The rubble had none.
- Difficulty spread is now T0 494 · T1 644 · T2 379 · T3 164 · T4 66 · T5 65.

`downtown_filler_park_*` and `downtown_filler_plaza_*` were **kept** — that is intended open
space in a downtown block, not junk. The 8×8 and 10×10 wilderness fillers were kept too; the
Compopack has nothing that size.

POIs come from nine packs: Compopack Classic AIO, MPLogue, Voltralux, Zeebark, WinterDawn,
Svarii, Caleseche, ShadowModernHouse and Cog's POIs.

## How the placement was done

Nothing was hand-placed. Sites were solved against the world's own data files, and both
placement rules were re-derived from this map before anything was written.

**Slots.** Every replacement reuses the lot it took over — same X/Y/Z, and an **exact footprint
match**, so no POI can overhang its lot and the town keeps its shape and lot count. Overlapping
POI pairs: 94 before, 94 after. All 94 are pre-existing in the stock Astoria map.

**Height.** Terrain is `dtm.raw[Z+4096, X+4096] / 256`, and a decoration's `Y` is
`round(terrain at POI centre) + 1`. `YOffset` is applied by the game at load, not baked into
`prefabs.xml` — verified against 1,055 wilderness POIs, 100% within 1.5 m. That is what keeps a
POI's ground floor level with the ground and its basement underground.

**Rotation.** A POI's facing is not its raw `rotation` value; it depends on how the prefab was
authored. Swapping A for B in a fixed slot is:

```
rotation_new = (rotation_old − RotationToFaceNorth_old + RotationToFaceNorth_new) mod 4
```

`(rotation − RotationToFaceNorth)` was constant across **401 of 401** repeated tile slots on this
map, so every replacement faces the street exactly the way its predecessor did.

**Fit.** Candidates were restricted to the slot's township context — downtown POIs into downtown
blocks, industrial into industrial, wilderness into open country — read from the RWG street tile
the lot sits in.

## Compopack compatibility (V 3.2.0)

The pack is built for an older game version, so it was audited before use. Nothing needed patching:

- All **2,099** prefabs parse (`.tts` versions 16–19; the game reads all of them).
- **Zero unknown block names** — every block it references exists in vanilla V 3.2.0. No
  dependency on any other mod.
- `biomes.xml`, `rwgmixer.xml` and `spawning.xml` are well-formed `<conditional>`-gated modlet
  patches whose xpath targets still exist. They only affect generating *new* worlds; Astoria is
  pre-generated.
- 12 name collisions with vanilla, all intentional overrides: 5 `part_terrain_greeble_*`
  (byte-identical) and 5 `rwg_tile_oldwest_*` (taller CP versions). Astoria places **zero**
  oldwest tiles, so that override changes nothing here.

**Duplicate check.** Only **5** Compopack POIs are byte-identical copies of vanilla ones —
`xcpv_LittleAsia_Park_01_TFP` (= `park_01`) and `xcpv_LittleAsia_filler_02/03/04/05_TFP`
(= `wilderness_filler_09/13/14/18`). All five excluded. No near-duplicates beyond those (same
dimensions *and* same block array under a different name), none against the other installed
packs, and none among the pack's own POIs. Net **1,424 → 1,419** distinct.

## Terrain edits

Astoria has **no** empty flat ground big enough for a 113×109 build — RWG levelled every large
plateau and then built a town on it. So `dtm.raw` was edited to level a pad under each own
build, with a smoothstep ramp blending back into natural terrain — 16 m for the four small pads,
28 m for the compound, whose west side drops into a gully.

| Pad | Size | Height | Mean cut/fill |
|---|--:|--:|--:|
| StarterBase_Prison_Cellblock | 113×109 | 58 m | 1.12 m |
| StarterBase_UFO_Farm | 69×75 | 56 m | 0.66 m |
| Walled compound | 146×202 | 56 m | 2.05 m |
| StarterBase_Hotel_Tower | 127×124 | 56 m | 2.20 m |
| StarterBase_Ranger_Station | 70×79 | 58 m | 1.08 m |

- 104,410 cells changed — **0.16%** of the map surface
- No existing POI footprint was touched. The ramp is frozen wherever it would reach one, so the
  ground under `cabin_16` (6 m off the compound's east wall) and `xcpv_Trailer_01_ZZTong` (7 m off
  the north wall) never moved.
- `dtm.raw.ORIGINAL-BACKUP` holds the original

## The starter bases

Seven builds from `LocalPrefabs`. The game ships no localization for POI names, so the prefab
filename *is* what shows on the compass, the map marker and in quest text. Each one is named for
the POI it was cut from, with a `StarterBase_` prefix so it reads as what it is.

Source POIs were identified by TF-IDF cosine similarity over each build's block-name set against
all 4,128 installed prefabs — what the blocks say, not a guess from the old filename.

| Cut from | Match | Name | Size |
|---|--:|---|--:|
| `prison_01` | 0.85 | **StarterBase_Prison_Cellblock** — holds the map spawn | 113×109 |
| `farm_17` | 0.70 | **StarterBase_UFO_Farm** | 69×75 |
| `ranger_station_07` | 0.89 | **StarterBase_Ranger_Station** | 70×79 |
| `hotel_03` | 0.90 | **StarterBase_Hotel_Tower** | 127×124 |
| `house_modern_18` | 0.94 | **StarterBase_Modern_House** — in the compound | 111×105 |
| `house_modern_31` | 0.93 | **StarterBase_Bunker_House** — in the compound | 69×75 |
| `Ayesoar_Mansion_by_MPLogue` | 0.83 | **StarterBase_Ayesoar_Mansion** — in the compound | 60×54 |

The UFO farm scores lowest because it is the most modified: it keeps `farm_17`'s 72 player farm
plots and planted corn, under 132 blocks of 5 m dome that is not in the original farm at all.

**None of them have sleeper volumes**, so they spawn no zombies and can be used as starter
bases from day one. Each carries a `YOffset` matched to its own build, so ground floors sit at
ground level and bunkers stay underground.

## The compound

The three houses share a walled yard, **146 × 202 m** at X 2012…2157, Z −586…−385. Each house also
keeps its own original walls inside it, so it is a wall within a wall.

**Why that size.** East and north are boxed in — `cabin_16` sits 6 m off the east side, a trailer
7 m off the north — so the only room is west and south. South is free; west runs into a gully, so
width is what costs earthwork:

| box | X clearance | Z clearance | west embankment over 8 m |
|---|--:|--:|--:|
| 140×193 (first attempt) | 2.3 m | 3.0 m | 29 m, max 14.1 m |
| **146×202 (built)** | **4.3 m** | **6.0 m** | **31 m, max 14.2 m** |
| 152×202 | 6.3 m | 6.0 m | 59 m, max 17.5 m |
| 158×202 | 8.3 m | 6.0 m | 69 m, max 18.4 m |

Going wider doubles the earthwork for 2 m of yard, so 146 is where it stops. There is no better
site either: searching the whole region for a clear box this size returned exactly two, and the
other is a lake bed at 3 m elevation. Minimum clearance to a wall is now **5 m**, up from 2 m.

**The wall** is four strip prefabs, 2 m thick, three buried courses of footing (`YOffset -3`). The
outer course stands six solid `concreteShapes:cube` high (~6 m) under a `polePlateHalf` cap; the
inner course is four high, so its top is a **walkway 4 m up** with the outer course rising 2 m
above it as a parapet, loopholed with `corrugatedMetalShapes:barsCentered` every 6 m. That is
7,363 structural blocks against ~2,570 before, and roughly 8× the concrete volume, since it is
full cubes rather than half blocks.

Each wall has a **stair dip** — the walkway steps down 7-6-5-4 and back up — so you can walk the
whole rampart in either direction and step off the low point into the yard.

**Five gates**, each 5 m wide with a `rollUpGate5x3White` and the walkway bridging over it: two
east (the road and city side), one north, one south, one west. Gate rotation follows the axis —
5-wide along Z needs rotation 1 or 3, along X 0 or 2 — measured across every roll-up gate in the
game's own prefabs rather than guessed.

## Spawn

`spawnpoints.xml` has a single point at **2430, −800**, in the prison yard of `StarterBase_Prison_Cellblock`,
close to the big city. The original ten are kept in `spawnpoints.xml.ORIGINAL-BACKUP`.

A Land Claim Block **cannot** be baked into the world — it needs an owner, and an unowned one
claims nothing. Place one yourself on arrival:

```
giveself keystoneBlock
```

Per the game's own description: *"LCBs will also prevent Sleeper and Biome respawns, but allow
Blood Moon or Screamer spawns."*

## Applying it to a save

New POIs only appear in chunks the save has not generated yet. For an existing save, rebuild
the affected chunks from the F1 console:

```
chunkreset <x1> <z1> <x2> <z2>
```

The player-relative form (`chunkreset` with no arguments) explicitly *does not reload POI data*,
so the coordinate form is required. It also destroys anything built in that box and resets loot
containers there. Given the scale of this pass, **a new save is the cleaner option**.

## Layout of the world folder

Generated alongside `%APPDATA%/7DaysToDie/GeneratedWorlds/Astoria 8K/`:

| File | Contents |
|---|---|
| `REPOPULATED.md` | the Compopack pass, compatibility audit and save notes |
| `REPOPULATED_pois.csv` | all 1,812 swaps — X, Z, reason, context, removed, added, tier |
| `MY_PREFABS.md` | the starter bases, spawn and claim-block notes |
| `ADDED_POIS.md` | the first 122, with coordinates |
| `ADDED_POIS_local.md` | the 124 local copies, by town, with reset commands |
| `TELEPORTS*.txt` | teleport commands |

Backups kept in place: `prefabs.xml.ORIGINAL-BACKUP`, `.BEFORE-ORPHAN-FIX`, `.BEFORE-DENSIFY`,
`.BEFORE-MYPREFABS`, `.BEFORE-REBUILD`, `.BEFORE-REPOP`, plus `dtm.raw.ORIGINAL-BACKUP` and
`spawnpoints.xml.ORIGINAL-BACKUP`.

## Known gaps

- **Own builds load from `LocalPrefabs`.** That is one of the game's prefab search paths, so it
  resolves in single player, but a dedicated server would need them moved into a mod folder.
- **Traders were deliberately skipped.** The Compopack's traders ship as a separate mod, and
  extra traders risk conflicting with Astoria's existing trader and quest routing.
- 50 junk POIs remain: the 8×8 and 10×10 wilderness fillers, which have no Compopack POI of
  matching size.
- `Mods\AAL-__vortex_tmp_00000001` was a leftover Vortex temp deployment, byte-identical to
  `AAJ-Classic All In One` including a second copy of `StallionsdensDoorTriggerVolumes.dll`.
  Two copies of the same Harmony assembly patch the game twice; it was moved to
  `Mods_disabled_by_claude\`.

## The map page

The base map is switchable: seven full-world renders live in `maps/`, and the **Map** chips
above Layer and Pack swap between them. Each is north up over the same 8192 m square as the
pins, so nothing needs registering, and only the one on screen is ever fetched.

`index.html` has no build step; serve the folder, or open the file directly.
