# CLAUDE.md

Guidance for Claude when working in this project.

## Project

Global Orchid Pollinators — mapping the global distribution of orchid
pollinators. Pollinator records (orchid species, locality, coordinates,
pollinator type, and the pollinator taxa themselves) are collected from
published papers into an Excel workbook and plotted on interactive maps. Work
is ongoing; coordinates are being added region by region (Americas first, then
Asia).

Authors of the analysis: Natalia Palou and Caleb Pacheco.

## Pipeline (read this first)

The project is now an all-Quarto pipeline:

1. **Master workbook** (raw data the students edit) →
2. **`pollinators_wrangling.qmd`** parses/cleans it and writes the **enriched
   workbook** →
3. **`mapa_final.qmd`** and **`pollinator_map.qmd`** read the enriched workbook
   to draw the maps.

So after the master changes (e.g. new coordinates), re-run
`pollinators_wrangling.qmd` before rendering the maps.

## Files

- `mapa_final.qmd` — main analysis (English, Quarto, `server: shiny`). Static
  Leaflet map, one static map per subfamily, and a Shiny app with a subfamily
  filter plus a **subfamily → pollinator group → family → genus** drill-down.
  Code chunks are folded (`code-fold`). Point colours double as the legend via
  coloured layer-control checkboxes. Replaces the old `mapa_final.rmd`
  (Spanish, R Markdown), which is superseded — archive it.
- `pollinators_wrangling.qmd` — data-prep (English, Quarto). Parses the free-text
  `pollinators` column into a tidy long table + wide roll-up columns, derives
  species-trait fields, cleans the subfamily column, and writes the enriched
  workbook. Includes an optional `rgbif` GBIF-backbone validation chunk
  (runs locally with internet; skips cleanly otherwise).
- `pollinator_map.qmd` — standalone Shiny search map: search points by orchid
  and by pollinator (group/family/genus), clustered/coloured/sized markers,
  all-pollinators popups, and a linked DT table.
- `Pollination_List_Thru_1 2024_merged26_08_13.xlsx` — **master workbook**
  (sheet `species`); the raw source `pollinators_wrangling.qmd` reads. It
  merged the prior main file with Caleb's coordinate additions (144 → 171
  points), restored the *Cremastra appendiculata* (Japan) record, fixed a
  *Stelis quadrifida* citation, and had 105 pollinator-name typos corrected in
  the `pollinators` column (see `typo_corrections_changelog.csv`).
- `Pollination_List_pollinators_enriched.xlsx` — **derived** workbook written by
  the wrangling doc. Five sheets: `species` (raw + `poll_*` + trait columns),
  `pollinators_long` (one pollinator taxon per row), `needs_review`,
  `pollinator_taxonomy` (present order→family→genus repository), `taxa_review`
  (likely name typos / mis-assignments). Do not hand-edit — regenerate it.
- `coordinate_assignment_tracker.xlsx` — 14-week coordinate-collection plan for
  the three students (Naan, Natalia, Caleb), with source references per species
  and an auto-tallying Progress tab.
- `Global_Orchid_Pollinators.Rproj` — RStudio project file; open this first.
- `_archive/` — superseded workbooks and backups.
- `PROGRESS.md` — dated georeferencing snapshots.

## Data

- Single sheet `species`, ~3,161 rows (3,127 real species), ~69 columns.
- Column names have inconsistent casing/trailing spaces; the code trims and
  lowercases on load, then detects columns by name pattern (order can change).
- Key columns: `species` (epithet only), `genus`, `subfamily`,
  `pollinator_type`, `pollinators` (free text), `locality`, `references`,
  `latitud`, `longitud`.
- **171 rows currently have valid coordinates (167 distinct species, 49
  genera)** — the rest await data entry. This is expected, not a bug.
- Subfamily coverage (georeferenced species): Epidendroideae 123,
  Cypripedioideae 34, Orchidoideae 6, Apostasioideae 4. **Vanilloideae has 60
  species but none georeferenced yet** — that is why it is absent from the maps.
- Pollinator group labels (full names, no contractions): Bees, Wasps, Diptera,
  Coleoptera, Lepidoptera, Hemiptera, Aves, …; "Not reported" = no pollinator
  recorded for that georeferenced record.

### Known data issues / conventions

- **Keep the master workbook as the raw source of truth**, but confirmed
  spelling typos have been corrected in it, and the wrangling doc keeps
  `order_recode` / `family_recode` maps (and a small vetted genus list) as a
  backstop. New variants: add to those maps rather than trusting free text.
- `genus` and `subfamily` are written once and left blank on following rows;
  the code fills them down (`tidyr::fill`) BEFORE any filtering, so row order
  must stay intact. Full binomial = genus + epithet.
- Subfamily junk values are cleaned in the wrangling: `2`→Epidendroideae
  (Bulbophyllum), `Orchidaceae`→Orchidoideae (Sirindhornia), `5`/`subfamily`→NA
  (blank template rows). Valid subfamilies: Apostasioideae, Vanilloideae,
  Cypripedioideae, Orchidoideae, Epidendroideae.
- The `pollinators` grammar: `ORDER:  Genus species, G. species (Family); …`.
  The parser splits orders, expands abbreviated genera, normalises `sp./spp.`,
  and recodes order/family typos. ~150 remaining name issues are flagged in the
  enriched `taxa_review` sheet; use the GBIF chunk to confirm before merging.
- The sheet has three columns literally named `notes`; `read_excel` renames the
  duplicates automatically.

## Working with the .qmd files

- They use `server: shiny`, so run with **Run Document** in RStudio (or
  `quarto preview`), not a plain static render.
- Packages: `readxl`, `openxlsx`, `dplyr`, `tidyr`, `stringr`, `purrr`,
  `leaflet`, `shiny`, `DT` (and `rgbif` only for the optional GBIF check).
- Each data file path lives in ONE place near the top of each doc — if a file
  is renamed, change just that line.
- Within a doc, the maps and the Shiny app share one dataset and one builder
  function — do not duplicate the data-prep pipeline.
- Per-subfamily maps are returned as one `htmltools::tagList` (heading + map);
  a plain for-loop does not render htmlwidgets correctly inside one chunk.

## Conventions

- User codes primarily in R.
- Documents are now **English throughout** (the published `mapa_final.qmd` and
  the wrangling doc). Keep new work English.
- Keep this CLAUDE.md and PROGRESS.md up to date as the project evolves.
