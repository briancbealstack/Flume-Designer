# FlumePath — water slide alignment solver

**Version 1.1.1** · single-file browser application · no build step, no install, no server

FlumePath is a preliminary-design tool for water slide (flume) alignments. You give it a
platform height, a site envelope and a flume cross-section; it sweeps several hundred
candidate alignments, solves the ride dynamics along each one, ranks them against limits
*you* supply, and exports the winning centreline to SolidWorks.

It is a concept-stage instrument. It is not a stamped design, and it does not certify
compliance with anything.

---

## Contents

| File | Purpose |
|---|---|
| `index.html` | The entire application — markup, styles, solver and exporters in one file |
| `USER_MANUAL.md` | How to drive it, end to end, with a worked example |
| `README.md` | This document — what it is, how it works, what it assumes |

## Running it

**Live:** <https://briancbealstack.github.io/Flume-Designer/>

Or open `index.html` in any modern browser — download it, double-click it, done. The file is
wholly self-contained, so it runs the same from a local disk, a network share or a USB stick
as it does from the hosted URL.

There is no build, no package manager and no server. The only external reference is a
Google Fonts stylesheet for the Barlow typefaces; with no network the app falls back to
system fonts and every function still works. Nothing is uploaded, and no data leaves the
browser — projects are saved to and loaded from local JSON files you control.

---

## What it does

### 1. Generates candidates

Four parametric alignment families are swept across a radius range you set:

| Family | Swept over |
|---|---|
| **Straight drop** | ramp length, 60 ft → site width, in 20 ft steps |
| **Helix** | radius × {1, 1.25, 1.5, 1.75, 2, 2.5, 3} turns × left/right hand |
| **Serpentine** | radius × {2…6} bends × {90°, 120°, 150°, 180°} arcs |
| **Double helix** | radius × {2, 3, 4} turns, as an S-reverse pair |

Every candidate is gated twice before it survives: its plan bounding box must fit inside
the site envelope, and its geometry must be able to absorb the required drop.

Sweep size is `n_radii × 37 + n_ramp_lengths`, where `n_radii` follows from your radius
min/max/step — so the radius range is the main control on how long a sweep takes. On the
shipped defaults (48 ft drop, 140 × 140 ft site, six radii) that is 227 attempts, of which
118 survive: 107 are rejected on footprint and 2 because no grade can absorb the drop.

### 2. Solves the ride

Each surviving alignment is solved for every enabled rider case, and the **worst** result
per criterion is kept along with the rider and station that governed it.

### 3. Ranks them

A weighted 0–100 score over four terms — compliance margin, footprint economy, flume
length, and thrill — with heavy penalties for alignments that stall a rider (×0.15) or
exceed a limit you set (×0.4).

### 4. Draws, tabulates and exports

Plan and profile drawings, five strip charts along the alignment, a compliance ledger, a
segment schedule, a systems summary, a 10 ft station table, and six export formats
including three independent routes into SolidWorks.

---

## Limits are yours to enter

**The tool ships no ASTM threshold values, by design.**

Ride dynamics are computed from first principles, which is deterministic and needs no
standard. Thresholds are a different matter: they are the licensed content of ASTM F2376
and F2291. FlumePath gives you twelve criteria with blank fields and a free-text clause
reference beside each one. You type the governing numbers from your own licensed copies.

A blank limit is reported as **not set** — never as a pass. An alignment with no limits
entered is labelled "no limits set", not "compliant". The status column distinguishes all
four states: no limits set, *n*/12 checked, within limits, and *n* over limit.

The twelve criteria are maximum velocity, peak lateral g, peak normal g, minimum normal g
(the airtime floor), maximum grade, peak roll rate, peak jerk, minimum freeboard, minimum
horizontal radius, minimum sag and crest vertical radii, and maximum exit velocity into
the runout.

Whatever you enter is carried into the exported assumptions log alongside your clause
references, so the calculation package records where each number came from.

---

## Method

### Horizontal geometry

Alignments are built from four segment primitives — tangent, easement-in, circular arc,
easement-out — assembled by family recipe and walked at 1 ft steps, integrating heading
against curvature:

```
tangent       κ = 0
arc           κ = ±1/R
easement in   κ = ±t/R        (t: 0 → 1 across the segment)
easement out  κ = ±(1−t)/R
```

The linear curvature ramp is a clothoid approximation. Easement length is the main lever
on roll rate and lateral onset, which is why it is exposed as an input.

### Vertical profile

The profile is a constant-grade descent, a circular sag that rotates the grade to
horizontal, then a level runout. Grade is an **output**, not an input: the horizontal
alignment is laid out first, then the tool solves for the descent angle θ that absorbs
exactly the required drop.

```
drop(θ) = L_desc · tan θ + R_sag · (1 − cos θ)
L_sag   = R_sag · sin θ
L_desc  = L_plan − L_runout − L_sag
```

This is not monotonic — as θ steepens, the sag consumes descent length — so the solver
scans 0…55° for the peak achievable drop, rejects the candidate outright if that peak
falls short, and otherwise bisects the rising branch to 80 iterations. Demanded drop is
hit to better than 0.0001 ft.

### Dynamics

The centreline is resampled to uniform arc length, then marched:

```
v² ← v² + 2·ds·(g·sin θ − μ·g·cos θ − k·v²)
```

gravity, Coulomb friction against the wetted surface, and an optional v² drag term. A
rider whose speed collapses to the floor is flagged as **stalled**, with the station
recorded.

Curvature is the circumscribed-circle (Menger) construction over a finite stencil of the
resampled centreline, resolved into a flume-fixed frame:

```
κ = 2·|AB × BC| / (|AB|·|BC|·|AC|)

lateral g     a_lat  = v²·κ_L / g
normal g      a_norm = (v²·κ_V + g·V_z) / g
bank angle    φ      = atan2(a_lat, a_norm)
wall climb    h      = r_flume · (1 − cos φ)
freeboard     f      = h_wall − h
```

Roll rate and jerk are finite differences of bank angle and resultant specific force with
respect to time, using dt = ds/v.

### Two-pass resolution

The sweep evaluates at ds = 2 ft for speed. When you click a candidate, it is re-solved at
ds = 1 ft before the ledger, drawings and exports are built. This keeps a several-hundred
candidate sweep interactive while the alignment you are actually reading is computed more
finely.

---

## Validation

The solver core is DOM-free and was checked against closed-form solutions.

| Check | Result |
|---|---|
| Frictionless ramp, v = √(v₀² + 2gh) | −0.041 % at ds = 0.25 ft |
| Energy balance with μ = 0.05 / 0.08 / 0.11 | −0.055 % / −0.069 % / −0.092 % |
| Level tangent → normal g | 1.000000 g exactly, lateral 0, bank 0° |
| Level arc R = 30 ft at 30 ft/s → v²/gR | 0.93243 vs 0.93243, radius recovered as 30.000 ft |
| Wall climb vs r(1 − cos φ) | exact |
| Profile solver hits demanded drop | exact to 4 decimal places across 15–75 ft |

Discretization error in the velocity march grows mildly with friction, as an explicit
scheme should, and stays under 0.1 % at production step sizes.

**One metric is genuinely resolution-sensitive: roll rate.** It is a finite difference of
bank angle, itself a derived quantity, so it converges slowest. Across the same alignment:

| ds | 2.0 ft (sweep) | 1.0 ft (selected) | 0.25 ft (reference) |
|---|---|---|---|
| Peak roll rate | 180.3 °/s | 186.6 °/s | 191.3 °/s |

Every other metric moves less than 0.1 % across the same range. Treat roll rate as the
softest number in the tool, read it from the ledger rather than the candidate table, and
allow margin against it.

---

## What is not modelled

Stated plainly, and repeated in every exported assumptions log:

- **Hydrodynamic lift** — the rider is a point mass on a surface, not a hull on a film of water
- **Water depth and flow distribution** — flow demand is a per-foot-of-width allowance, nothing more
- **Banking built into the flume section** — the rider banks by riding up a circular wall; a
  pre-banked extrusion is not represented
- **Rider body dynamics** — no articulation, no rider-to-rider variation beyond the μ you set
- **Vehicle behaviour in the sag** — no suspension, no hull deformation, no submergence

The friction coefficients shipped with the rider presets (0.05 single tube through 0.11
body slide) are **starting estimates for wetted gelcoat, not standard values**. Calibrate
them against your own timing runs before you rely on any output.

---

## Known behaviour worth knowing

- The candidate table shows coarse-pass numbers until a row is selected and refined. Editing
  a ranking weight or a limit re-ranks and refreshes the row to refined values. The ledger is
  always refined.
- `checkOf` treats an infinite radius as a pass on the minimum-radius criteria, so a straight
  ramp does not fail "minimum horizontal radius" for having none.
- The plan alignment is laid out first and elevation is applied as a function of horizontal
  distance, so the true 3D arc length exceeds the plan length and the space-curve radius
  differs slightly from the nominal segment radius. The station table and exports report the
  solved 3D geometry, not the nominal.
- Crest and sag vertical radii are measured in the **flume-fixed frame**, which tilts through
  banked turns — not in the global vertical plane. So a helix reports a finite crest radius
  even though its profile only ever descends. On the shipped defaults, 117 of 118 surviving
  candidates carry a real crest; only a pure straight ramp reports none.
- The header's "rev N" is a session counter that increments on each generate, stamped into the
  assumptions log for traceability. It is not the application version.

---

## Deployment

The site is served by GitHub Pages at
<https://briancbealstack.github.io/Flume-Designer/>.

| Setting | Value |
|---|---|
| Source | Deploy from a branch |
| Branch | `claude/busy-gates-fuslp4` (the repository default branch), `/` root |
| Build | GitHub's built-in `pages-build-deployment`, no workflow file of our own |

Because the app is one static file, deployment is just the file being present: `index.html`
at the repository root *is* the site. Pushing to the branch above republishes it within a
minute or so, and `.nojekyll` tells Pages to serve the tree verbatim rather than running it
through Jekyll.

`index.html` and the downloadable copy are the same file — there is no build output and no
separate hosted variant to keep in sync.

---

## Licence and standing

Preliminary alignment study only. Not a stamped design. Not a compliance certification.
Nothing produced here substitutes for review by a qualified engineer against your own
licensed copies of the governing standards.
