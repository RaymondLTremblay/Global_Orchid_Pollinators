# CLAUDE.md

Guidance for Claude when working in this project.

## Project

Global Orchid Pollinators — mapping the global distribution of orchid
pollinators. Pollinator records (orchid species, locality, coordinates,
pollinator type, and the pollinator taxa themselves) are collected from
published papers into an Excel workbook and plotted on interactive maps. Work
is ongoing; coordinates are being added region by region (Americas first, then
Asia).

Authors (the `author:` line in every .qmd): Raymond L. Tremblay, Natalia Palou,
Caleb Pacheco and Naan Lamoso.
Georeferencing: Natalia Palou, Caleb Pacheco and Naan Lamoso. All three are
students adding lat/long, and they contribute EQUALLY -- always credit them as
one group, never split them into tiers.
Dates: the .qmd files use `date: today`, so each render stamps itself. Do not
hard-code a date back in.

Source database: compiled by James D. Ackerman and colleagues, archived on Zenodo
(doi:10.5281/zenodo.7263689, CC BY 4.0) and published as Ackerman et al. (2023),
Bot. J. Linn. Soc. 202: 295-324. This project adds the georeferencing. Cite the
Zenodo record and the paper in anything derived from these data.

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

- The popup carries `compatibility` and `breeding_system` on **one** line, via
  `join_traits()` beside `trait_row()`: SI/SC and chasmogamous / mixed mating /
  autonomous selfing are one statement to a reader, and the popup was already
  seven lines. `evidence for selfing` and `evidence for reward` are still
  unused: both hold codes 1 and 2 whose meaning is not documented here, and
  `is_present()` only accepts 1, so a 2 would vanish silently. Ask Raymond for
  the Ackerman et al. 2023 definitions before wiring them in; do not guess.
- `mapa_final.qmd` — main analysis (English, Quarto, `server: shiny`). Static
  Leaflet map, one static map per subfamily, and a Shiny app with a subfamily
  filter plus a **subfamily → pollinator group → family → genus** drill-down.
  Code chunks are folded (`code-fold`). Point colours double as the legend via
  coloured layer-control checkboxes. Replaced the old `mapa_final.rmd`
  (Spanish, R Markdown), now in `_archive/`.
- `pollinators_wrangling.qmd` — data-prep (English, Quarto). Parses the free-text
  `pollinators` column into a tidy long table + wide roll-up columns, derives
  species-trait fields, cleans the subfamily column, and writes the enriched
  workbook. Includes an optional `rgbif` GBIF-backbone validation chunk
  (runs locally with internet; skips cleanly otherwise).
- `pollinator_map.qmd` — standalone Shiny search map: search points by orchid
  and by pollinator (group/family/genus), clustered/coloured/sized markers,
  all-pollinators popups, and a linked DT table.
- `Pollination_List_MASTER.xlsx` — **master workbook**
  (sheet `species`); the raw source `pollinators_wrangling.qmd` reads. Renamed from `Pollination_List_Thru_1 2024_merged26_08_13.xlsx` on
  2026-08-16; the name is undated on purpose, since the file is edited in place.
  It merged the prior main file with Caleb's coordinate additions (144 → 171
  points), restored the *Cremastra appendiculata* (Japan) record, fixed a
  *Stelis quadrifida* citation, and had 105 pollinator-name typos corrected in
  the `pollinators` column (see `typo_corrections_changelog.csv`).
- `Pollination_List_pollinators_enriched.xlsx` — **derived** workbook written by
  the wrangling doc. Five sheets: `species` (raw + `poll_*` + trait columns),
  `pollinators_long` (one pollinator taxon per row), `needs_review`,
  `pollinator_taxonomy` (present order→family→genus repository), `taxa_review`
  (likely name typos / mis-assignments). Do not hand-edit — regenerate it.
- `private/coordinate_assignment_tracker.xlsx` — 14-week coordinate-collection plan for
  the three students (Naan, Natalia, Caleb), with source references per species
  and an auto-tallying Progress tab. This is also where their returned work is
  merged back, so between merges it holds coordinates the master does not have.
  See "Student coordinate returns" below before merging a new batch.
- `private/coordinate_additions_changelog_2026-09-10.csv`: every coordinate written into
  the master on 2026-09-10, with the master row, the student, and the source
  exactly as given. The pre-write master is in `_archive/`.
- `private/coordinate_additions_changelog_2026-09-15.csv` and
  `private/coordinate_review_2026-09-15.xlsx`: the same pair for the second batch. The
  changelog also lists the three coordinates held back (`HELD`).
- `private/` (in `.gitignore`, never pushed): everything that names the
  students or reports on their work, because the repo is public. It holds the
  tracker, the batch reviews, the coordinate changelogs, the coordinate error
  list and the `correo_*.md` messages to the students (Spanish). New files of
  that kind go here.
- `private/coordinate_review_2026-09-10.xlsx`: review of the first returned batch,
  flagged records, shared coordinates, the reporting problems to raise with the
  students, and a log of every date and precision cell repaired on merge.
- `Global_Orchid_Pollinators.Rproj` — RStudio project file; open this first.
- `_archive/` — superseded workbooks, old documents and backups. Local only:
  `.gitignore` keeps it out of the repo. `mapa_final.rmd` (the original Spanish
  R Markdown) lives here now.
- `PROGRESS.md` — dated georeferencing snapshots.

## Data

- Single sheet `species`, ~3,161 rows (3,127 real species), ~69 columns.
- Column names have inconsistent casing/trailing spaces; the code trims and
  lowercases on load, then detects columns by name pattern (order can change).
- Key columns: `species` (epithet only), `genus`, `subfamily`,
  `pollinator_type`, `pollinators` (free text), `locality`, `references`,
  `latitud`, `longitud`.
- **245 rows currently have valid coordinates (233 distinct species, 63
  genera)**, as of 2026-09-15 — the rest await data entry. This is expected, not
  a bug.
- **Subfamilies are always presented in phylogenetic order**: Apostasioideae,
  Cypripedioideae, Vanilloideae, Orchidoideae, Epidendroideae. That order holds
  in every table, figure, map series, menu and list, here and in the .qmd files,
  the landing page and `PROGRESS.md`. Never fall back to alphabetical or to
  descending count. Each .qmd defines `subfamily_order` and `order_subfamilies()`
  next to `group_order` / `order_groups()`; use those rather than `sort()`, and
  remember a ggplot y axis needs `rev(subfamily_order)` to read top-down.
- Subfamily coverage (georeferenced species): Apostasioideae 4,
  Cypripedioideae 34, Vanilloideae 37, Orchidoideae 12, Epidendroideae 146.
  **Vanilloideae was absent from every map until 2026-09-10**, when Naan's first
  batch gave it 36 of its 60 species.
- A species with more than one locality gets **one row per locality, and only
  the last row of the block carries the full record** (traits, `locality`,
  `references`); the rows above it carry subfamily, genus, species and the
  coordinates alone. *Cypripedium passerinum*, *Phragmipedium lindenii*, *Vanilla hartii* (2026-09-10) and, as of
  2026-09-15, *Acianthera johannensis*, *A. ochreata*, *Coryanthes speciosa* and
  *Elleanthus crinipes* are built this way. Follow it: duplicating a
  full record instead would double-count that species' trait flags in the
  summary block at the foot of the sheet. `pollinators_wrangling.qmd` puts the
  record back together at load time (see "Multi-locality blocks" in the `load`
  chunk): within a run of consecutive rows for one species, the richest row is
  the record and every other row **carrying coordinates** inherits its empty
  cells, 78 cells across those three species. The coordinate test is what keeps
  the continuation rows out, the ones holding nothing but a second
  `Pollinator_type` (*Bletilla striata* 80 and 81, *Coelogyne rigida* 97,
  *Calanthe alismaefolia* 115, *C. argento-striata* 118). Those are extra
  pollinator types for the record above, not extra localities, and filling them
  would duplicate records in `pollinators_long`.
- **The summary block (rows ~3154 onward since 2026-09-15) holds 103 formulas
  whose ranges are written out in full** (`=SUM(D2:D3152)`). openpyxl does not shift them when
  rows are inserted, so re-point them by hand after any insertion, then recalc.
- Pollinator group labels (full names, no contractions): Bees, Wasps, Diptera,
  Coleoptera, Lepidoptera, Hemiptera, Aves, …
- **"No pollinator in the source"** is the last group label, renamed from "Not
  reported" on 2026-09-10 because that read as a gap in data entry. It is not.
  Those records are in the database for a different reason: their paper studied
  breeding system or reward and never named a visitor. Of 228 mapped points, 62
  were of this kind, and **not one carries the `Pltr data` flag**, which is the
  tell. *Cypripedium dickinsonianum* (Hágsater 1984) is an autonomous-selfing
  record; the whole Cardoso-Gustavson et al. 2018 block of *Epidendrum* and
  *Encyclia* is nectary anatomy. Do not treat these as missing data to chase.
- A record whose free text names a group but no binomial is a **different**
  case and is not in that bucket: 47 of the 166 mapped records with text give a
  genus and `sp.`, a family (`DIPTERA: Drosophilidae`) or, in about four cases,
  only the order. They keep their group and their colour.

### Known data issues / conventions

- **Keep the master workbook as the raw source of truth**, but confirmed
  spelling typos have been corrected in it, and the wrangling doc keeps
  `order_recode` / `family_recode` / `genus_recode` maps (and a small vetted
  genus list) as a backstop. New variants: add to those maps rather than
  trusting free text.
- `genus_recode` holds corrections from two sources and no others: those GBIF
  confirmed without inference (both spellings resolving to one accepted name, or
  one placed and the other unplaceable), and those **J. D. Ackerman arbitrated
  on 2026-08-18**, marked `# JDA`. Corrections resting on the
  family-consistency rule are never applied unqualified — that rule assumes the
  recorded family is right, and Ackerman rejected one of the ten on exactly that
  ground (*Pontia*, not *Polia*). Suprageneric and informal names
  (`Anthophorid`, `Halictid`, `Meliponids`, and `Augochlorini` at tribe rank via
  `SUPRAGENERIC`) are excluded on purpose: promoting them to a genus asserts
  precision the source never gave.
- **Two family maps, and the distinction matters.** `family_recode` = the author
  mistyped (`Aipdae` → Apidae). `family_current` = the author was right at the
  time and the classification has since moved (`Ctenuchidae` → Erebidae, per
  Ackerman). Never merge them; the second is not an error report. Six further
  old circumscriptions (Arctiidae, Danaidae, Satyridae, Eumenidae, Otitidae,
  Coerebidae) are deliberately left alone pending his ruling.
- `genus` and `subfamily` **used to be** written once per block and left blank
  on following rows. As of 2026-08-19 they are filled in on every row that
  carries a species (2,111 genus + 2,722 subfamily cells; see
  `filldown_changelog_2026-08-19.csv`). **Keep the `tidyr::fill` in the wrangling
  doc anyway** — sixteen rows deliberately still have no genus (see below), and
  new hand-entered rows will arrive blank. Row order must still stay intact.
  Full binomial = genus + epithet.
- Sixteen rows have neither genus nor species and were left blank on purpose:
  rows 2081–2089 are empty spacers after *Zeuxine*, and rows 31, 32, 50, 51,
  60, 61, 75 each carry a stray copy of the **next** species' pollinator text.
  Those seven are still parsed as records at run time, which double-counts the
  following species' pollinator list and, for row 75, credits *Calopogon
  barbatus*'s pollinators to *Arundina*. Known, not yet fixed.
- Do not use the `genus count` column as the genus-block marker. Three blocks
  start without it (*Bipinnula* row 113, *Ceratostylis* 1027, *Porpax* 1062), so
  the running total in row 3146 is short by three. Three genera are also split
  across non-adjacent blocks — *Coelogyne* (83–100 and 187), *Bipinnula* (113
  and 2018), *Schizochilus* (2942–2945 and 2958–2961) — which is why 426
  markers, 428 name runs and 425 distinct genera all disagree. Fill down from
  the nearest genus written above instead; that is correct in every case.
- Subfamily junk values are cleaned in the wrangling: `2`→Epidendroideae
  (Bulbophyllum), `Orchidaceae`→Orchidoideae (Sirindhornia), `5`/`subfamily`→NA
  (blank template rows). Valid subfamilies: Apostasioideae, Cypripedioideae,
  Vanilloideae, Orchidoideae, Epidendroideae. The two junk cells that sit in
  the data (row 757 `2`, row 2962 `Orchidaceae`) were **left as written** by the
  fill-down and were not used as a fill source, so neither propagated down its
  block. Keep the recode — do not "tidy" those two cells away, since the recode
  is the record that they were wrong.
- The `pollinators` grammar: `ORDER:  Genus species, G. species (Family); …`.
  The parser splits orders, expands abbreviated genera, normalises `sp./spp.`,
  and recodes order/family typos. ~150 remaining name issues are flagged in the
  enriched `taxa_review` sheet; use the GBIF chunk to confirm before merging.
- **The colon after an ORDER is optional** (fixed 2026-09-10). It used to be
  required, and 34 cells written without one lost data three different ways: a
  cell holding only `DIPTERA` fell through as `informal` with no order, so the
  record read as having no pollinator at all (*Plocoglottis porphyrophylla*,
  *Stelis hymenantha*); `COLEOPTERA Astylus trifasciatus (Melyridae).
  HYMENOPTERA: …` dropped the beetle outright, since everything before the
  first colon-marked order was ignored (*Bipinnula fimbriata*); and a leading
  `DIPTERA. HYMENOPTERA: …` dropped the flies the same way. Making it optional
  is safe **because every UPPERCASE run of four or more letters in this column
  is an order name**, verified across all 3,164 rows, taxa and families being
  Capitalised rather than upper case. If that ever stops being true, the marker
  has to go back to a known-order list rather than back to requiring the colon.
- An order named with nothing after it now yields a row of `rank = "order"`
  rather than being skipped. "No pollinator in the source" is a claim about the
  source and must stay reserved for records that make it.
- That grammar closes a family with `;` or `.` **usually** — some records use a
  `,` or a `:`. `parse_poll()` rewrites those to `;`, but only after a
  parenthesis that actually names a family, so `(non-native),` and
  `(Coelopid flies),` do not cut the segment. Do not simplify that test: before
  the fix, 95 taxa carried a neighbouring segment's family (skippers filed as
  swallowtails, *Ectemnius* in Formicidae, hummingbirds with no family at all).
- Ackerman's marked-up review and the full record of what was done with each
  verdict are in `Resolution_documents/`. Read
  `Ackerman_review_resolution_2026-08-19.md` before revisiting any pollinator
  name. One row is still open: *Tetralona nipponensis* on *Bletilla striata*.
  The stale draft that predates the copy he was sent,
  `Ackerman_pollinator_name_review_2026-08-16.xlsx`, was moved to `_archive/`
  on 2026-09-15; use the `JDArev` file. The first email to him now sits in
  `Resolution_documents/` beside the follow-up.
- The sheet has three columns literally named `notes`; `read_excel` renames the
  duplicates automatically.

## Student coordinate returns

Each student edits their own copy of `private/coordinate_assignment_tracker.xlsx` and
sends the whole workbook back, so every copy carries all three tabs and only one
of them is current. First merge: 2026-09-10 (Naan 41 coordinates, Natalia 18;
Caleb had not started). Pre-merge file in `_archive/`.
Second merge: 2026-09-15 (one shared copy carrying both tabs; Naan +12
rows / 7 species in *Acianthera*, Natalia +7 rows / 5 species in *Coryanthes*
and *Elleanthus*; *Vanilla humblotii* re-done on land; Caleb still 0). Review
and repairs log in `private/coordinate_review_2026-09-15.xlsx`; pre-merge file in
`_archive/`. Written to the master the same day (17 records, 12 species);
*Acianthera luteola*, the Serra da Calçada point of *A. limae* and the
cultivated-plant point of *A. sonderana* were held back.

Merging a batch:

1. Take the copy whose own tab is filled and carry columns J:P (lat, lon,
   source, precision, date, status, notes) into one workbook. Assert
   species-by-species while doing it: columns A:I were byte-identical to the
   original in every copy returned so far, and that check is what catches a
   student who inserted or reordered rows.
2. Drop the example row (*Cattleya coccinea*, source "Smith et al. 2019, Fig.2")
   from every tab. Left in place it counts as `Done`.
3. Rebuild the `All assignments` tab from the three student tabs, then update
   the row bound in every `Progress` formula (`$863` as of 2026-09-15). The row count changes
   whenever a student adds rows.
4. Reset the Status data validation range, since deleting the example row
   shifts it.

Recurring entry problems, all seen in the first batch:

- **Dates.** Students type dd/mm/yyyy. Excel converts what it can to US dates
  (01/09/2026 becomes 9 January) and leaves the rest as text, so the column ends
  up holding three formats. 117 cells needed repair. The swap-back is only safe
  while every affected value has day <= 12; ask for yyyy-mm-dd instead.
- **`Coordinate source`.** 28 of Naan's 41 rows said only "R", "r" or "google
  earth" rather than the citation the README asks for. Do not guess what "R"
  refers to. Natalia's "En el paper <cite>" is usable.
- **`Precision`.** Used for place names instead of site/locality/region. Move
  the place name into `Notes` and leave `Precision` blank rather than inventing
  a level.
- **Extra rows.** A species with several study sites comes back as one row per
  site (*Vanilla hartii* became four Osa Peninsula rows). Keep them in the
  tracker; the master takes one row per record, so which coordinate goes forward
  is Raymond's call.
- **Access, not absence.** 25 of the 48 `Cannot find` rows say the student could
  not obtain the article. Those are recoverable and should be chased with PDFs
  before any of them is treated as a dead end.

QC on a returned batch: `global_land_mask` for the land test (1 km, so island
records do not false-flag the way `maps::world` did) plus Natural Earth 50m
polygons for country attribution, compared against the `Locality hint` string.
That combination found the *Vanilla planifolia* point in Sichuan, the
*V. humblotii* point 14 km out to sea, and five hint-versus-coordinate
mismatches. R is not installed in the desktop VM, so this ran outside the
`pollinators_wrangling.qmd` checks rather than reusing them.

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
