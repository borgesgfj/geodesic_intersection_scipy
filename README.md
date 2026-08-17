# Geodesic Intersection — Schwarzschild Spacetime

A numerical simulation of null geodesics (light rays) propagating through a **Schwarzschild black hole** spacetime surrounded by a **thin material spherical shell**. The project computes the spacetime intersection curves of two light wavefronts, enabling the study of gravitational lensing, refraction at a matter shell, and the geometry of causal encounter regions in General Relativity.

---

## Physics Background

The simulation models light propagation in two distinct spacetime regions separated by a thin shell of radius $R$, constructed via the **Israel junction conditions**:

- **Exterior** (`r > R`): Schwarzschild curved spacetime with mass parameter `M`
- **Interior** (`r < R`): flat Minkowski spacetime with a time dilation factor imposed at the boundary

Two light sources (lamps) emit fans of rays. Each ray is numerically integrated through the full manifold. The code then finds all spacetime events where rays from different sources coincide — tracing out the **leading wavefront intersection boundary**, a curve with direct physical meaning in terms of signal arrival times and optical caustics.

---

## Repository Structure

```
geodesic_calc_scipy/
├── geodesic_intersection_v1.ipynb   # Main notebook (all code)
├── requirements.txt                 # Python dependencies
├── pyproject.toml                   # Linting config (ruff)
├── .gitignore
└── data/                            # Local CSV output (git-ignored)
    └── intersections/
        └── <run_label>.csv
```

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/borgesgfj/geodesic_intersection_scipy.git
cd geodesic_intersection_scipy
```

### 2. Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate      # Linux / macOS
# .venv\Scripts\activate       # Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Register the kernel with Jupyter (VS Code)

```bash
python -m ipykernel install --user --name=geodesic-venv --display-name "Python (geodesic)"
```

### 5. Open the notebook

Open `geodesic_intersection_v1.ipynb` in **VS Code** (recommended) or JupyterLab.
Select the **"Python (geodesic)"** kernel.

> **Note:** The notebook uses `%matplotlib widget` for interactive plots, which requires the `ipympl` package (included in `requirements.txt`).

---

## Notebook Structure

The notebook is organized as a linear pipeline of self-contained cells. Run them **top to bottom** for a full simulation.

### Cell 0 — Imports

Loads `numpy`, `scipy`, `matplotlib`, and sets the interactive plot backend via `%matplotlib widget`.

---

### Cell 1 — Physics Engine

Defines the spacetime geometry classes:

| Class | Role |
|---|---|
| `SchwarzschildSpacetime` | Exterior curved spacetime. Encodes the Schwarzschild null geodesic ODE and integrates rays with `solve_ivp` (RK45). Stops rays that reach the event horizon (`r ≤ 2.001M`). Returns `(t, r, φ)`. |
| `MinkowskiSpacetime` | Interior flat spacetime. Propagates rays with centrifugal force only (no gravity). Applies a constant time dilation factor to match the shell boundary. |
| `SpacetimeManifold` | Composite geometry using the Israel junction condition. Chains up to three integration segments per ray: **exterior inward → interior crossing → exterior outward**. Handles refraction of `pᵣ` at the shell boundary using `refract_inward` / `refract_outward`. |
| `LightSource` | Defines a lamp at polar position `(r₀, φ₀)`. Computes initial conditions `(y₀, b)` from a given emission angle — the impact parameter `b` and initial radial momentum `pᵣ₀`. |

---

### Cell 2 — `WavefrontSimulator`

The orchestrator class. Owns the manifold and manages ray tracing and analysis for all light sources.

| Method | Description |
|---|---|
| `trace(lamp, angles, stride)` | Fires rays from a lamp across all emission angles, converts polar to Cartesian, stores data internally keyed by `lamp.label`. |
| `find_intersections(label1, label2, eps_t, eps_s)` | Searches for spacetime coincidences between two stored wavefronts. Uses binary search (`np.searchsorted`) on the time axis followed by a vectorized Euclidean distance filter. Returns intersection points and the death times of each consumed ray. |
| `get_rays(label)` | Returns the per-ray trajectory dict `{ray_id: {t, x, y}}` for plotting. |
| `get_data(label)` | Returns the raw spacetime array `(t, x, y, ray_id)`. |
| `get_lamp(label)` | Returns the `LightSource` object for a given wavefront. |
| `labels()` | Lists all currently stored wavefront labels. |

---

### Cell 3 — `# SIMULATION`

**Configure and run your simulation here.**

Sets the black hole mass `M`, shell radius `R`, lamp positions, emission angle wedges, and number of rays. Instantiates a `WavefrontSimulator` and traces both wavefronts. Intersection search is run at the end.

Key parameters to experiment with:
- `M` — black hole mass (sets the unit scale)
- `R` — shell radius (must be `> 2M`, the Schwarzschild radius)
- `lamp.r0`, `lamp.phi0` — lamp position in polar coordinates
- `angles_lampN` — emission angle range (in radians)
- `total_number_of_rays` — resolution of the wavefront fan
- `eps_t`, `eps_s` — time and spatial tolerance for the intersection search

---

### Cell 4 — `# PLOTTING`

Plots the current simulation run:
- 🟠 **Orange rays** — Lamp 1 wavefront trajectories
- 🔵 **Cyan rays** — Lamp 2 wavefront trajectories
- 🟣 **Magenta dots** — spacetime intersection points (encounter curve)
- 🔴 **Dashed red circle** — material spherical shell at `r = R`
- ⭐ **Stars** — lamp positions

---

### Cell 5 — `# EXPORT`  *(optional — run manually)*

Saves the current run's intersection points to a CSV file in `data/intersections/`. The filename is **auto-generated** from the simulation parameters:

```
data/intersections/R3.99_lamp1-r4.0-phi0_lamp2-r4.5-phi0_rays100.csv
```

Each CSV has two columns: `x, y`.
The `data/` folder is git-ignored — files are local only.

---

### Cell 6 — `# ENCOUNTER CURVES`

Multi-run comparison plot. Edit `files_to_plot` to select which saved CSV files to overlay:

```python
plot_config = {
    "xlim": (-10, 10),   # or None for auto
    "ylim": (-2, 12),
    "figsize": (10, 10),
    "title": "Encounter Curves — Multi-Parameter Comparison",
}

files_to_plot = [
    {
        "file": "R3.99_lamp1-r4.0-phi0_lamp2-r4.5-phi0_rays100.csv",
        "color": "magenta",
        "marker": "o",
        "linestyle": "--",
        "markersize": 4,
    },
    {
        "file": "R2.99_lamp1-r4.5-phi0_lamp2-r5.0-phi0_rays100.csv",
        "color": "cyan",
        "marker": "s",
        "linestyle": ":",
        "label": "Smaller shell",  # optional: overrides the auto label
    },
]
```

Any key beyond `"file"` and `"label"` is passed directly to `matplotlib`'s `ax.plot()`.

---

### Cell 7 — Legacy (Pure Schwarzschild, no shell)

A standalone simulation using `SchwarzschildSpacetime` only — no material shell, no refraction. Useful for comparing pure Schwarzschild lensing against the shell refraction case. Also uses `WavefrontSimulator`.

---

## Typical Workflow

```
1. Edit Cell 3   → set M, R, lamp positions, angles, rays
2. Run Cell 3    → trace wavefronts and find intersections
3. Run Cell 4    → visualize the current run
4. Run Cell 5    → (optional) save intersection data to CSV
5. Run Cell 6    → (optional) compare multiple saved runs
```

---

## Dependencies

| Package | Purpose |
|---|---|
| `numpy` | Array operations, numerical data |
| `scipy` | ODE integration (`solve_ivp`, RK45), spatial indexing |
| `matplotlib` | 2D plotting |
| `ipympl` | Interactive `%matplotlib widget` backend in VS Code |
| `ipywidgets` | Notebook widget support |
| `ipykernel` | Jupyter kernel registration |

---

## Linting

The project uses [ruff](https://docs.astral.sh/ruff/) for linting and formatting. Configuration is in `pyproject.toml`. Scientific naming conventions (`M`, `G`, `b`) are explicitly allowed.

```bash
ruff check .
ruff format .
```
