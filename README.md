# nature_plot_skills_r

An **R / ggplot2 port** of [`fyng/nature_plot_skills`](https://github.com/fyng/nature_plot_skills)
by [Feiyang Huang](https://github.com/fyng) — an agent skill for generating
publication figures in a Nature-compatible style.

The original ships prompt assets for **Python + Matplotlib + Seaborn**. This
repository keeps the same editorial rules (layout units, legend placement,
colorblind-safe palette, editable vector text, FDR correction, export
resolution, RGB color space) and re-expresses them for **R + ggplot2**, with a
reusable `theme_nature()` implementation.

## Attribution

This is a derivative work. All credit for the original skill design and the
Nature-style editorial rules goes to **Feiyang Huang**. The original project is
MIT licensed; that license and copyright notice are preserved in [LICENSE](LICENSE).

- Original: https://github.com/fyng/nature_plot_skills (Python/Matplotlib/Seaborn)
- This port: R/ggplot2, by Ziad Bakouny

## What's here

```
skills/nature_publication_figures_r/
├── SKILL.md         # the Nature-style rules, retargeted to R idioms
└── theme_nature.R   # reusable ggplot2 theme + helper functions
```

`theme_nature.R` provides:

| Function | Purpose |
| --- | --- |
| `theme_nature()` | ggplot2 theme: Arial default, 6 pt text, 1 pt outward ticks, no gridlines, white/black |
| `scale_color_nature()` / `scale_fill_nature()` | Okabe–Ito colorblind-safe palette |
| `nature_legend(n_entries)` | Legend above (<5 entries) or right (≥5), per the rules |
| `nature_size(w, h)` | Figure size in 1.5 in layout units, warns past the 5×7 in caps |
| `fdr_annotate(results)` | BH/FDR-correct p-values and attach compact `*`/`**`/`***` labels |
| `save_nature_figure(p, file, ...)` | Export editable vector PDF/EPS in RGB at ≥450 dpi, auto-selecting a working device |

## Requirements

- R with **ggplot2** (required) and **systemfonts** (for Arial registration).
- Optional, used by specific rules: `patchwork` (multi-panel), `ggrepel`
  (direct labels), `ggsignif`/`ggpubr` + `rstatix` (significance annotation),
  `ggrastr` (raster layers).
- **Arial**: ships with macOS. On Linux install `ttf-mscorefonts-installer`, or
  pass `base_family = "Helvetica"` / `"Nimbus Sans"` to `theme_nature()`.

## Usage

### As a Claude Code / agent skill

Copy the skill into your agent's skills directory. For Claude Code:

```bash
cp -r skills/nature_publication_figures_r ~/.claude/skills/
```

The skill then auto-loads by description; ask your agent for a "Nature-style"
figure and it will apply the rules and source `theme_nature.R`.

### Directly in R

```r
source("skills/nature_publication_figures_r/theme_nature.R")

p <- ggplot(df, aes(wt, mpg, colour = group)) +
  geom_point() +
  scale_color_nature() +
  theme_nature() +
  nature_legend(n_entries = 3) +
  labs(x = "Weight (1000 lb)", y = "Efficiency (mpg)")

save_nature_figure(p, "figure.pdf", width_units = 2, height_units = 2)
```

## License

MIT — see [LICENSE](LICENSE). Original copyright © 2026 Feiyang Huang; R port
© 2026 Ziad Bakouny.
