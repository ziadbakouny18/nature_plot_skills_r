# AGENTS.md — Nature-style publication figures (R)

Entry point for coding agents (Codex, Claude Code, and anything that reads
`AGENTS.md`). This repository is an agent skill for making Nature-style
publication figures in R with ggplot2.

## When to use

When the user asks you to create or revise a publication figure in R for a
Nature-style manuscript.

## What to do

1. Read `skills/nature_publication_figures_r/SKILL.md` — it is the authoritative
   set of rules (layout, colours, fonts, statistics, export).
2. In the figure script, `source()` the helpers in
   `skills/nature_publication_figures_r/theme_nature.R` (adjust the path to wherever
   the file lives in the user's project) and reuse them instead of re-deriving style.
3. Produce output that follows the rules summarised below.

## Output rules (summary — `SKILL.md` is authoritative)

- Deliver a **self-contained, runnable `.R` script**, not a snippet: packages →
  data → plot → stats → export, top to bottom.
- **Prefer the tidyverse** (`readr`, `dplyr`, `tidyr`, `ggplot2`, `forcats`); load it
  with `library(tidyverse)`. Use short, readable pipes and break long chains into a
  few named intermediate tibbles so each can be inspected.
- **Keep it flat and simple.** No custom functions, loops, or tidy-eval
  metaprogramming (`!!`, `.data[[ ]]`) unless the user asks. Reuse the helpers:
  `theme_nature()`, `scale_color_nature()`/`scale_fill_nature()`, `nature_legend()`,
  `nature_size()`, `fdr_annotate()`, `save_nature_figure()`.
- **Make it inspectable in RStudio.** Add `glimpse(df)` / `head(df)` lines, assign
  the plot to `p` and print `p` on its own line (so it shows in the Plots pane), and
  add `stopifnot(...)` checks before plotting.
- **Section the script** with `# ---- Packages ----`, `# ---- Data ----`,
  `# ---- Plot ----`, `# ---- Stats ----`, `# ---- Export ----` headers, each with a
  short plain-language comment.

## Requirements

R with the **tidyverse** and **systemfonts** installed. Optional, only when a
figure needs them: `patchwork` (multi-panel), `ggrepel` (direct labels),
`ggsignif`/`ggpubr` + `rstatix` (significance annotation), `ggrastr` (raster layers).
Arial ships with macOS; on Linux install `ttf-mscorefonts-installer` or pass
`base_family = "Helvetica"` to `theme_nature()`.
