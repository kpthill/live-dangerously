# AutoBiomes — implemented climate-simulation pipeline

Implementation of the climate simulation from **Fischer, Dittmann, Weller & Zachmann,
"AutoBiomes: procedural generation of multi-biome landscapes", *The Visual Computer* (2020)**
(`AutoBiomes.pdf`, §3.2), adapted to the browser map app.

The original latitude-band heuristics (`_spec/climate-map.md`) are replaced by a genuine
grid-based simulation. The user still paints land / water / mountain; the simulation derives a
full climate from that terrain and classifies biomes with a discretized Whittaker diagram.

The pipeline is a sequence of grids (paper Fig. 2), each computed from the previous:

```
cells (painted) → elevation → temperature → wind field → precipitation → biomes
```

## 0. Elevation
Derived from painted cell types: ocean = 0, land = 0.12, mountain = 1.0. Two box-blur passes
let peaks slope into surrounding hills (`max(0.5·self, neighbourhood mean)` keeps summits high);
ocean is pinned to sea level. Columns wrap east–west; rows clamp at the poles.

## 1. Temperature (°C) — paper step 1
Two interpolation modes, both with a **height-based falloff** (`−lapse · elevation`):
- **Sine (latitude):** `T = Teq − (Teq − Tpole)·sin(|lat|/90 · π/2)` — the equator-to-pole gradient.
- **Bilinear (corners):** bilinear blend of four corner temperatures (N edge / S edge), for
  arbitrary regional gradients.

## 2. Wind — paper step 2 (simplified semi-Lagrangian)
A vector field forced by four user-set **corner directions** (the "external forces"). Initialised
as a bilinear blend of the corner vectors, then refined for *N* iterations. Each iteration, per cell:
1. **self-advection** — backtrace one step along the current wind and bilinearly sample the field;
2. **persistence** — lerp toward the nearest corner force;
3. **micro-disturbance** — rotate by a small seeded-random angle (turbulence).
Vectors are renormalised to a constant speed (only direction matters downstream). Diffusion and
pressure are dropped, exactly as the paper does.

## 3. Precipitation (cm/yr) — paper step 3 (iterative moisture transport)
Moisture and accumulated-precipitation grids evolved over *N* iterations:
- **Evaporation:** water cells are sources; the amount is a temperature-dependent function
  (exponential or linear) — warmer seas evaporate more.
- **Advection:** each cell pushes a `flow` fraction of its moisture downwind — 70 % to the main
  downwind cell and 15 % to each shoulder cell (dispersion); the rest stays.
- **Precipitation, two-step:**
  - *during transport* — moving into a colder cell wrings out moisture
    (`orographic · max(0, ΔT)` + a base fraction), producing **rain shadows** behind mountains and
    on lee coasts;
  - *local/convective* — moisture held over a cell rains out in proportion to local temperature.
Raw precipitation is scaled to a 0–450 cm/yr range for classification.

## 4. Biome classification — paper step 4 (discretized Whittaker diagram)
A 6 × 6 lookup table indexed by temperature band (edges −13/0/8/16/24 °C) and precipitation band
(edges 10/40/100/200/350 cm/yr). 15 biomes: ice cap, tundra, taiga, boreal forest, cold desert,
steppe, grassland, shrubland, temperate forest, temperate rainforest, desert, savanna, tropical
seasonal forest, tropical rainforest, plus ocean. The table can be freely edited (paper).

## UI
- **Climate Parameters** panel grouped by pipeline step, with live sliders/selects (seeded, so a
  given parameter set is reproducible).
- **View** selector renders any intermediate grid as the base layer — Biomes / Temperature /
  Precipitation / Elevation — mirroring the paper's per-step visualisation (Fig. 3), with a colour
  ramp legend.
- **Overlays:** latitude lines and the simulated wind vector field.
- Hover tooltip reports elevation, temperature, precipitation and biome per cell.
- Example map, PNG export, and localStorage save/load (terrain **and** parameters).
