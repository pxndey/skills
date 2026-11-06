---
name: paper-plots
disable-model-invocation: false
description: Conference Ready — publication-quality matplotlib conventions for neuroscience/data figures, reverse-engineered from sd5963's plotting style. Use when creating or reviewing plot scripts, setting up rcParams, choosing palettes or colormaps, styling panels, or when the user asks for "Conference Ready", "paper-quality", "publication-style", or better-looking figures.
---

# Paper Plots — Conference Ready

Conventions distilled from sd5963's plot scripts (`/scratch/ap7151/_SYMLINKS/sd5963-paths/scripts/`, e.g. `plot_mishi_tracking_clean_all5_v1.py`, `plot_np1_velocity_full_time.py`). No style sheet needed — the look comes from a small set of habits applied consistently per script. Figures built this way are Conference Ready: clean, colorblind-safe, self-documenting, and publication-quality without further polish.

## The rcParams block

One line near the top of `main()`, never a global theme:

```python
plt.rcParams.update({'font.size': 9, 'svg.fonttype': 'none', 'pdf.fonttype': 42,
                     'axes.spines.top': False, 'axes.spines.right': False})
```

- Base font size 9–10, never the matplotlib default. Individual labels may drop to 8.
- `svg.fonttype: 'none'` and `pdf.fonttype: 42` keep text as editable/selectable TrueType in vector output.
- Top and right spines off for the clean scientific look.

## Color

- Categorical series use the **Okabe-Ito colorblind-safe palette** defined as a script-level constant:

```python
COLORS = ['#0072B2', '#E69F00', '#009E73', '#CC79A7', '#D55E00', '#56B4E9', '#9467bd', '#666666']
```

- For compared arms (e.g. two methods × two conditions), pick muted light/dark pairs of the same hue so the grouping reads instantly: `#7899b8`/`#315f8c` vs `#e69b72`/`#b84f28`.
- Near-black `#111111` for ground truth; soft slate gray `#94a3b8` for structural markers (session gaps, guides); `#c0392b` / `#f1c40f` for warning/flag overlays.
- Line plots: prediction/derived traces at lw 0.75–0.9 with `alpha=0.85`; truth at lw 0.9 full opacity.

## Heatmaps

- `magma` (or `viridis` for ordered/cyclical data), never jet or rainbow.
- Shared physical scale across all panels of one comparison figure.
- Clip to the data: `vmin, vmax = np.percentile(values, [0.5, 99.5])`; use `LogNorm` for data spanning decades.
- Mask invalid cells to neutral gray: `cmap = plt.get_cmap('magma').copy(); cmap.set_bad('#bdbdbd')`, plot with `np.ma.array(...)`.
- `rasterized=True` in `pcolormesh` so vector output stays small.

## Layout and canvas

- Wide, data-dense canvases: figsize typically (14–20) × (5–15), e.g. `(16, 11)`, `(20, 15)`.
- `constrained_layout=True` (or `fig.tight_layout(rect=(0,.02,1,.96))` when a footer is reserved).
- Per-panel titles `loc='left', fontweight='bold'` so each title sits over its own panel; figure-level `suptitle` carries subject/session/split context.
- One shared colorbar via `fig.colorbar(mesh, ax=axes, fraction=0.012, pad=0.012, extend='both')`.

## Quiet decoration

- Grids barely visible: `ax.grid(alpha=.18)` or `axis.grid(axis='y', alpha=.18)`.
- Gap/segment markers: `ax.axvline(..., color='#94a3b8', lw=.3, alpha=.35)`.
- Bars: `edgecolor='white', linewidth=.5, capsize=3` with SEM error bars, zero-line `axhline(0, color='black', lw=.8)` where the metric crosses zero.

## In-panel honesty

- Print the metric inside the panel as a small corner box:

```python
ax.text(.99, .97, f'R² = {r2:.3f} · r = {rv:.3f}', transform=ax.transAxes,
        ha='right', va='top', fontsize=8,
        bbox=dict(facecolor='white', alpha=.85, edgecolor='none'))
```

- Figure footer (`fig.text(.5,.007,...)`) states metric definitions and caveats ("velocity error, not speed error; no smoothing; gaps retained").
- Titles name the full story: dataset · subject · cell/split · what the panel shows.

## Saving

A per-script `save()` helper, three formats every time, with provenance:

```python
def save(fig, out, stem, art):
    for ext in ('svg', 'pdf', 'png'):
        p = out / f'{stem}.{ext}'
        fig.savefig(p, dpi=180 if ext == 'png' else None,
                    bbox_inches='tight', facecolor='white')
        art[p.name] = sha256(p)
    plt.close(fig)
```

- PNG dpi 150–180 (220 for detail sheets); `bbox_inches='tight'` always.
- Optionally emit a small HTML gallery index (`img{width:100%}`) listing the stems for browsing result pages.