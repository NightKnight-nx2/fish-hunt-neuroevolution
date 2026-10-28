# fish hunt · neuroevolution

A single-file (`index.html`), browser-only, humorous/educational **neuroevolution** simulation. A predator fish controlled by a tiny neural network learns to hunt prey fish through a **genetic algorithm** — with zero human intervention. The scene narrates the predator's evolution from "clueless and aimless" to "apex predator" as a time-lapse, framed with a tongue-in-cheek "Level 1/6 · blind" style narrative.

> This project is an experiment in the short-video, algorithm/AI-explainer format inspired by the Instagram account [**@reelgorithm.py**](https://www.instagram.com/reelgorithm.py). The format idea comes from there; the code and content in this repository are original.

## Live demo

Just open `index.html` directly in a browser — no build step, no dependency install.

- **Simplest:** double-click the file / drag it into your browser.
- **Recommended** (some browsers restrict Web Workers over `file://`): serve it locally:
  ```bash
  # from the project folder
  python3 -m http.server 8000
  # then open: http://localhost:8000
  ```
  or
  ```bash
  npx serve .
  ```

### Controls / URL parameters
- `SPACE` → restart the timeline
- `R` → toggle 9:16 ↔ 1:1 aspect ratio
- `F` → fullscreen
- `?ratio=1x1`, `?seed=7`, `?gens=60` → set aspect ratio, RNG seed, and total generation count via the URL

## How it works (technical summary)

- **Rendering / loop:** [p5.js](https://p5js.org) `1.9.4` draws to a `canvas`; the main loop (`setup`/`draw`) drives the scene flow (levels, time-lapse montages, HUD).
- **Training:** The simulation and neural-network logic (the `Core` module inside the `#core` script tag) runs identically on the main thread and inside a **Web Worker**; training runs continuously in the background and streams generation results back via `postMessage` (falling back to a `setTimeout` loop if the Worker fails to start).
- **Perception:** The predator's "brain" takes the relative angle and proximity of the `K=3` nearest prey as input (6 inputs total).
- **Neural network:** A simple **feedforward** network — `input (6) → hidden layer (8, tanh) → output (2, tanh)`. The outputs drive the predator's turn rate and speed. The live weight count is shown on screen (`Core.genomeSize`).
- **Genetic algorithm:**
  - Population: 50 individuals, each a genome (weight array)
  - Fitness: prey eaten × 10 + survival time (seconds)
  - Selection: tournament selection (`tourn=4`)
  - Elitism: the top 3 individuals carry over unchanged
  - Crossover: neuron-wise — each hidden neuron's input weights are inherited from a single parent
  - Mutation: Gaussian noise, `mutRate=0.07`, `mutSigma=0.22`
  - Immigrants: 3 fully random individuals injected each generation to escape local optima
  - Stagnation reset: if progress stalls by certain generations (16, 30), the population is reseeded from scratch
- **Scene structure:** A 6-level narrative is built from the training history (`hist[]`) — each level picks the first generation whose best fitness crosses a threshold (`thr`) as its "champion": `blind → twitchy → curious → hungry → locked on → apex`. Fast-forwarded "evolution" montages play between levels.

## Screenshot / GIF

No screenshot or GIF is included in this repository at this time.

## References

| Dependency | Purpose | License | Link |
|---|---|---|---|
| [p5.js](https://p5js.org) `1.9.4` | Canvas rendering, animation loop (`setup`/`draw`), input handling | LGPL-2.1 | [Source](https://github.com/processing/p5.js) · [License page](https://archive.p5js.org/copyright.html) |

### License note (p5.js vs. this project's license)

p5.js is loaded at runtime as a separate library via a single `<script src="...">` tag from **cdnjs**. Its source is **not embedded, not statically linked, and not modified** in this repository — the project code only calls p5.js's public/global API (p5 global mode).

This is exactly the usage pattern the LGPL (Lesser/Library GPL) carves an exception for: a library used unmodified, as a separate component (dynamic linking / a separate file), rather than statically embedded into a derivative work. The LGPL's copyleft obligations apply to modifications of the library **itself** and to works that statically embed it — not to a separate program that merely calls it. Since this project neither modifies p5.js nor embeds its source, **applying the MIT license to our own code (this repo's `index.html`) does not conflict with the LGPL.**

Bottom line: this project's own code is **MIT**-licensed (see `LICENSE`); the p5.js library remains under its own **LGPL-2.1** license, loaded as a separate CDN dependency.

## License

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

This project's own code is licensed under the [MIT License](LICENSE). © 2026 Musa Buğra Demir.
