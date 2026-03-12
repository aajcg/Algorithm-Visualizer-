# D&C Studio — Divide & Conquer Algorithm Visualizer

An interactive, step-by-step visualizer for **Divide & Conquer algorithms**, built for Design and Analysis of Algorithms (DAA) Lab. No frameworks, no build tools — pure HTML, CSS, and vanilla JavaScript.

---

## What It Does

D&C Studio lets you watch classic algorithms run in slow motion. Each algorithm is broken into discrete **steps** that you can play, pause, scrub forward/backward, and speed up or slow down. Every step shows:

- Which **phase** is active: Divide (cyan), Conquer (purple), or Combine (emerald)
- A plain-English **explanation** of what's happening
- A live **visualization** (bar chart, matrix grid, or 2D canvas)
- The current **pseudocode line** highlighted (where applicable)

---

## Algorithms Included

| Category | Algorithm | Complexity |
|---|---|---|
| Sorting | Merge Sort | Θ(n log n) |
| Sorting | Quick Sort | Θ(n log n) avg, O(n²) worst |
| Matrix | Matrix Multiply (D&C) | Θ(n³) |
| Matrix | Strassen's Multiply | Θ(n^2.807) |
| Search & Optimization | Min & Max Finding | Θ(n) |
| Search & Optimization | Largest Subarray Sum | Θ(n log n) |
| Geometry | Closest Pair of Points | O(n log²n) |
| Geometry | Convex Hull (D&C) | Θ(n log n) |

---

## Project Structure

```
/
├── index.html                  # Single-page app shell, all HTML layout
├── app.js                      # Core controller: state, playback, rendering helpers
├── styles.css                  # All styles (light theme, design tokens, responsive)
└── algorithms/
    ├── mergeSort.js
    ├── quickSort.js
    ├── matrixMultiply.js
    ├── strassen.js
    ├── minMax.js
    ├── largestSubarray.js
    ├── closestPair.js
    └── convexHull.js
```

---

## How It Works — Architecture Deep Dive

### 1. The Module System (`app.js` + each algorithm file)

The app is built around a central singleton `AlgorithmLab`, defined as an IIFE (Immediately Invoked Function Expression) in `app.js`. It exposes a single public method — `AlgorithmLab.register(algo)` — which every algorithm file calls as soon as it loads.

```
index.html loads app.js  →  AlgorithmLab singleton created
index.html loads mergeSort.js  →  AlgorithmLab.register({ id: 'mergeSort', ... })
index.html loads quickSort.js  →  AlgorithmLab.register({ id: 'quickSort', ... })
... and so on for all 8 algorithms
```

Each registered algorithm object must conform to this interface:

```javascript
{
  id: string,              // unique key, e.g. 'mergeSort'
  name: string,            // display name
  category: string,        // 'Sorting' | 'Matrix' | 'Search & Optimization' | 'Geometry'
  usesCanvas: boolean,     // true = render to <canvas>, false = render to DOM div
  description: string,     // HTML string shown in the Overview accordion
  complexity: { ... },     // recurrence, Master's Theorem, best/avg/worst cases, etc.
  pseudocode: string,      // newline-separated pseudocode shown with syntax highlighting
  generateInput(): any,    // returns a fresh random input for the algorithm
  run(input): Step[],      // runs the algorithm, returns an array of step objects
  render(step, target),    // draws one step into a canvas context or DOM container
}
```

### 2. Step-Based Animation Engine

The core insight is that **the algorithm runs once, upfront**, producing an array of `Step` objects. Playback just walks through this array.

When you click **Play** or **Step**:

1. If no steps exist yet, `generateAndRun()` calls `algo.generateInput()` and then `algo.run(input)`, storing the result in `steps[]`.
2. A `stepIndex` pointer advances through the array.
3. `renderStep()` reads `steps[stepIndex]` and calls `algo.render(step, target)` to update the display.
4. The `tick()` function uses `setTimeout` to advance automatically during playback, respecting the speed slider.

This design means **step-back is free** — just decrement `stepIndex` and re-render. No need to "undo" anything.

```
generateInput() → run() → [step0, step1, step2, ..., stepN]
                                         ↑
                               stepIndex pointer
                               (play = auto-advance)
                               (step back = decrement)
```

### 3. How Algorithms Produce Steps

Each `run(input)` function executes the real algorithm recursively, but **pushes a step object to an array before/after each significant operation** rather than returning a value directly.

Example from Merge Sort — the divide phase:

```javascript
// Divide step
steps.push({
  phase: 'divide',
  explanation: `Divide [${lo}..${hi}] into [${lo}..${mid}] and [${mid+1}..${hi}].`,
  array: [...arr],          // snapshot of the array at this moment
  highlights: { 0: 'left-half', 1: 'left-half', 2: 'right-half', ... }
});

// Then recurse normally
mergeSort(a, lo, mid, depth + 1);
mergeSort(a, mid + 1, hi, depth + 1);
```

Each step captures **everything the renderer needs** — it's a full snapshot, not a delta. This keeps `render()` stateless and simple.

### 4. Two Rendering Modes

**Bar chart / DOM rendering** (Sorting, Search & Optimization, Matrix):

`render(step, container)` builds an HTML string and sets `container.innerHTML`. Bar heights are proportional to values; CSS classes on each bar control its color (`active`, `sorted`, `pivot`, `left-half`, `right-half`, `merging`, etc.).

**Canvas rendering** (Geometry):

`render(step, ctx, canvas)` draws directly onto a `<canvas>` element using the 2D API. The canvas is cleared each frame and redrawn from the step's point/hull data. Points are mapped from normalized [0,1] coordinates to pixel space. Color coding: cyan = left half, amber = right half, purple = strip/active, emerald = best result.

### 5. UI Layout and Panel System

The HTML in `index.html` contains all panels stacked in `.content-area`. Panels are hidden by default (`display: none` in CSS) and toggled visible by `app.js` when an algorithm is selected:

```javascript
dom.homePanel.style.display = 'none';
dom.vizPanel.style.display = 'block';
dom.detailsPanel.style.display = 'block';
dom.controlsBar.style.display = 'flex';
dom.complexityPanel.style.display = 'block';
dom.comparisonPanel.style.display = 'block';
```

Navigating back to Home reverses this.

### 6. Compare Mode

When Compare Mode is active, `generateAndRun()` finds a second algorithm from the same category and runs **both algorithms on the same input**. Two step arrays (`steps1`, `steps2`) are produced. The render loop displays both side-by-side, advancing both in lockstep (clamping the shorter array at its last frame so both finish naturally).

### 7. Complexity Panel

Each algorithm's `complexity` object is read directly into the DOM when `switchTo(id)` is called — no computation, just data display. The comparison table at the bottom is built once from all registered algorithms and is always visible for cross-algorithm reference.

### 8. Sound Engine

A Web Audio API `AudioContext` generates short tones during playback. The frequency maps to the value of the currently highlighted bar element, so sorting algorithms produce an audible "sweep" as values move into place. Sound is gated by `soundEnabled` and suspended during Compare Mode to avoid doubled output.

---

## Controls

| Control | Keyboard | Description |
|---|---|---|
| Play / Pause | `Space` | Start or pause auto-playback |
| Step Forward | `→` | Advance one step manually |
| Step Back | `←` | Go back one step |
| Reset | `R` | Clear the visualization |
| Random | — | Generate new random input and run |
| Custom | — | Enter your own comma-separated values |
| Speed slider | — | Range from 0.5× to 3× playback speed |
| Sound toggle | — | Mute / unmute the audio tones |
| Compare Mode | — | Side-by-side comparison of two algorithms in the same category |

---

## Design System

The visual language uses a **soft light theme** with CSS custom properties defined in `:root`. Three semantic colors map to the three D&C phases:

| Phase | Color | CSS Variable |
|---|---|---|
| Divide | Cyan | `--divide-color: #0891b2` |
| Conquer | Purple | `--conquer-color: #7c3aed` |
| Combine | Emerald | `--combine-color: #059669` |

Category accent colors follow the same pattern (cyan for Sorting, purple for Matrix, amber for Search, emerald for Geometry). Typography uses **Inter** for UI text and **JetBrains Mono** for code, values, and pseudocode.

---

## Browser Compatibility

Requires ES6+, CSS Grid, Flexbox, and the Canvas API. Tested on:

- Chrome 90+
- Firefox 88+
- Edge 90+
- Safari 14+

The layout is responsive: the sidebar collapses to a hamburger menu below 900px, and the complexity grid drops to a single column on mobile.

---

## Adding a New Algorithm

1. Create `algorithms/yourAlgo.js`.
2. Call `AlgorithmLab.register({ id, name, category, usesCanvas, description, complexity, pseudocode, generateInput, run, render })`.
3. Inside `run(input)`, push step objects to a local `steps` array and return it at the end.
4. Inside `render(step, target)`, use `AlgorithmLab.renderBarChart()` or `AlgorithmLab.renderMatrixViz()` for standard layouts, or draw directly to canvas for custom visuals.
5. Add a `<script src="algorithms/yourAlgo.js">` tag at the bottom of `index.html` (after `app.js`).
6. Add a nav button in the sidebar and an algo card on the home grid in `index.html`.

---

*Built for DAA Lab by Aksh Garg.*