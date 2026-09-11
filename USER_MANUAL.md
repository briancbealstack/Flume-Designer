# FlumePath user manual

Version 1.1.1

---

## Contents

1. [Before you start](#1-before-you-start)
2. [The screen](#2-the-screen)
3. [A worked example](#3-a-worked-example)
4. [The input rail, group by group](#4-the-input-rail-group-by-group)
5. [Entering your ASTM limits](#5-entering-your-astm-limits)
6. [Reading the candidate table](#6-reading-the-candidate-table)
7. [Reading the drawings and strip charts](#7-reading-the-drawings-and-strip-charts)
8. [The compliance ledger](#8-the-compliance-ledger)
9. [Segment schedule, systems and stations](#9-segment-schedule-systems-and-stations)
10. [Exporting to SolidWorks](#10-exporting-to-solidworks)
11. [Saving and reloading a project](#11-saving-and-reloading-a-project)
12. [Troubleshooting](#12-troubleshooting)
13. [Limits of the tool](#13-limits-of-the-tool)

---

## 1. Before you start

Open <https://briancbealstack.github.io/Flume-Designer/>, or open a downloaded copy of
`index.html` in any browser. Nothing to install, nothing to configure, and it behaves
identically either way — the file is entirely self-contained.

The app generates a first set of candidates immediately from its defaults, so you land on
a populated screen rather than a blank one. Those defaults are a 48 ft platform on a
140 × 140 ft site — placeholders to show you the shape of the output. Replace them with
your own numbers before reading anything into the results.

**Have your licensed ASTM copies to hand.** FlumePath computes ride dynamics but ships no
threshold values. Until you enter limits, it can rank alignments by footprint, length and
thrill, but it cannot tell you whether any of them is acceptable.

---

## 2. The screen

**Header** — running totals: total drop, candidate count, which alignment is selected, and
the session revision number.

**Left rail** — every input, in six collapsible groups, with the three action buttons at
the bottom. The first three groups are open by default because they are the ones that
change most often.

**Main sheet** — output, top to bottom, in the order you work through it: your limits, the
candidate table, drawings, strip charts, compliance ledger, segment schedule, systems
summary, station table, exports.

The layout is a single scrolling page. Nothing is hidden behind tabs.

---

## 3. A worked example

A 52 ft tower on a 160 × 120 ft pad, single and double tubes only.

1. **Elevations & site** — platform `52`, base `0`, site width `160`, site depth `120`,
   runout `35`.
2. **Flume section** — leave the 2.0 ft radius and 2.5 ft wall for now. These describe your
   extrusion; change them when you know it.
3. **Rider cases** — untick *Mat, head first* and *Body slide*. Leave single and double tube
   ticked. The header drop should read **52.0 ft**.
4. **Sweep ranges** — open the group, set radius min `20`, max `36`, step `4`. Set catalog
   radii to whatever your fabricator actually sells.
5. Click **Generate candidates**. A few hundred survive the footprint and drop gates.
6. **Enter your limits.** Scroll to the ASTM limits table and type the governing numbers
   from F2376 and F2291 — at minimum peak lateral g, minimum normal g, peak roll rate and
   minimum freeboard. Put the clause in the reference column beside each. The candidate
   table re-ranks and re-colours as you type.
7. Click the top-ranked row. The drawings, ledger and station table fill in for that
   alignment, re-solved at fine resolution.
8. Read the **compliance ledger**. Any row tagged *exceeded* tells you the computed value,
   your limit, the margin, which rider case governed and at what station.
9. If it fails, work the levers in [§12](#12-troubleshooting) — usually easement length or
   arc radius.
10. When you have an alignment you believe in, export the **assumptions log** first, then
    the geometry.

---

## 4. The input rail, group by group

### Elevations & site

| Input | Meaning |
|---|---|
| Platform elevation | Top of the start tub, in your site datum |
| Base / pool deck elevation | Bottom of the runout |
| Site width / depth available | Plan envelope. Any candidate whose bounding box exceeds it is discarded before it is ever solved |
| Runout length | Level deceleration length appended to every alignment |

Drop is platform minus base, shown live in the header. It must exceed 1 ft.

### Flume section

| Input | Meaning |
|---|---|
| Flume inside radius | Half-width of the extrusion. Drives wall climb, flow demand and surface area |
| Wall height above invert | Sets freeboard: freeboard = wall height − wall climb |
| Sag vertical radius | Radius of the circular sag that rotates the descent grade to horizontal |
| Easement (spiral) length | Length over which curvature ramps from zero into each arc |

**Easement length is your main tool.** It is the dominant lever on roll rate and on how
abruptly lateral acceleration arrives. If roll rate is failing, lengthen it before you
touch anything else.

### Rider cases

Five presets, each with a tick box and an editable friction coefficient:

| Case | Default μ |
|---|---|
| Single tube | 0.05 |
| Double tube | 0.06 |
| 4-person raft | 0.07 (off by default) |
| Mat, head first | 0.09 |
| Body slide | 0.11 |

Every enabled case is solved separately and the **worst** value per criterion is what the
ledger reports, with the governing case named. Enable only the cases your slide will
actually carry — leaving an irrelevant one ticked will govern your design for no reason.

*Dispatch velocity* is the push-off speed at the start tub. The *v² drag coefficient*
defaults to zero; leave it there unless you have data to fit it to.

> These μ values are starting estimates for wetted gelcoat. They are **not** standard
> values. Calibrate them against your own timing runs before relying on the output.

### Sweep ranges

Radius min, max and step define the arc radii swept across the helix, serpentine and
double-helix families. A wider range with a finer step gives more candidates and a slower
sweep.

*Catalog radii* is a comma-separated list of radii you can actually buy. It does not
constrain the sweep — it drives the "nearest catalog R" column in the segment schedule, so
you can see how far a candidate sits from a stock part.

### Ranking weights

Four weights, normalised against their own sum, so only their ratios matter:

| Weight | Default | Rewards |
|---|---|---|
| Compliance margin | 40 | Distance inside the limits you set |
| Footprint economy | 25 | Using less of the site envelope |
| Flume length / cost | 25 | Shorter flume |
| Thrill | 10 | Higher peak speed and lateral g |

Thrill starts low deliberately. Until you enter limits there is nothing to rank compliance
against, so the tool favours the tamest alignment that fits over the fastest one. Raise it
once your limits are in and you can see the margin you are trading away.

Editing a weight re-ranks instantly without re-sweeping.

### Systems inputs

Feed the systems summary only. They do not affect geometry, dynamics or ranking, so you can
adjust them freely after selecting an alignment: riders per vehicle, dispatch interval,
design rider weight, flume + water weight per foot, tower spacing, flow per foot of width,
pump friction allowance, and runout deceleration.

---

## 5. Entering your ASTM limits

Twelve criteria, all blank on load:

| Criterion | Unit | Direction |
|---|---|---|
| Maximum rider velocity | ft/s | maximum |
| Peak lateral acceleration | g | maximum |
| Peak normal (positive) g | g | maximum |
| Minimum normal g (airtime floor) | g | minimum |
| Maximum flume grade | deg | maximum |
| Peak roll rate | deg/s | maximum |
| Peak jerk | g/s | maximum |
| Minimum freeboard above climb | ft | minimum |
| Minimum horizontal radius | ft | minimum |
| Minimum sag vertical radius | ft | minimum |
| Minimum crest vertical radius | ft | minimum |
| Maximum exit velocity into runout | ft/s | maximum |

Type the number in the limit column and the clause you took it from in the reference
column. Both flow into the exported assumptions log.

**A blank limit is never a pass.** It is reported as *not set*, and it is excluded from the
margin score rather than counted as satisfied. The status column tells you exactly where
you stand: *no limits set* → *n/12 checked* → *within limits* or *n over limit*.

Everything re-ranks as you type. There is no apply button.

---

## 6. Reading the candidate table

Up to 120 rows, ranked by score. Click any row to select it.

| Column | Notes |
|---|---|
| Family / Configuration | Which recipe and its parameters |
| Grade | Solved descent angle, not an input |
| Length | 3D flume length |
| v max | Peak speed in mph |
| Lat g / Norm g / Roll / Freeboard | Worst case across enabled riders; red when over a limit you set |
| Footprint | Plan bounding box, W × D |
| Transit | Ride time |
| Status | The four-state tag described above |
| Score | 0–100 weighted rank |

Two things to keep in mind:

- Rows show **coarse-pass** numbers until selected. Clicking re-solves at fine resolution;
  the ledger below is always the refined figure. Editing a weight or limit refreshes the row.
- **Roll rate is the one column that shifts noticeably** between coarse and refined — around
  6 % on a typical helix, and still not fully converged at fine resolution. Read roll rate
  from the ledger, not this table, and leave margin against it.

An empty table means nothing survived both gates. The message tells you which lever to pull.

---

## 7. Reading the drawings and strip charts

**Plan** — the centreline seen from above, coloured by lateral acceleration: blue is low,
orange marks the peak, so the hardest-working part of the alignment is visible at a glance.
Grid is 25 ft, tick marks are 50 ft stations, and a scale bar sits in the corner.

**Profile** — the invert centreline against station, annotated with the solved grade.

**Five strip charts**, all plotted against station so they read against each other and
against the plan:

1. Velocity
2. Lateral acceleration (absolute)
3. Normal acceleration
4. Freeboard remaining
5. Roll rate

When you have entered the matching limit, each chart shades the violating band and
overdraws the exceeding portion of the trace in red — so you can see not just *that* a
limit is broken but *where*, and for how long.

Charts are drawn for the **representative rider**, the fastest enabled case. The ledger
covers all of them.

---

## 8. The compliance ledger

One row per criterion:

| Column | Meaning |
|---|---|
| Computed | Worst value across all enabled rider cases |
| Limit | What you entered, or *not set* |
| Margin | How much room is left — positive is inside |
| Governing rider | Which case produced the worst value |
| Station | Where along the alignment it occurred |
| Result | *within* / *exceeded* / *not set* |

If any rider stalls, a **"Rider completes the course"** row is inserted at the top with the
stall station and the cases affected. A stalling alignment is unusable no matter how it
scores elsewhere, which is why it carries the heaviest penalty in the ranking.

Use the station number to jump back to the strip charts and see the context of the failure.

---

## 9. Segment schedule, systems and stations

**Segment schedule** — the alignment as buildable pieces: tangents, easements and arcs with
length, radius, arc angle, and start and end stations. The last column gives the nearest
radius from your catalog list and the delta, green within 0.75 ft and amber beyond. This is
where you find out whether a candidate can be built from stock parts.

**Systems** — fourteen derived quantities: theoretical hourly capacity, riders on slide at
once, transit time, minimum headway, flow demand, static + friction head, water horsepower,
support bent count, vertical and lateral reaction per bent, runout required vs provided
(flagged *adequate* or *short*), exit velocity, and fibreglass surface area.

Runout required is the distance to decelerate from exit velocity to 3 ft/s at the
deceleration rate you set. If it reads *short*, lengthen the runout input and re-generate —
runout length changes the geometry, so it is not a live-update field like the rest of the
systems group.

**Station table** — every 10 ft: coordinates, grade, horizontal radius, speed in both units,
lateral and normal g, bank angle, wall climb, freeboard and roll rate. The full-resolution
version of this table is what the CSV export contains.

---

## 10. Exporting to SolidWorks

Three independent routes build the same geometry. Pick whichever suits how you work.

### Controls

| Control | Effect |
|---|---|
| Point file units | Units for the `.sldcrv` only — inches, feet or millimetres |
| Curve control points | How many points define the curve, 12–400, default 80 |
| Station point every | Spacing of the visible 3D-sketch points, default 25 ft |
| Also build a 3D sketch of station points | Adds selectable points alongside the curve |

Coincident points are stripped automatically — a repeated coordinate will kill curve and
spline creation in SolidWorks.

### Route 1 — `.sldcrv`, no scripting

Insert → Curve → Curve Through XYZ Points, then browse to the file.

**Set your document units to match the dropdown before importing.** This is the most common
mistake: a file written in inches imported into a millimetre document gives a slide 25×
too small.

### Route 2 — `.bas`, a VBA module

A plain-text VBA module. Note that a `.swp` is a *binary* VBA project container, so a text
file renamed to `.swp` will not open. Import it properly:

1. Tools → Macro → New, save an empty macro
2. In the VBA editor: File → Import File, choose the `.bas`
3. Run `BuildCurve`, or `BuildSketchSpline` for an editable 3D sketch

`BuildSketchSpline` needs a typed `Double` array, which is exactly what VBScript cannot
produce — so it exists only in this route, not in the `.vbs`.

### Route 3 — `.vbs`, standalone

Double-click to run. No macro editor. It attaches to a running SolidWorks or starts one,
uses the active part or creates one from your default template, and builds the curve with
scalar API calls only.

If Windows blocks the downloaded script: right-click → Properties → Unblock.

### Both script routes

- **Exit sketch mode before running.** Both scripts check and refuse politely if a sketch is open.
- Both require a **part** document, not a drawing or assembly.
- Both judge success by whether a feature actually appeared rather than by return values,
  because the `InsertCurveFile*` methods do not report status consistently across versions.
  If nothing is created you get the feature counts and the error, plus a suggested fallback.
- Coordinates in the scripts are **metres**, because the SolidWorks API always takes metres
  regardless of document units. This is correct and not a units bug.

### The other three exports

| Export | Contents |
|---|---|
| Full station table `.csv` | 16 columns at solver resolution, always in feet |
| Segment schedule `.csv` | The buildable-pieces table |
| Assumptions log `.txt` | See below |

### The assumptions log

Export this every time, and keep it with the geometry.

It records the alignment and its solved grade, every input value, every rider case with its
μ and whether it was enabled, **every limit as you entered it with your clause reference**,
the computed worst case per criterion with pass/fail and governing rider, any stall warning,
and a method section stating the governing equations and — explicitly — what the tool does
not model.

It is what makes the output defensible six months later, and it is the only artefact that
carries your clause references off the screen.

---

## 11. Saving and reloading a project

**Save project file** writes a JSON containing every input, your limits, your clause
references, and each rider's μ and enabled state.

**Load project file** restores all of it and re-generates candidates immediately.

Candidates and the current selection are *not* stored — they are regenerated from the
inputs, deterministically, so a reloaded project produces the same sweep.

---

## 12. Troubleshooting

**"No alignment absorbs *n* ft of drop inside this footprint."**
Nothing passed both gates. In rough order of effectiveness: widen the site envelope,
shorten the runout, or open up the radius range. A tall drop on a small pad needs tight
radii and many turns to dissipate.

**Everything shows "no limits set".**
Expected until you fill in the ASTM table. The tool is ranking on footprint, length and
thrill alone. It is not telling you anything is compliant.

**A rider case stalls.**
Friction is beating gravity somewhere — usually a long flat run-in after a shallow grade,
or a μ set too high. Check the stall station in the ledger, then either steepen the grade
(raise the platform or shorten the plan length) or re-examine your μ.

**Roll rate is failing.**
Lengthen the easement first. It is the direct lever. If that is not enough, open up the arc
radius. Also remember roll rate is the tool's softest number — see [§6](#6-reading-the-candidate-table).

**Freeboard is failing.**
The rider is climbing too far up the wall, which means lateral g is too high for your
section. Increase arc radius, reduce speed by lengthening the alignment, or raise the wall.

**Exit velocity is too high / runout reads "short".**
Lengthen the runout, increase the deceleration rate if your surface justifies it, or slow
the ride down upstream.

**The candidate table and the ledger disagree slightly.**
Expected. The table is the coarse pass until the row is selected; the ledger is always
refined. Trust the ledger.

**A script produced no feature in SolidWorks.**
Check in order: is a sketch still open, is the active document a part, and did Windows
block the file. The dialog reports the before/after feature counts and the error — if
`BuildCurve` fails, try `BuildSketchSpline`, or import the `.sldcrv` by hand.

**Fonts look wrong.**
No network access to Google Fonts. Cosmetic only; every function still works.

---

## 13. Limits of the tool

FlumePath does not model hydrodynamic lift, water depth or flow distribution, banking built
into the flume section, rider body dynamics, or vehicle behaviour in the sag. The rider is a
point mass on a surface.

The friction coefficients are estimates for wetted gelcoat, not standard values. The whole
velocity solution scales with them, so calibrate against your own timing runs before you
trust a number.

Grade is solved as a single constant descent plus one sag. There is no rolling grade, so the
profile itself never crests. Crest vertical radius is still meaningful and almost always
reports a finite value, because it is measured in the flume-fixed frame, which tilts through
banked turns — on the shipped defaults, 117 of 118 surviving candidates carry a real crest.
Only a pure straight ramp reports it as unconstrained.

This is a preliminary alignment study. It is not a stamped design and it does not certify
compliance with any standard. Nothing it produces substitutes for review by a qualified
engineer against your own licensed copies of the governing standards.
