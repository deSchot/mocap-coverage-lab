# MoCap Coverage Lab

Browser-based **camera coverage and occlusion analysis** for OptiTrack motion-capture rigs.

Drop in a Motive `.cal` calibration file and the tool reconstructs your cameras in 3D, then lets you analyse how well a walking loop (and a participant moving around it) is covered — and where it isn't. It also suggests where to put new cameras to fix the weak spots.

It is a **single self-contained HTML file**. No build step, no server, no dependencies to install — open it in a browser. Three.js is pulled from a CDN at runtime.

> ⚠️ **Before you rely on it:** the coverage maths is verified, but two things are environment-specific — see [Adapting it to your lab](#adapting-it-to-your-lab) and [Limitations](#limitations). Open it in your own browser and sanity-check the geometry against reality first.

---

## What it does

- **Parses Motive `.cal` files** — extracts each camera's serial, position (Motive XYZ) and optical axis directly from the binary calibration file (see [The `.cal` format](#the-cal-format)).
- **Optional `device_info.csv`** — assigns the correct model and field-of-view to each camera by serial.
- **3D scene** — camera frustums (colour-coded by FOV), the walking loop and lane edges, the measuring zone and its capture volume, room walls, an origin/axes gizmo, and per-camera labels.
- **Coverage map** — samples the loop at three editable heights and colours each point by how many cameras can see it. "Well-covered" = seen by ≥3 cameras with at least one pair of sightlines 20–160° apart (good triangulation).
- **Focus region** — restrict all stats and suggestions to the measuring-zone half of the loop (the zone plus the zone-side halves of the two bends), or use the whole loop.
- **Obstacles & occlusion** — drop *N* body cylinders into the loop; one focal body is rendered opaque with its surface coloured by how well-covered it is, accounting for self-occlusion and the other bodies blocking sightlines. Animate the bodies around the loop and re-analyse at any position.
- **Placement suggestions** — a greedy search proposes new camera positions/aims (constrained to the room) to maximise either loop coverage or the focal body's visible surface under occlusion.
- **Edit everything** — change any camera's number, model, position (X/Y/Z) and aim (pan/tilt) via fields or by dragging in 3D (grab the body to move, the white tip to re-aim). Suggested cameras are editable the same way. Reset to the loaded `.cal`, or delete cameras to test "what if".
- **Analytics pane** — a resizable split panel plots coverage vs. distance along the centreline, with the well-covered threshold and the measuring-zone / entry-exit-arc bands marked. Updates live as cameras move.

---

## Usage

1. Open `cal_coverage_tool.html` in a modern desktop browser (Chrome/Edge/Firefox).
2. Drag in your Motive **`.cal`** file (and optionally `device_info.csv`).
3. Orbit with the mouse; use the panels on the left to edit cameras, set the focus region, add obstacles, and request suggestions.
4. Click **📊 Analytics** for the coverage-vs-position plot.

Controls: left-drag orbit · scroll zoom · right-drag pan · grab a camera body to move it · grab its white tip to re-aim.

---

## Coordinate frame

Motive's right-handed, Y-up convention is used throughout:

- **X** — along the length of the loop (the straights)
- **Y** — up
- **Z** — across the width of the loop
- **origin** — at one end of the measuring zone

---

## The `.cal` format

The Motive `.cal` file is an undocumented binary. This tool locates each camera by scanning the file for a valid orthonormal 3×3 rotation matrix (nine consecutive little-endian `float64`s forming a matrix with `M·Mᵀ ≈ I` and `|det| ≈ 1`). For a matrix found at byte offset `off`:

| Field | Type | Location |
|---|---|---|
| Serial number | `uint32` LE | `off − 251` |
| Position X | `float64` LE | `off − 26` |
| Position Y (up) | `float64` LE | `off − 17` |
| Position Z (width) | `float64` LE | `off − 8` |
| Optical axis | — | `−1 ×` third column of the matrix |

The camera count is a `uint32` LE at byte 8.

**This layout was reverse-engineered from Motive 2.3.3 files.** It may differ in other Motive versions. If parsing fails, that's the first thing to check.

---

## Adapting it to your lab

Two pieces are hardcoded near the top of the script and will need editing for a different space:

- **`ROOM`** — the room footprint (an L-shaped loop area + staging area, in metres, Motive XZ). Suggestions are constrained to this; set it to your room.
- **`GEO`** (straight length, centreline radius, lane width) and the coverage heights are editable live in the UI, but the defaults assume a stadium-shaped loop.

Camera field-of-view and the usable depth range (`RMIN`/`RMAX`, default 0.5–12 m) are nominal values; adjust the `MODELS` table and constants if your hardware differs.

---

## Coverage model & assumptions

- A point is "seen" by a camera if it lies within the camera's rectangular view frustum (half-FOV horizontally and vertically) and within the usable depth range. This is a **line-of-sight / frustum-containment test**, not a full raytrace against room geometry.
- The **loop coverage map and placement suggestions ignore body occlusion** — they describe where cameras can geometrically reach.
- The **focal-body surface** is the occlusion-aware view: it accounts for the body hiding its own markers (back-facing surface) and for other bodies blocking sightlines.
- "Well-covered" requires ≥3 cameras **and** a triangulation pair separated by 20–160°.

---

## Limitations

- `.cal` parsing is tied to the Motive 2.3.3 byte layout (see above).
- The room footprint is hardcoded; the loop is assumed stadium-shaped.
- Coverage is line-of-sight; walls, trusses and fixtures are not treated as occluders (only the body cylinders are).
- FOV and range values are nominal, not measured per-unit.
- Desktop browser only; relies on a CDN for three.js (needs an internet connection on first load).

---

## Tech

Single HTML file. [three.js](https://threejs.org/) r0.160 (modules + `OrbitControls` + `DragControls`) loaded from unpkg. No build tooling.

---

## License

MIT — see `LICENSE`. *(Add your preferred license; MIT is a sensible default for a tool like this.)*

---

## Acknowledgements

The `.cal` binary layout was reverse-engineered empirically; corrections and additions for other Motive versions are welcome via issues or PRs.
