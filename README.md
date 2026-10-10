> [!IMPORTANT]
> **Consolidated into [SurveySuite](https://github.com/crieck2010/survey-suite)**  
> This engine has been consolidated into the unified [SurveySuite monorepo](https://github.com/crieck2010/survey-suite). Active development, bug fixes, releases, and unified testing now live in `survey-suite`. This repository is preserved as an archive.

# survey-levels

A pure-Python, dependency-free **differential leveling engine** for surveying: height-of-instrument reduction of level notes, arithmetic page checks, loop misclosure against standard allowable errors, and proportional distribution of corrections.

This is module two of a larger surveying + remote sensing suite. It is engine-only on purpose — no UI, no file I/O — so it can be imported by scripts, embedded in a desktop app, or installed as a library.

## Features

- **HI reduction** — reduce level runs by the height-of-instrument method, with sideshot (intermediate sight) support
- **Page check** — arithmetic verification: ΣBS − ΣFS must equal last − first elevation
- **Loop misclosure** — computed vs. known closing elevation
- **Allowable error** — C·√K with standard first/second/third-order specs (4/8/12 mm per √km), or a custom C
- **Adjustment** — distribute misclosure proportionally to setup count so the loop closes exactly

## Installation

Requires Python 3.9+.

```bash
# From source
pip install .

# Editable install (for development)
pip install -e .
```

## Quickstart

```python
from levels import Setup, reduce_levels, page_check, loop_misclosure, allowable_misclosure, passes, adjust_loop

# Field notes: BM_A @ 100.000 m, two setups, closing back on BM_A
setups = [
    Setup(backsight_station="BM_A", backsight=1.500,
          foresight_station="TP1", foresight=0.800,
          sideshots=(("SS1", 1.000),)),
    Setup(backsight_station="TP1", backsight=1.200,
          foresight_station="BM_A", foresight=1.905),
]

pts = reduce_levels("BM_A", 100.000, setups)
for p in pts:
    print(f"{p.station:6s} {p.elevation:.3f}  ({p.role})")
# BM_A   100.000  (benchmark)
# TP1    100.700  (turning_point)
# SS1    100.500  (sideshot)
# BM_A   99.995   (turning_point)

# Arithmetic page check
closing = next(p.elevation for p in pts if p.station == "BM_A" and p.role == "turning_point")
chk = page_check(setups, 100.000, closing)
print(chk.ok)  # True — the notes add up

# Misclosure vs. allowable (third-order, 1.0 km loop)
mis = loop_misclosure(100.000, closing)          # -0.005 m
allow_mm = allowable_misclosure(1.0, "third_order")  # 12.0 mm
print(passes(abs(mis) * 1000, allow_mm))          # True — 5 mm < 12 mm

# Distribute the misclosure; the loop now closes exactly
adj = adjust_loop(pts, mis, n_setups=2)
```

## API reference

### `levels.reduction`

| Function | Description |
|---|---|
| `reduce_levels(start_station, start_elev, setups)` | HI reduction → `[LevelPoint, …]`; raises if a backsight references an unknown station |
| `page_check(setups, start_elev, end_elev)` | → `PageCheck(bs_sum, fs_sum, check, actual, ok)` |

### `levels.adjustment`

| Function | Description |
|---|---|
| `loop_misclosure(known_elev, computed_elev)` | Computed − known closing elevation |
| `allowable_misclosure(distance_km, spec="third_order")` | Allowable misclosure in **mm**: C·√K; spec is `"first_order"` (4), `"second_order"` (8), `"third_order"` (12), or a custom C |
| `passes(misclosure_mm, allowable_mm)` | True if within the limit |
| `adjust_loop(points, misclosure, n_setups)` | Proportional correction; closing point lands exactly on the known elevation |

### Models

- `Setup(backsight_station, backsight, foresight_station, foresight, sideshots=(), distance_km=0.0)` — one instrument setup; sideshots are `((station, reading), …)` and don't propagate the line.
- `LevelPoint(station, elevation, role, setup_index)` — role is `"benchmark"`, `"turning_point"`, or `"sideshot"`.

## Conventions

- Elevations and rod readings share one consistent linear unit (metres typical).
- `allowable_misclosure` always returns **millimetres**; convert your misclosure to mm before calling `passes`.
- All functions are pure (no I/O, no global state) and raise `ValueError` on invalid input (unknown backsight station, negative loop distance, unknown spec, non-positive setup count).

## Testing

No test-runner dependency — stdlib `unittest` only:

```bash
python -m unittest discover -s tests
```

13 tests covering reduction, sideshots, page checks, misclosure, allowable specs, and adjustment, built on a hand-checked two-setup loop.

## Project structure

```
survey-levels/
├── src/levels/
│   ├── __init__.py        # exports + __version__
│   ├── station.py         # Setup / LevelPoint models
│   ├── reduction.py       # HI reduction, page check
│   └── adjustment.py      # misclosure, allowable, adjust_loop
├── tests/test_levels.py
├── pyproject.toml
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## Roadmap

Part of the surveying + remote sensing suite alongside `survey-cogo` (coordinate geometry). Planned siblings: `survey-adjust` (least squares), `survey-geodesy` (datums/state plane), `survey-raster` (remote sensing), `survey-pointcloud` (LiDAR/DEM/volumes), `survey-gnss` (RINEX/baselines) — all tied together by a `survey-suite` desktop app.

## License

MIT — see [LICENSE](LICENSE).

