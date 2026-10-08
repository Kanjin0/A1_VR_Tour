# VR Virtual Tour — Captured Environment

A browser-based VR experience built with [A-Frame](https://aframe.io/) that lets
users move between three 360° photographic scenes via in-world portals. Each
scene includes an information panel with contextual text, and transitions are
handled with a gaze-fuse mechanism designed to be comfortable and intuitive in
head-mounted displays.

This project was developed for Exercise #1 ("Virtual Tour on Captured
Environment") of the Interaction in VR Environments course at Universidade de Coimbra - Faculty of Science and Technology.

---

## Features

- **Three 360° equirectangular scenes**
  - Scene 1 — Abandoned Slipway
  - Scene 2 — Little Paris (Eiffel Tower)
  - Scene 3 — The Sky on Fire
- **Interactive portals**
  - Glowing rings with a preview thumbnail of the destination scene.
  - Colour-coded: pink = Scene 2, orange = Scene 3, cyan = Scene 1.
  - Idle pulse animation draws attention without being distracting.
  - Hover highlight: the ring turns white and the whole portal scales up when
    the user's gaze is on it.
- **Information panels**
  - Floating panels in each scene with a title and a short description.
  - Placed directly in front of the default view for immediate readability.
- **Comfort-conscious navigation**
  - Gaze-fuse click (1.5 s) — no controller required.
  - Look-away lock: after travelling, the user must look away from *every*
    portal for an adjustable ammout of ms (currently 250) before another transition
    can be triggered. This prevents accidental double-transitions when the new scene's
    portals occupy the same world position as the old ones.
  - Invisible rectangular hitboxes behind each ring make the whole portal
    clickable, not just the thin rim.

---

## Project Structure

```
.
├── index.html                    # The entire experience (HTML + inline JS)
├── assets/
│   ├── abandoned_slipway.jpg     # 360° equirectangular image, Scene 1
│   ├── little_paris_eiffel_tower.jpg
│   └── the_sky_is_on_fire.jpg
└── README.md
```

There is intentionally no build step, no `node_modules`, and no bundler. The
whole app is a single HTML file plus three images.

---

## Running Locally

Because the page uses relative paths for the 360° images, it must be served
over HTTP rather than opened via `file://`. Any static server works:

**Python 3 (built-in):**
```bash
python3 -m http.server 8000
```
Then open <http://localhost:8000>.

**Node (if you have `npx`):**
```bash
npx serve .
```

**VS Code:** install the *Live Server* extension and click "Go Live" in the
bottom-right corner while `index.html` is open.

---

## Controls

| Action | Desktop | VR |
|---|---|---|
| Look around | Mouse drag (or pointer lock) | Move your head |
| Trigger a portal | Click on the portal | Gaze at it for 1.5 s (fuse) |
| Enter VR | Click "Enter VR" (if a headset is connected) | — |

A gaze fuse is used instead of a controller button in VR so that the
experience works on any headset without needing to know the controller layout.

---

## Design Notes

### Why gaze-fuse and not controllers
Fuse clicking is the lowest-common-denominator interaction for WebXR. It works
with a cardboard viewer, a Quest, a Vive, or a desktop browser without
conditional code. A 1.5 s fuse is short enough not to feel sluggish and long
enough to avoid accidental triggers.

### Why a look-away lock
Every scene places its portals at the same world coordinates (a wrapper entity
rotated ±50° around the camera rig). When a transition swaps the visible group,
the user's gaze is still pointing at the *position* where a portal now exists —
belonging to the new scene. Without a lock, the fuse would refill and fire
again, teleporting the user a second time with no input. The lock requires the
user to deliberately look away before another transition is allowed.

### Why the raycaster is set manually
A-Frame's `raycaster` component exposes an `objects` property that accepts a
CSS selector. In practice, when the selector referenced a group that wasn't
yet in the scene graph, it resolved to an empty array and silently fell back
to raycasting every object in the scene — including invisible portals from
other content groups. The current code bypasses the selector and assigns the
`Object3D` array directly from `group.querySelectorAll('.portal')`. This is
documented in the inline comments in `index.html`.

### Why invisible hitboxes
Raycasting a ring only succeeds when the ray hits the annulus — the middle is
empty. Users reliably aim at the centre of a target, so a thin-ring-only hit
area makes portals feel unresponsive. A transparent plane behind the ring
expands the effective click area to a comfortable 1.3 m × 1.3 m window.

---

## Assets

The three 360° photographs are equirectangular JPGs (2:1 aspect ratio).
Replacements must be the same projection; A-Frame's `<a-sky>` expects a
standard equirectangular image, not a cubemap or an HDR/EXR file.

To swap in your own scenes:
1. Drop the new JPG into `assets/`.
2. Update the corresponding `<img id="sceneN" src="...">` in `<a-assets>`.
3. Optionally update the `<a-text value="...">` of the info panel and the
   portal labels that reference the scene.

---

## Browser Support

Tested in:

- Chrome(desktop) — WebXR supported where the platform provides it

WebXR requires HTTPS (or `localhost`) in most browsers. When deploying to
GitHub Pages or any other host, make sure the URL uses `https://`.

---

## License

Coursework submission. 360° photographs are used for educational purposes
only;
