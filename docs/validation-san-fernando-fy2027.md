# Validation: City of San Fernando, Pampanga, FY 2027 (IAN-7)

An independent check of `portal.config.ts` against the live BetterGov.ph Budget API
(`source_commit a0caa778`, built 2026-10-26), run 2026-11-03. Every figure below is read from
the API's list and per-project responses; nothing is computed beyond sums of those figures.

**Recommendation: GO for the prototype scope**, with the caveats at the end.

## Scope

| | |
| --- | --- |
| Query | `q=San Fernando Pampanga`, region `III` |
| District card | `leg:pampanga:3rd` |
| Raw DPWH matches | 8 (the other three programs: 0) |
| Kept / excluded | 6 / 2 |

## Accepted records (6)

All six are `region III`, zone `local`, `held: false`, and name "City of San Fernando, Pampanga"
in the title. Pages are the PDF page the API links; `H2`/`H3` are House bill second/third reading
(same page and amount in both for all six).

| ID `dpwh:house-reading:` | Title (shortened) | DPWH office | NEP | House | Group | NEP page | House page |
| --- | --- | --- | ---: | ---: | --- | ---: | ---: |
| `12396` | Multi-Purpose Building, City of San Fernando (15.086378, 120.657827) | Central Office | — | ₱150,000,000 | house only | — | 638 |
| `12422` | Multi-Purpose Building, Barangay Lourdes | Central Office | ₱57,000,000 | ₱57,000,000 | unchanged | 565 | 637 |
| `7757` | Del Carmen-Bulaon-Calulut Road, Sta. 0+000 – 2+054 | Central Office | — | ₱59,246,000 | house only | — | 421 |
| `8004` | Roads incl. Drainage, 1st and 4th Street, Alasas Village, Barangay Alasas | Pampanga 1st DEO | — | ₱25,000,000 | house only | — | 525 |
| `7959` | Concreting of Road incl. Lined Canal, Barangay San Felipe | Pampanga 1st DEO | — | ₱15,000,000 | house only | — | 526 |
| `16225` | Multi-Purpose Building (Motorpool), DPWH Pampanga 1st DEO, Sindalan (15.080596, 120.634261) | Pampanga 1st DEO | ₱10,000,000 | ₱10,000,000 | unchanged | 684 | 935 |

Documents: NEP is `nep2027-vol-iib`; House bill is `hb10858-2r-vol-ic` / `hb10858-3r-vol-ic`. Each
record's `pdf_url` is `https://budget-assets.bettergov.ph/hgab/2027/pdf/<document>.pdf#page=<n>`.

## Excluded records (2)

| ID | Why | Evidence |
| --- | --- | --- |
| `12536` | Held pending location verification | Title: "City Town Center, Barangay Calulut, City of San Fernando", ₱10M House only, Pampanga 1st DEO, p. 740. Its coordinates (15.154250, 120.771463) are about 14 km from `12396` and 17 km from `16225` (the two coordinate-bearing records that sit in San Fernando), and about 10–11 km from the nearest Mexico- and Magalang-titled lines. The title and the coordinates disagree, and the API does not say which is right. |
| `4112` | Outside the City of San Fernando | Title: "San Fernando-Sto. Tomas-Minalin Tail Dike (Minalin Section) … Barangay Sta. Catalina, Minalin, Pampanga", ₱120M House only, Central Office, p. 304. "San Fernando" is part of the dike's corridor name; the section is in Minalin. |

`12536` is held, not shown to be outside the city. It is a data conflict that needs a source check
(for example the printed coordinates against the project list of the city government).

## Classification of every returned record

| Class | Records |
| --- | --- |
| Matches existing verified record | `12396`, `12422`, `16225`, `7757`, `7959`, `8004` |
| New plausible San Fernando record | none |
| False positive / namesake | none |
| Outside San Fernando | `4112` |
| Needs review | `12536` |

"Matches existing verified record" means the ID is on BetterSanFernando's list of six. This check
compared IDs, titles and the API's own figures; it did not have the manual verification sheet, so
the amounts were not compared against it.

## Reconciliation

| | Lines | NEP | House bill (3rd reading) |
| --- | ---: | ---: | ---: |
| API total for the 8 raw matches | 8 | ₱67,000,000 (2 lines) | ₱446,246,000 (8 lines) |
| Less `4112` | | — | ₱120,000,000 |
| Less `12536` | | — | ₱10,000,000 |
| **Portal (6 lines)** | **6** | **₱67,000,000 on 2** | **₱316,246,000 on 6** |

`npm run check` reports the same. The versions are two readings of one budget and are never
added to each other. By status: 4 House bill only (₱249,246,000), 2 same amount in both
(NEP ₱67,000,000; House ₱67,000,000). By office: Central Office 3 lines (₱266,246,000 House),
Pampanga 1st DEO 3 lines (₱50,000,000 House).

## Why three records sit under the Pampanga 1st DEO

`8004`, `7959` and `16225` are printed under "Pampanga 1st District Engineering Office" in both the
NEP (where one exists) and the House bill. They are in the portal because their titles name the
city, not because of the office, and that is correct:

- The API associates that DEO with two districts, `leg:pampanga:3rd` and `leg:pampanga:4th` (Tier B,
  "shared office total, not split"). It is not a San Fernando-only office.
- The DEO has 156 House bill lines (₱1.58B). Only 4 name San Fernando (`8004`, `7959`, `16225`, and
  the held `12536`; ₱60M). The rest name Macabebe, Candaba, Masantol, Apalit, Minalin, San Luis,
  San Simon, Bacolor, Mexico, Sto. Tomas, Sta. Ana, Arayat and others.
- Adding the office to `scope.offices` would put about 150 lines and roughly ₱1.5B of other
  municipalities' projects into a city portal. Keeping the three by title is the narrower and
  more accurate choice, and `npm run check` already lists them as a "title names the place, office
  not included" warning for review. That warning is expected here.
- `16225` is also literally at the DEO's own compound ("at DPWH Pampanga 1st DEO, Sindalan").

## Semantics checked

- NEP and House bill are never added. `summarise()` in `src/lib/scope.ts` keeps them in separate
  totals and the pages say "compared, never added".
- A missing NEP is `nep_php: null`, shown as "—", never ₱0 (`src/lib/format.ts`). The API's own
  wording for the four House-only lines is "no matching line found in the NEP. A candidate, not a
  confirmed addition."
- House-only is an evidence state, not proof of an enacted change. The API notice also says
  absence from its snapshot does not establish absence from the published NEP. The 2027 figures
  are proposals in the NEP and House bill, not spending.
- Before this documentation-only validation commit, only `portal.config.ts` differed from `main`; this PR adds only this validation document and makes no source, dependency, configuration, or UI changes.

## Caveats

- **Source PDFs were not opened.** The sandbox could not reach `budget-assets.bettergov.ph`. Page
  numbers and document names are as exposed by the API; they were not read against the PDF text.
  Open the six `pdf_url` links once by hand before relying on them in public.
- `16225` carries the API flag `nep_evidence_review` ("NEP amount still being checked against the
  scan"). Its NEP ₱10M (page 684, line `p684:r7`) matches the House figure, but the API has not
  finished checking it.
- `12536` is also flagged `group_exhausted`: it is a leftover House line in the same "Multi-Purpose
  Building, City of San Fernando" group as `12422`, with no NEP line left to pair. That concerns
  matching, not location. Revisit both together if it is released.
- No independent geocoder was reachable, so the location conflict on `12536` rests on distances
  between coordinates in the API, not on a map lookup.
- **Recall is limited by title text** (the gap noted in [api.md](api.md#gaps)). A line that names
  only a barangay, or spells the city "Sn Fernando", would not be found. In Pampanga the only such
  candidate is the national road "Sn Fernando-Lubao Rd" (`662`–`665`, Pampanga 2nd DEO, K0077–K0083,
  ₱115.5M House). It is a corridor name, not evidence of a location in the city, so it is not
  included. Other "San Fernando" titles nationwide (La Union, Cebu, Bukidnon, Camarines Sur, Masbate,
  Albay, Iloilo, Nueva Vizcaya, and in Region III Zambales and Tarlac) are correctly left out by
  the `Pampanga` word in the query.
- The enacted-budgets section (2020–2026) finds one program by name: a Multi-Purpose Building,
  Brgy. San Isidro, "San Fernando Pampanga", ₱5.0M in 2020 and ₱0 afterwards. It is outside the
  FY2027 six and was not part of this check beyond the portal's own reconciliation against the API.
- The API describes itself as independent and unofficial. Nothing here implies irregularity.

## Re-running

```bash
npm run check -- --verbose
```

Expected: 6 lines; NEP ₱67.0M on 2; House bill ₱316.2M on 6; 4 house-only, 2 unchanged; `OK`.
