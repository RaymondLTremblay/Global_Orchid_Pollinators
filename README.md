# Global Orchid Pollinators

Mapping the global distribution of orchid pollinators.

Pollinator records — orchid species, locality, coordinates, pollinator type and
the pollinator taxa themselves — are compiled from published papers into an
Excel workbook and plotted on interactive maps. Work is ongoing: coordinates are
being added region by region, Americas first, then Asia.

Analysis by **Natalia Palou** and **Caleb Pacheco**.

## Pipeline

Everything is Quarto. Run it in this order:

```
master workbook  ──▶  pollinators_wrangling.qmd  ──▶  enriched workbook  ──▶  maps
   (raw data)          (parse / clean / derive)        (derived data)
```

1. **`pollinators_wrangling.qmd`** reads the master workbook, parses the
   free-text `pollinators` column into a tidy long table plus wide roll-up
   columns, derives the species-trait fields, cleans the subfamily column, and
   writes `Pollination_List_pollinators_enriched.xlsx`.
2. **`mapa_final.qmd`** and **`pollinator_map.qmd`** read the *enriched*
   workbook to draw the maps.

**After the master changes — new coordinates, for example — re-run
`pollinators_wrangling.qmd` before rendering any map.** The maps never read the
master directly.

## Files

| File | What it is |
|---|---|
| `Pollination_List_Thru_1 2024_merged26_08_13.xlsx` | **Master workbook** (sheet `species`) — the raw source of truth, hand-edited by the students |
| `Pollination_List_pollinators_enriched.xlsx` | **Derived** — written by the wrangling doc. Five sheets: `species`, `pollinators_long`, `needs_review`, `pollinator_taxonomy`, `taxa_review`. Do not hand-edit; regenerate it |
| `pollinators_wrangling.qmd` | Data prep. Also has an optional `rgbif` GBIF-backbone validation chunk |
| `mapa_final.qmd` | Main analysis. Static Leaflet map, one map per subfamily, and a Shiny app with a subfamily filter and a pollinator drill-down (group → family → genus). Uses `server: shiny` |
| `mapa_final_static.qmd` | Static, self-contained build of the above for sharing by email — no Shiny, one portable `.html` |
| `pollinator_map.qmd` | Standalone Shiny search map with a linked DT table |
| `coordinate_assignment_tracker.xlsx` | 14-week coordinate-collection plan for the three students, with an auto-tallying Progress tab |
| `typo_corrections_changelog.csv` | The 105 pollinator-name corrections applied to the master |
| `_archive/` | Superseded workbooks and backups — **local only, not in the repo** (see `.gitignore`) |
| `PROGRESS.md` | Dated georeferencing snapshots |

## Data notes

Single sheet `species`, ~3,161 rows (3,127 real species), ~69 columns.

- **Coordinates are stored as text**, and the `latitud` column mixes real
  numbers with Spanish free-text notes from the students (`no encuentro la
  flor`, `no especifica lugar`, …). Always coerce with `as.numeric()`; a naive
  type check will report hundreds of phantom differences.
- **171 rows currently have valid coordinates** (167 distinct species, 49
  genera). The rest await data entry — that is expected, not a bug.
- `genus` and `subfamily` are written once and left blank on the following
  rows. The code fills them down with `tidyr::fill()` **before** any filtering,
  so row order must stay intact. Full binomial = `genus` + epithet.
- Column names have inconsistent casing and trailing spaces; the code trims and
  lowercases on load, then finds columns by name pattern so the order can change
  safely.
- **Use `poll_groups`, not `pollinator_type`.** `pollinator_type` is the old
  hand-typed column and is filled in for only 99 of 3,161 rows. `poll_groups` is
  derived by the wrangling doc from the free-text `pollinators` column and
  covers 1,844.
- The `pollinators` grammar is
  `ORDER:  Genus species, G. species (Family); …`. The parser splits orders,
  expands abbreviated genera, normalises `sp.`/`spp.`, and recodes order and
  family typos. Remaining name issues are flagged in the enriched
  `taxa_review` sheet.

## Running the documents

`mapa_final.qmd` and `pollinator_map.qmd` use `server: shiny`, so they need a
live R session:

- In RStudio: **Run Document**
- From the terminal: `quarto preview mapa_final.qmd`

A plain `quarto render` will **not** work on those two — the `context: setup`
chunks never execute and you get a page with no maps. Use
`mapa_final_static.qmd` when you need a file you can send someone.

Packages: `readxl`, `openxlsx`, `dplyr`, `tidyr`, `stringr`, `purrr`, `leaflet`,
`shiny`, `DT` (and `rgbif` only for the optional GBIF check).

## Sharing

**The public site** — <https://raymondltremblay.github.io/Global_Orchid_Pollinators/>

Served by GitHub Pages from `docs/`. `docs/index.html` is a hand-written landing
page; `docs/mapa_final_static.html` is the rendered map it links to. To refresh
the site after the data changes:

```r
quarto::quarto_render("mapa_final_static.qmd")
file.copy("mapa_final_static.html", "docs/mapa_final_static.html", overwrite = TRUE)
```

then commit and push. Note that `.gitignore` excludes `/mapa_final_static.html`
at the repo root but **not** the copy under `docs/` — that anchoring is
deliberate, so don't drop the leading slash.

The summary figures on the landing page are typed in by hand, so update them
there too when the counts move.

**A single file to email** — no server, no R on the other end:

```r
quarto::quarto_render("mapa_final_static.qmd")
```

Produces one self-contained `mapa_final_static.html`. Send it on its own; the
data is baked in. The recipient needs an internet connection only because the
base map tiles stream from OpenStreetMap and Esri.

**The interactive Shiny version** — needs a host:

```r
# Run Document locally first, then:
quarto::quarto_publish_app(server = "shinyapps.io")
```

`.rscignore` keeps the upload down to the document plus the enriched workbook.
