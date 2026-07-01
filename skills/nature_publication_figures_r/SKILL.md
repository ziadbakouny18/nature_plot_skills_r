---
name: nature_publication_figures_r
description: Use when generating or revising publication figures in R for a Nature-style manuscript.
---

# Nature Publication Figures (R)

Ported from the Python/Matplotlib/Seaborn version at
https://github.com/fyng/nature_plot_skills. Same editorial rules, R idioms.

## Output form (how every figure this skill produces must be written)

The user runs the generated code in RStudio and needs to read, inspect, and reuse
it. So the deliverable is always a plain, runnable **`.R` script**, not a snippet
or a wrapped black box.

- **Deliver a self-contained `.R` script.** It must run top to bottom on its own:
  load packages → load/define data → build the plot → run any stats → save. Put
  `source("theme_nature.R")` (adjust the path) near the top to pull in the helpers.
- **Keep it flat and linear. Minimize complicated functions.** Prefer plain
  ggplot2 + base R written out step by step. Do NOT wrap the figure logic in custom
  functions, loops, or `apply`/`purrr` machinery unless the user asks. Avoid deep
  pipe chains, metaprogramming (`!!`, `.data[[ ]]`, `do.call`), and niche packages.
  One visible statement per step beats one clever line.
- **Reuse, don't re-derive.** Use the bundled helpers (`theme_nature()`,
  `scale_color_nature()`/`scale_fill_nature()`, `nature_legend()`, `nature_size()`,
  `fdr_annotate()`, `save_nature_figure()`) instead of re-writing theme or export
  code inline. Assign the plot to a named object (e.g. `p`) so it can be reused,
  extended, and re-saved.
- **Make it inspectable in RStudio.** After each meaningful step, leave the object
  in the environment and show it: `head(df)`, `str(df)`, or `summary(fit)` on its
  own line, and print the plot object (`p`) on its own line so it appears in the
  Plots pane. Never only pipe straight into `ggsave()` without a viewable `p`.
- **Add light checks so problems surface early**, in plain readable form, e.g.
  `stopifnot(all(c("x", "y") %in% names(df)))` before plotting, and report the saved
  file (`save_nature_figure()` already prints/returns the path).
- **Label sections with `# ---- Packages ----`, `# ---- Data ----`, `# ---- Plot ----`,
  `# ---- Stats ----`, `# ---- Export ----`** headers, so RStudio's document outline
  lets the user jump around. Add a short plain-language comment above each block
  saying what it does.

## Stack

- Must use R plotting code.
- Must prefer ggplot2, built on the bundled `theme_nature()` in [theme_nature.R](theme_nature.R).
- Use `patchwork` for multi-panel composition, `ggrepel` for direct labels,
  `rstatix`/`broom` for tests, and `p.adjust(method = "BH")` for correction.
- Consider raster layers (`ggrastr`, `annotation_raster`) only when content is
  intrinsically image-based (e.g. microscopy, heatmap of very large matrices).

## Layout

- Must size figures in 1.5 inch by 1.5 inch units — use `nature_size(width_units, height_units)`
  from `theme_nature.R` to compute `ggsave()` dimensions (each unit = 1.5 in).
- Must keep total width within 5 inches and total height within 7 inches.
  `nature_size()` warns if the requested `width_units`/`height_units` exceed those caps.
- Consider 2 or 3 width units when category labels are dense or rotated.
- Consider the full width budget for dense heatmaps or multi-panel comparisons that
  would otherwise become unreadable.
- Must share one legend across subplots when the mapping is the same — build panels
  as separate ggplot objects, strip legends from all but one, and combine with
  `patchwork::plot_layout(guides = "collect")`.
- Must place legends above the plot in a single row when there are fewer than 5
  legend entries, and to the right in a single column when there are 5 or more.
  Use `nature_legend(n_entries)` from `theme_nature.R` to apply this automatically.
- Never add an overall title (`labs(title = ...)` / `ggtitle()`).
- Never add methodological descriptions inside the figure.
- Consider terse subplot subtitles (`strip.text`, via `facet_wrap(..., labeller = ...)`)
  only when panel differences are not obvious from axes and labels.

## Graph rules

- Must include 1-pt axis lines and outward tick marks. `theme_nature()` sets
  `axis.line`/`axis.ticks` to 1 pt and uses a negative `axis.ticks.length` to push
  ticks outward (ggplot2 has no native "outward ticks" option).
- Must label every axis and put units in parentheses if available, e.g.
  `labs(x = "Age (years)")`.
- Must use an accessible palette with strong contrast under color-vision deficiency —
  use `scale_color_nature()` / `scale_fill_nature()` (Okabe-Ito palette) from
  `theme_nature.R`, or `scale_*_viridis_d()`.
- Must use no gridlines, and white backgrounds with black text — this is the
  `theme_nature()` default; do not re-add `panel.grid`.
- Never use colored text to encode groups; use point shapes, linetypes, or keyline
  boxes instead (`scale_shape_manual()`, `scale_linetype_manual()`).

## Text and annotation

- Must keep all text editable in the exported file — use `save_nature_figure()`
  (vector `cairo_pdf` device), never rasterize text.
- Must prefer Arial or Helvetica. `theme_nature()` defaults `base_family = "Arial"`,
  and sourcing `theme_nature.R` auto-registers Arial via `systemfonts` if present.
  On Linux without Arial, install `ttf-mscorefonts-installer` or pass
  `base_family = "Helvetica"`/`"Nimbus Sans"`. `save_nature_figure()` auto-selects a
  vector device that embeds the font as editable text (cairo where available, else
  native macOS quartz PDF).
- Panel letters should be 8 pt bold — set via facet `strip.text` (already 8pt bold
  in `theme_nature()`) or `patchwork::plot_annotation(tag_levels = "A")` with
  `theme(plot.tag = element_text(size = 8, face = "bold"))`.
- Other text should be 5-6 pt regular — `theme_nature()` defaults `base_size = 6`.
- Never outline text (avoid `element_text(... , colour outlines)` / shadow effects).
- Never place text on busy backgrounds.
- Never allow labels to overlap each other or the plotted data — use
  `ggrepel::geom_text_repel()` / `geom_label_repel()` for any dense direct-label layout.
- Must use black text with keylines or swatches instead of colored label text.

## Statistics

- Consider whether the figure is making an inferential comparison rather than only
  showing descriptive structure.
- Must choose a statistical test that matches the design, sample pairing,
  distributional assumptions, and number of groups (e.g. paired vs unpaired
  t-test/Wilcoxon for 2 groups, ANOVA/Kruskal-Wallis + post hoc for >2 groups) —
  `rstatix` provides tidy wrappers for most of these.
- Must correct multiple hypothesis tests with FDR — use `fdr_annotate()` from
  `theme_nature.R` (wraps `p.adjust(method = "BH")`) so every comparison in the
  figure shares one correction.
- Must annotate significance in a restrained Nature-style form, using brackets or
  compact comparison marks rather than prose — `ggsignif::geom_signif()` or
  `ggpubr::stat_pvalue_manual()` with the labels from `fdr_annotate()`.
- Never add significance marks without naming the underlying test and correction
  in the caption or accompanying text.

## Export

- Must export line art, text, arrows, boxes, and scale bars as editable vector
  elements — `save_nature_figure()` uses `ggsave(..., device = cairo_pdf)`.
- Must prefer PDF or EPS for final vector export (`cairo_pdf` or `cairo_ps`).
- Must keep artwork in RGB color space — `save_nature_figure()` uses the
  `cairo_pdf`/`cairo_ps` devices, which render RGB by default.
- Must supply raster content (any embedded raster layer) at 450 dpi or higher —
  `save_nature_figure()` defaults `dpi = 450`.

## Agent behavior

- When asked for a publication figure, consider whether a simpler chart form
  communicates the claim more clearly.
- Always return the figure as a plain, runnable, well-commented `.R` script that
  follows the "Output form" section above (flat structure, reused helpers, inline
  inspection lines, section headers) so the user can open and step through it in
  RStudio.
- If no `theme_nature()`-based style function exists in the repo yet, offer to
  create/copy one (point to `theme_nature.R`) so it can be reused across figures.
- When many categories or annotations compete for space, must solve readability
  first, even if that means expanding the figure to more size units or splitting
  into panels.
- Never optimize for decorative style over legibility, editability, and export
  correctness.
