# Fantasy Climate Map Generator — Spec

## Overview
A single-page browser app (plain HTML/CSS/JS, no build step) where users paint a world map and generate realistic climate biomes based on Earth-like rules. Deploys automatically to GitHub Pages on every push.

---

## Climate Rules

**Latitude bands (Hadley cells):**

| Latitude | Pressure | Base moisture |
|---|---|---|
| 0–10° | ITCZ (low) | Very wet |
| 10–30° | Subtropical high (horse latitudes) | Very dry |
| 30–60° | Temperate (westerlies) | Moderate |
| 60–75° | Subpolar low | Wet |
| 75–90° | Polar high | Very dry |

**Prevailing wind directions** (determines rain shadow side):
- 0–30°: Trade winds → west (blow eastward landmasses dry on west side)
- 30–60°: Westerlies → east
- 60–90°: Polar easterlies → west

**Rain shadow:** Mountains block moisture from the windward side. Leeward cells get significantly drier, with the effect decaying over ~5–10 cells distance.

**Coastal moisture bonus:** Land cells adjacent to ocean get a wetness boost, decaying with distance inland.

**Biomes** (from temperature + moisture):

| | Very Dry | Dry | Moderate | Wet | Very Wet |
|---|---|---|---|---|---|
| Hot (0–20°) | Desert | Savanna | Savanna | Tropical forest | Rainforest |
| Warm (20–40°) | Desert | Shrubland | Grassland | Deciduous forest | Temperate rainforest |
| Cool (40–60°) | Cold desert | Steppe | Boreal forest | Boreal forest | Temperate rainforest |
| Cold (60–75°) | Tundra | Tundra | Taiga | Taiga | Taiga |
| Polar (75–90°) | Ice cap | Ice cap | Ice cap | Ice cap | Ice cap |

---

## Drawing Tools
- **Land** — paint land cells (left-click drag)
- **Water** — erase back to ocean
- **Mountain** — paint mountains on top of land; if painted on water, auto-fills a small radius (~2 cells) of land first
- **Erase** — set cells back to ocean
- Brush size selector (small / medium / large)

Map is a grid of cells mapped to latitudes (top = 90°N, bottom = 90°S). Blank cells = ocean.

---

## UI Layout

```
[ Land | Water | Mountain | Erase ]  Brush: [S|M|L]  [ Generate Biomes ]
+------------------------------------------------------------------+
|                                                                  |
|                        Canvas (grid)                             |
|                                                                  |
+------------------------------------------------------------------+
[ Legend ]   [ Toggle: latitude lines | wind arrows | moisture overlay ]
```

---

## Stages

**Stage 1 — Painting + basic biomes**
- Grid canvas with land/water/mountain painting
- Latitude lines overlay (toggle)
- Biome generation from latitude only (no wind/mountains yet)
- Color-filled biomes + legend
- *Push & deploy*

**Stage 2 — Rain shadow**
- Mountain layer rendered on top of biomes
- Prevailing wind direction per latitude band
- Rain shadow: reduce moisture leeward of mountains
- Coastal moisture bonus
- Biomes recalculated with moisture
- *Push & deploy*

**Stage 3 — Overlays + polish**
- Wind arrow overlay (toggle)
- Moisture intensity overlay (toggle, blue tint)
- Biome labels on hover (tooltip showing cell biome, temp band, moisture)
- Mountain auto-land-fill when painted on water
- *Push & deploy*

**Stage 4 — Extras**
- Export map as PNG
- Save/load map state (JSON in localStorage)
- *Push & deploy*
