# Global Orchid Pollinators

Mapping the global distribution of orchid pollinators.

Pollinator records — orchid species, locality, coordinates, pollinator type and
the pollinator taxa themselves — are compiled from published papers into an
Excel workbook and plotted on interactive maps. Work is ongoing: coordinates are
being added region by region, Americas first, then Asia.

Coordinates compiled from the primary literature by **Natalia Palou**,
**Caleb Pacheco** and **Naan Lamoso** — the three share the work equally.

The pollination records come from the global orchid pollination database compiled
by **James D. Ackerman** and colleagues, archived on Zenodo
([doi:10.5281/zenodo.7263689](https://doi.org/10.5281/zenodo.7263689), CC BY 4.0)
and published as Ackerman *et al.* (2023), *Beyond the various contrivances by
which orchids are pollinated: global patterns in orchid pollination biology*,
[Botanical Journal of the Linnean Society](https://academic.oup.com/botlinnean/article/202/3/295/7076252)
**202**: 295–324. The work in this repository adds georeferencing to that
database so the records can be mapped.

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
| `Pollination_List_MASTER.xlsx` | **Master workbook** (sheet `species`) — the raw source of truth, hand-edited by the students. Deliberately undated: it is updated in place, so the filename never changes |
| `Pollination_List_pollinators_enriched.xlsx` | **Derived** — written by the wrangling doc. Five sheets: `species`, `pollinators_long`, `needs_review`, `pollinator_taxonomy`, `taxa_review`. Do not hand-edit; regenerate it |
| `pollinators_wrangling.qmd` | Data prep. Also has an optional `rgbif` GBIF-backbone validation chunk |
| `mapa_final.qmd` | Main analysis. Static Leaflet map, one map per subfamily, and a Shiny app with a subfamily filter and a pollinator drill-down (group → family → genus). Uses `server: shiny` |
| `mapa_final_static.qmd` | Static, self-contained build of the above for sharing by email — no Shiny, one portable `.html` |
| `pollinator_map.qmd` | Standalone Shiny search map with a linked DT table |
| `coordinate_assignment_tracker.xlsx` | 14-week coordinate-collection plan for the three students, with an auto-tallying Progress tab. Also where their returned work is merged back, so it holds coordinates the master does not have yet |
| `typo_corrections_changelog.csv` | Every pollinator-name correction applied to the master — the original 105, plus the 16 Ackerman confirmed in August 2026 |
| `filldown_changelog_2026-08-19.csv` | The 4,833 `genus` and `subfamily` cells filled down on 2026-08-19, one row per cell |
| `genus_corrections_changelog.csv` | **Derived** — the GBIF-confirmed genus spellings the wrangling doc applies automatically |
| `gbif_genus_review.csv` | **Derived** — every pollinator genus checked against the GBIF backbone, with how it resolved |
| `coordinate_errors_2026-08-16.xlsx` | Coordinates flagged by the land check, for the students to re-check against the source papers |
| `coordinate_review_2026-09-10.xlsx` | Review of the first batch returned by the students: flagged records, shared coordinates, the reporting problems to fix, and a cell-by-cell log of the date and precision values that had to be repaired on merge |
| `coordinate_additions_changelog_2026-09-10.csv` | Every coordinate written into the master on 2026-09-10, with the master row, the student, and the source exactly as they gave it |
| `Resolution_documents/` | Specialist review of the disputed pollinator names — the workbook sent to J. D. Ackerman, his marked-up reply, a record of what was done with each verdict, and the follow-up note |
| `_archive/` | Superseded workbooks and backups — **local only, not in the repo** (see `.gitignore`) |
| `PROGRESS.md` | Dated georeferencing snapshots |

## Data notes

Single sheet `species`, ~3,161 rows (3,127 real species), ~69 columns.

- **Coordinates are stored as text**, and the `latitud` column mixes real
  numbers with Spanish free-text notes from the students (`no encuentro la
  flor`, `no especifica lugar`, …). Always coerce with `as.numeric()`; a naive
  type check will report hundreds of phantom differences.
- **228 rows currently have valid coordinates** (221 distinct species, 62
  genera), as of 2026-09-10. The rest await data entry — that is expected, not a
  bug.
- **Subfamilies appear in phylogenetic order everywhere**: Apostasioideae,
  Cypripedioideae, Vanilloideae, Orchidoideae, Epidendroideae. That holds for
  every table, figure, per-subfamily map and menu, and for the landing page and
  `PROGRESS.md`, so a reader moves through the family in the same sequence each
  time and can read a gap in coverage as a position on the tree. Each document
  defines `subfamily_order` and `order_subfamilies()` for it, beside the
  equivalent pair for pollinator groups; a ggplot y axis takes
  `rev(subfamily_order)` so it reads top-down.
- **Vanilloideae joined the maps on 2026-09-10**: 36 of its 60 species now have
  coordinates, from the first batch Naan and Natalia returned. Two records from
  that batch were deliberately left blank, and *Vanilla hartii* now occupies
  four rows, one per Osa Peninsula site. `PROGRESS.md` has the detail;
  `coordinate_additions_changelog_2026-09-10.csv` has the cell-by-cell record.
- **Coordinates are sanity-checked two ways.** The wrangling doc tests every
  georeferenced point against a coastline (an orchid is not a marine plant) and
  against the `locality` the record already carries (`C Am` cannot be in
  Brazil), and also flags latitude identical to longitude and coordinates
  shared across genera. Of the 171 points present in August 2026, 11 fell in
  open water and 20 contradicted their locality — 25 records once the overlap
  was removed. Those figures predate the 2026-09-10 additions and will change
  the next time the doc is run over all 228 points. Results go to the
  `coord_review` sheet of the enriched workbook. The check only reports; it
  never edits the master.
- `genus` and `subfamily` **used to be** written once per block and left blank
  on the following rows. Since 2026-08-19 both are written on every row that
  carries a species. The code still fills them down with `tidyr::fill()`
  **before** any filtering, and should keep doing so — sixteen rows are blank on
  purpose (below) and new hand-entered rows arrive blank. Row order must stay
  intact. Full binomial = `genus` + epithet.
- **Sixteen rows have neither genus nor species, and were left that way.** Nine
  are empty spacers after *Zeuxine*; the other seven carry a stray copy of the
  *next* species' pollinator text. They are still parsed as records at run time,
  so the following species' pollinator list is counted twice and one row credits
  *Calopogon barbatus*'s pollinators to *Arundina*. Known; not yet fixed.
- **Do not use `genus count` to find where a genus block starts.** Three blocks
  begin without that marker, so the running total in row 3146 is three short,
  and three genera (*Coelogyne*, *Bipinnula*, *Schizochilus*) appear in two
  non-adjacent places. Take the nearest genus written above instead.
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
- **A family is usually closed with `;` or `.` — but not always.** Some records
  use a comma or a colon, and until August 2026 the parser read those as list
  punctuation, so every taxon in the segment inherited the *next* family named.
  Ninety-five taxa carried the wrong family: skippers filed as swallowtails,
  *Lasioglossum* as apidae, hummingbirds with no family at all. `parse_poll()`
  now treats `,` and `:` as segment ends, but **only** after a parenthesis that
  actually names a family — `(non-native),` is an annotation and must not cut.
  Don't simplify that test.

## Pollinator names

Name disputes the GBIF backbone could not settle were sent to J. D. Ackerman and
arbitrated in August 2026. `Resolution_documents/` holds his marked-up workbook
and a record of what was done with each verdict; read it before revisiting any
pollinator name. Three principles came out of it and are worth stating here:

- **A synonym is not a typo, and an old family is not an error.** Corrections go
  in `genus_recode` / `family_recode`; superseded circumscriptions go in
  `family_current`. The master keeps what was published in both cases.
- **The family-consistency rule is a heuristic, not evidence.** It assumes the
  family a record was published under is correct, so when the family is wrong it
  launders the error into a name change. Ackerman rejected one of ten
  corrections on exactly that ground — *Pontia*, a pierid, would have become the
  noctuid *Polia*.
- **Suprageneric names stay suprageneric.** `Augochlorini` is a tribe and
  `Sp.` is an abbreviation; neither is promoted to a genus.

One question is still open: *Tetralona nipponensis* on *Bletilla striata*.

## Running the documents

`mapa_final.qmd` and `pollinator_map.qmd` use `server: shiny`, so they need a
live R session:

- In RStudio: **Run Document**
- From the terminal: `quarto preview mapa_final.qmd`

A plain `quarto render` will **not** work on those two — the `context: setup`
chunks never execute and you get a page with no maps. Use
`mapa_final_static.qmd` when you need a file you can send someone.

Packages: `readxl`, `openxlsx`, `dplyr`, `tidyr`, `stringr`, `purrr`, `leaflet`,
`shiny`, `DT`. Three are used only by optional checks in the wrangling doc:
`rgbif` (GBIF backbone), `maps` (coastline and country test) and `mapdata`
(high-resolution coastline). Without `mapdata` the land test falls back to a
coarse outline that omits small islands and reports island records as sea.

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

**The interactive Shiny version** — hosted on Posit Connect Cloud:

<https://raymondltremblay-global-orchid-pollinators.share.connect.posit.cloud/>

It deploys from this GitHub repo, so a push updates it. Regenerate the manifest
whenever the document or the installed packages change, then commit and push:

```r
rsconnect::writeManifest()
```

**When publishing a new piece of content, choose `Quarto` as the framework — not
`Shiny`.** This document is a Quarto document that happens to contain Shiny
elements. Picking `Shiny` makes Connect Cloud treat it as a *Python* Shiny app:
it ignores `manifest.json`, never uses R, and fails with the misleading error
"A Python requirements.txt file is required in your project". Connect Cloud
distinguishes R Quarto from Python Quarto by which dependency file it finds
(`manifest.json` vs `requirements.txt`), but only within the Quarto framework.

Two URLs, and only one of them is shareable:

- `connect.posit.cloud/<user>/content/<id>` — the owner/management view. Requires
  a login; do not send this to anyone.
- `https://<slug>.share.connect.posit.cloud` — the public share link. This is the
  one that goes in `SHINY_URL` and in emails.

Paste the share URL into the one line near the top of `docs/index.html`:

```js
const SHINY_URL = "https://raymondltremblay-global-orchid-pollinators.share.connect.posit.cloud/";
```

That activates the "Interactive explorer" card, which opens the app in a new tab.
Left empty, the card stays greyed out rather than showing a dead link. The app is
deliberately **not** embedded in the page: Connect Cloud sends frame-blocking
headers, so an `<iframe>` pointing at it renders blank.

`.rscignore` keeps any rsconnect upload down to the document plus the enriched
workbook.

Re-deploy after every `pollinators_wrangling.qmd` run — the data travels inside
the deployed bundle, it is not read live.
