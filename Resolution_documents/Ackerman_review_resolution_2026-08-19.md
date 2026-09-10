# Pollinator name review — Ackerman's verdicts and what was done with them

**Reviewed by:** James D. Ackerman
**Returned:** `Resolution_documents/Ackerman_pollinator_name_review_2026-08-16 JDArev.xlsx`
**Resolved into the pipeline:** 2026-08-19

Of the 22 questions put to him — nine spelling pairs, three family placements
and ten overrides — twenty-one are now closed. One is open.

The most useful thing in his reply was not a taxonomic ruling at all. Three of
the queries turned out to be the same bug in our parser, and he diagnosed it
from the verbatim text without seeing the code.

***

## The one bug behind three questions

Our `pollinators` grammar is `ORDER: Genus species, G. species (Family); …`,
and `parse_poll()` ended a family segment at a `)` followed by `;` or `.`. Some
records close the family with a **comma** or a **colon** instead. When they do,
the segment ran on and every taxon in it inherited the *next* family named.

> *Epipactis palustris*: `… Rhopalum clavipes (Crabronidae wasps), Formica
> cunicularia, … Myrmica ruginodis (Formicidae);`

Eleven crabronid wasps, *Ectemnius* among them, were filed under Formicidae.
His note reads: *"for palustris there was a comma after the family name instead
of a semicolon — perhaps that is why your search did not find the family."*
It was.

The same comma explains *Pontia deplidice* on *Anacamptis morio*, which our
family-consistency rule then wanted to "correct" to the noctuid *Polia*, and
*Lymire edwardsii* on *Epidendrum amphistomum*, which had picked up Geometridae
from three taxa further down the list.

`parse_poll()` now rewrites a `,` or `:` to `;` when it follows a parenthesis
that **names a family** — `[A-Z][a-z]+dae` inside the brackets. The family test
is what makes it safe: `(non-native),` and `(Coelopid flies),` are annotations
and must not cut the segment.

The effect is much wider than the three records he asked about. Across all
1,888 pollinator cells, **95 taxa change family and 7 gain one that was
previously missing. None lose a family.** Among the corrections:

| was | is now | n |
|---|---|---|
| Papilionidae | Hesperiidae | 13 |
| Halictidae | Apidae | 13 |
| Formicidae | Crabronidae | 11 |
| Papilionidae | Nymphalidae | 6 |
| Muscidae | Syrphidae | 5 |
| Geometridae | Erebidae | 3 |
| Heleomyzidae | Mycetophilidae | 3 |
| — | Trochilidae | 2 |

Fourteen skippers filed as swallowtails, thirteen *Lasioglossum* as apidae,
two hummingbirds with no family at all. All from one missing delimiter.

Two source typos surfaced only once the families stopped being overwritten, and
have been added to `family_recode`: `Syprphidae` → Syrphidae, `Cerembicidae` →
Cerambycidae.

***

## Sheet 1 — spelling pairs

| pair | his verdict | applied as |
|---|---|---|
| *Mycomya* / *Mycomyia* | spelling error → **Mycomya** | `genus_recode`; master corrected (3 records) |
| *Oedemera* / *Oedemeronia* | synonym → **Oedemera** | `genus_recode` only — see note below |
| *Scathophaga* / *Scatophaga* | **Scathophaga**; *"flies, not fishes for us"* | `genus_recode`; master corrected |
| *Scathophaga* / *Scatophagia* | **Scathophaga**; *"Scatophaga is a fish"* | `genus_recode`; master corrected |
| *Phaethornis* / *Phaetornis* | **Phaethornis**; *"the font makes r + n look like m"* | `genus_recode`; master corrected |
| *Augochlorine* / *Augochlorini* | no error; *"ID'd only to tribe level"* | `genus_recode` + `SUPRAGENERIC` |
| *Vespa* / *Vespula* | **both valid genera of Vespidae** | no change |
| *Eristalis* / *Eristalinus* | **both valid genera of Syrphidae** | no change |
| *Tetralonia* / *Tetralona* | *no verdict returned* | **open** |

He caught something we had got backwards. Our earlier GBIF pass had settled
`Scatophagia` → `Scatophaga` — which is a genus of **fish**. The correct name
for both records is *Scathophaga* (Scathophagidae). That entry has been
replaced rather than added to.

*Oedemeronia* is corrected in the recode map but **not** in the master, because
"synonym" is not "typo": the source printed a real name that has since been
subsumed. The same reasoning keeps *Vespa*, *Eristalinus*, *Pontia* and the
`Sp.` abbreviations exactly as published.

**Still open — *Tetralona nipponensis* on *Bletilla striata*.** He returned no
verdict, so nothing has been applied and the record stands as published. Our
own workbook did not help him here: the evidence column mistakenly compared
*Tetralonia* against *Tetragona* rather than the recorded *Tetralona*. That is
worth correcting before we ask again.

## Sheet 2 — family placements

| genus | recorded | his verdict | applied as |
|---|---|---|---|
| *Ectemnius* | Formicidae (3); Sphecidae (1) | **Crabronidae** | parser fix + one master correction |
| *Lymire* | Geometridae (1); Ctenuchidae (1) | **Erebidae**; *"old family circumscriptions"* | `family_current` |
| *Sp* | Chloropidae (1); Scoliidae (1) | *"the families are correct… Sp. = species"* | `NOT_A_GENUS` |

He split *Ectemnius* into two different faults. The *Epipactis palustris*
records were the comma bug above. The *Epipactis flava* record — `Ectemnius
(Sphecidae)` — he called *"an error"* outright, and that one cell has been
corrected in the master.

*Lymire* introduces a distinction the project did not have. Ctenuchidae is not
a misspelling; it is a family circumscription that has since been folded into
Erebidae. Putting it in `family_recode` alongside `Aipdae` → Apidae would
record the author as having made a mistake he did not make. It goes in a new
map, **`family_current`**, kept separate for exactly that reason.

*Sp.* was our bug, as he says — the tokeniser read the abbreviation for
"species" as a genus. `NOT_A_GENUS` now drops the genus claim while keeping the
family, so the two *Schizochilus* records still count at family rank.

## Sheet 3 — overrides

Nine of the ten spellings our family-consistency rule had discarded, he
confirmed as typos: *Bombis*, *Chalicodima*, *Proseca* (twice), *Agrostis*,
*Scaria*, *Thynoides*, *Zasipilothynnus*, *Megasella*. All are now in
`genus_recode` and corrected in the master.

He rejected one, and it was the one worth asking about:

> *Polia* / *Pontia* — **"No — the other name is right."**
> *"after the family name was a comma, not a semicolon"*

*Pontia deplidice* (Pieridae) on *Anacamptis morio* is correct as published.
Had we applied the rule, a pierid butterfly would have become a noctuid moth.
Nothing is recoded; the parser fix makes the family read correctly on its own.

The rule's weakness is now documented rather than theoretical: it assumes the
recorded family is right, and when the family is wrong *because of our own
parsing*, it launders the error into a name change.

## Sheet 4 — the 143 settled by GBIF

He annotated two rows, both agreeing: *Bombus* is Apidae, and *"Andrena is in
Andrenidae, not Apidae."* No action.

***

## What changed on disk

**`pollinators_wrangling.qmd`**

- `parse_poll()`: family-aware `,`/`:` segment delimiter; `NOT_A_GENUS`;
  `SUPRAGENERIC` (new `rank` value); `last_genus` no longer absorbs `Sp.`
- `genus_recode`: 12 entries added, 1 corrected (`Scatophagia`)
- `family_recode`: `Syprphidae`, `Cerembicidae`
- `family_current`: new map, `Ctenuchidae` → Erebidae
- new section "Specialist arbitration, 2026-08-18" tabulating every verdict

**`Pollination_List_MASTER.xlsx`** — 16 cells in the `pollinators` column,
each logged in `typo_corrections_changelog.csv`. Row counts, coordinates and
all other columns are byte-identical.

(A second, unrelated pass on the same day filled the `genus` and `subfamily`
columns down — 4,833 cells, logged separately in
`filldown_changelog_2026-08-19.csv`. It changed no pollinator data and no
counts. See `PROGRESS.md`.)

Parsing the edited master and the original master through the new parser gives
identical output (4,988 long rows; 2,132 binomials; 130 families) — the master
corrections and the recode maps agree, which is the point of keeping both.

**Re-run `pollinators_wrangling.qmd`, then re-render both maps.** The enriched
workbook does not yet reflect any of this.

***

## Worth a second question, when there is a reason to write again

- ***Tetralona nipponensis*** — the open row, with our comparison text fixed.
- **Other old circumscriptions.** *Lymire* set a precedent that applies to
  names we did not ask about: `Arctiidae` (1 record), `Danaidae` (4),
  `Satyridae` (3), `Eumenidae` (3), `Otitidae` (1), `Coerebidae` (1). Each is a
  family that has been subsumed or renamed. They are left alone for now —
  `family_current` has one entry, deliberately.
- **Duplicate records.** *Schizochilus bulbinella*, *S. flexuosus* and
  *Coelogyne prolifera* each appear twice with the same pollinator list
  differently punctuated. Probably a merge artefact rather than two
  observations, but that is a question for whoever compiled them.
