# Annotated Example User Procedures

Real procs written by Brian Pietsch (Data Development Manager, TWE Barossa). These show the house style the skill should reproduce.

## Example 1 — WO.852-BY (scheduled extract, standard heading)

```
* Owner = Brian Pietsch
* Extracts addition reconciliation data for Winery Operations team
* Runs at 12.05 pm and 8.05 pm daily


*PZZZ019
DATA DEFAULT,,,999999,300,,,,,,WO.852-BY
*PZZZ072
DATA WO.852-BY,Incomplete chits at TWEB with Additives,300

LIST CHIT _
WITH MACTIV = 'R' _
WITH ADDN.CODE _
_
ID.SUP MCNOTE                                 COL.HDG "Additive Chit " FMT "14L" _
MOPCD                                         COL.HDG "Operation Code" _
MAST.OPCD                                     COL.HDG "Operation Master" _
CREATE.DATE                                   COL.HDG "Date Created" _
CREATE.TIME                                   COL.HDG "Time Created" _
_
MRDATE                                        COL.HDG "Requested Date" FMT "10R" _
ADDN.TANK                                     COL.HDG "Tank Added To          "  _
_
ADDN.CODE                                     COL.HDG "Add Code" FMT "4L" _
MADESCP                                       COL.HDG "Add Description" FMT "72L" _
ADDN.RATE                                     COL.HDG "Rate Added" FMT "9R" _
ADDN.UNIT                                     COL.HDG "Rate UOM" FMT "6T" _
ADDN.QTY                                      COL.HDG "Quantity " FMT "9R" _
ADDN.AUOMISS                                  COL.HDG "Issuing UOM " FMT "12T" _
_
HEADING "<<PZZZ072.HEADING>>"_
HDR.SUP COUNT.SUP

* Copied from WM.838-03A on 9 Jun 2026
* File name is WO852BY
* WITH EVAL "TRANS(MOPS,MOPCD,'OTYPE','X')"='A' _
* RUN UT.BP PZZZ214 BWBY
* TERM 300,33
```

What to notice:
- **Header comments**: owner, purpose, and (because it's scheduled) the run times.
- ***PZZZ019 / *PZZZ072** precede the sentence: spool defaults overridden (999999 rows per page, 300 chars wide), job description and report title both carry the proc ID. The width in PZZZ019, PZZZ072, and any TERM line should agree.
- `WITH MACTIV = 'R'` — requested (incomplete) chits; `WITH ADDN.CODE` — only chits that have an additive code (non-null test, no operator).
- `ID.SUP MCNOTE ...` — suppresses the default record ID, then leads with the chosen key field.
- **Bare `_` lines** create visual grouping between blocks of output fields without ending the sentence.
- Column headings are aligned with whitespace padding for readability in the 108-char editor.
- **Footer comments** record provenance (copied from WM.838-03A, date), the pub/lists file name, and retired/optional lines kept as commented code (a selection via TRANS into MOPS, an account-switch command, a TERM setting).
- One thing NOT to copy: `HDR.SUP` alongside `HEADING "<<PZZZ072.HEADING>>"` is redundant — HEADING overrides HDR.SUP. When drafting, include one or the other.

## Example 2 — WM.838-03A (multi-account extract with account switching)

```
* Owner = Brian Pietsch
* Extracts addition data for Elara project

RUN UT.BP PZZZ214 *CONTROL*<<F(CONTROL,MASTER.ACCOUNT,1,0,0)>>*1
RUN UT.BP PZZZ214 BWBY
TERM 300,33

LIST CHIT _
WITH MACTIV = 'R' _
WITH ADDN.CODE _
_
ID.SUP MCNOTE                                 COL.HDG "Additive Chit " FMT "14L" _
MOPCD                                         COL.HDG "Operation Code" _
MAST.OPCD                                     COL.HDG "Operation Master" _
EVAL "'-'"                                    COL.HDG "Date Started" FMT "10R" _
EVAL "'-'"                                    COL.HDG "Time Started" FMT "8R" _
CREATE.DATE                                   COL.HDG "Date Created" _
CREATE.TIME                                   COL.HDG "Time Created" _
_
ADDN.TANK                                     COL.HDG "Tank Added To          "  _
EVAL "'-'"                                    COL.HDG "Previous Chit " FMT "14L" _
EVAL "'-'"                                    COL.HDG "Next Chit     " FMT "14L" _
MTBATCH                                       COL.HDG "Batch  " FMT "7L" _
_
ADDN.CODE                                     COL.HDG "Add Code" FMT "4L" _
MADESCP                                       COL.HDG "Add Description" FMT "72L" _
EVAL "'-'"                                    COL.HDG "Lot Num " FMT "8R" _
ADDN.RATE                                     COL.HDG "Rate Added" FMT "9R" _
ADDN.UNIT                                     COL.HDG "Rate UOM" FMT "6T" _
ADDN.QTY                                      COL.HDG "Quantity " FMT "9R" _
ADDN.AUOMISS                                  COL.HDG "Issuing UOM " FMT "12T" _
MRDATE                                        COL.HDG "Requested Date" FMT "10R" _
_
HDR.SUP COL.SUP COUNT.SUP

* Repeated for each Bulkwine Account
* Then finishes with

RUN UT.BP PZZZ214 *
TERM

* Written 16 Jan 2026, for Elara project
* Aligns with WM.838-03
* Removed 'WITH EVAL "TRANS(MOPS,MOPCD,'OTYPE','X')"='A' _' on 24 Jun 2026
```

What to notice:
- **RUN UT.BP PZZZ214 <account>** switches the session into a bulkwine account before the sentence; the sentence block is repeated per account, and `RUN UT.BP PZZZ214 *` at the end restores the original account. `TERM 300,33` sets terminal width/depth for the run; a bare `TERM` at the end resets it.
- `EVAL "'-'"` outputs a literal dash — placeholder columns so the layout of this extract aligns column-for-column with a sibling proc (WM.838-03) for a downstream consumer (the Elara project). Placeholders still get COL.HDG and FMT.
- `HDR.SUP COL.SUP COUNT.SUP` — headings and counts suppressed on the repeated blocks so the per-account outputs concatenate cleanly.
- **Change history** in dated footer comments, including the exact line removed and when.

## House style checklist (apply when drafting)
1. `* Owner = <name>` — always ask the requester who the owner should be; never assume.
2. `* <purpose>` on line 2; add schedule/audience comments if relevant.
3. Precursors next: prompts grouped as comments, RUN/TERM commands, *PZZZ019 + *PZZZ072 (heading block recommended by default for FW/TD extracts).
4. The sentence: verb + file, selections (literal values always in single quotes), sort, breaks, then output specs one per line with aligned COL.HDG/FMT, bare `_` lines to group blocks, report qualifiers last (no trailing underscore on the final line). If using `HEADING "<<PZZZ072.HEADING>>"`, do not also add HDR.SUP.
5. Footer comments: `* Written <date>, for <requester/project>` plus any change history.
6. Keep every line ≤108 characters; everything uppercase except headings, prompt text, mixed-case data values, and comments.
