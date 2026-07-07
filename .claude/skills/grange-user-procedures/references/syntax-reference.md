# Grange User Procedure Syntax Reference

Distilled from "Introduction to Writing User Procedures" (Brian Pietsch) and the "Grange User Procedures Manual" v6.0 (Aug 2023). Grange runs on the Rocket UniVerse multi-value database; user proc sentences are RetrieVe queries with Grange-specific conventions.

## Hard rules

- **All uppercase**, except specific mixed-case data values, column headings, comments, and parameter prompt text.
- **Max 108 characters per line** (including any trailing underscore).
- The sentence executes as **one long statement**. End a line with ` _` (space then underscore) to continue on the next line. The final line of the sentence must NOT end with an underscore — that terminates the statement.
- **Proc ID**: max 15 characters, alpha/numeric, dashes/dots/commas OK, avoid special characters. Convention: writer's initials or team prefix, a dot, then keywords (e.g. HD.TONNES, GC.GRADES, WO.852-BY).
- **Description**: max 40 characters, ends with the type descriptor: **SO** (Screen Only), **FW** (Fixed Width, run to Spool Display), or **TD** (Tab Delimited, run to Pub/Lists only).
- **Comments**: a line starting with `* ` (asterisk THEN A SPACE — omitting the space causes an "invalid verb" error). Comments may appear before or after a sentence, never inside one.
- Only **one file** per sentence (the file after the verb). Reach into other files with `EVAL "TRANS(...)"`.
- KESSAR principle: Keep Every Step Simple And Relevant.

## Sentence anatomy (in order)

```
[Verb] [FILE] [record-IDs] [Selection] [Sort] [Breaks] [Output specs] [HEADING/FOOTING] [Report qualifiers]
```

### Verb
`LIST` is standard. Also available: SORT, COUNT, SUM, SELECT, SPOOL.

### File and record IDs
The Grange file (dictionary) to query — see `dictionaries/INDEX.md`. Explicit record IDs can follow the filename directly and display in the order given (record ID is always attribute 0). `FIRST 10` limits records — useful while testing. `LIST VOC LIKE ...*part*...` helps find a file name.

### Selection clauses
```
WITH <FIELD> <op> <value(s)>
```
- `WITH` — single-value field; or multivalue where matching one value shows the whole record
- `WHEN` — multivalue field, show only the matching multivalues within the record
- `WITHOUT` — exclude records matching the condition
- `WITH <FIELD>` alone (no operator) — field is non-null (e.g. `WITH ADDN.CODE`)

Operators: `=` EQ EQUAL | `#` NE `<>` `><` NOT | `>` GT AFTER | `<` LT BEFORE | `>=` GE `=>` | `<=` LE `=<` | LIKE MATCHES MATCHING | UNLIKE NOT.MATCHING | SAID SPOKEN `~` | IS.NULL | IS.NOT.NULL

Values: one or more quoted literals separated by spaces (`WITH SOURCE = 'BRS' 'EVG'` means OR); patterns `...*part*...` for LIKE/UNLIKE; another field name; or an EVAL (e.g. `WITH VINTAGE >= EVAL "MVYEAR+1"`). **Always enclose literal selection values in single quotes**, numeric or not (e.g. `WHEN ALL.LITRES > '100000'`); only field names and keywords like MVYEAR go unquoted.

Multiple clauses stack (implicit AND); can be grouped with brackets and joined with AND/OR.

`MVYEAR` = the current vintage year, self-updating. `WITH VINTAGE = MVYEAR` always returns the current vintage; `WITH VINTAGE = '2015'` is frozen.

### Sort clauses
- `BY <FIELD>` ascending | `BY.DSND <FIELD>` descending (single-value)
- `BY.EXP <FIELD>` / `BY.EXP.DSND <FIELD>` — multivalue; **also carries single-value data down** so every exploded row is filled (essential for Excel export when mixing single and multivalued output)

### Breaks, totals and summaries
- `BREAK.ON <FIELD>` — subtotal row whenever the value changes (field appears in output; don't repeat it in the output specs)
- `BREAK.SUP <FIELD>` — break without showing the field (useful with EVAL-combined break keys)
- Break fields should match the sort fields, or subtotals will be wrong.
- `TOTAL <FIELD>` — sum a numeric column at each break and at report end
- Column maths keywords before a field: `AVG` (AVERAGE), `ENUM` (ENUMERATE = count), `MAX`, `MIN`, `PCT` (% of total)
- `DET.SUP` — suppress detail rows, show only subtotal/total rows
- **Summary recipe**: `BY X` + `BREAK.ON X` + (`TOTAL Y` or `ENUM X COL.HDG "Count"`) + `DET.SUP`

### Output specifications
One field per line (ending ` _`), in display order. D-types and I-types both allowed. If none given, the file's `@` phrase default set is used.

### Field qualifiers (append after the field)
- `COL.HDG "Heading"` (aka DISPLAY.NAME) — mixed case allowed. Strongly recommended on every EVAL (default heading is the whole algorithm).
- `FMT "10L"` (aka FORMAT) — width + justification: L left, R right, T text-wrap; may include decimals and `,` for thousands. Strongly recommended on every EVAL. Dictionary "Output Format" column shows each field's default.
- `CONV 'code'` (aka CONVERSION) — display conversion. Dates start with D: `D4/` → 07/02/2016, `DMA[3]` → JUL, `DDMY2[Z,A3]` → 2 JUL 16, `DWAL` → Sunday, `DQ` → quarter. Times start with MT: `MTH` → 05:00pm, `MTS` → 17:00:00. Characters MC: `MCU` uppercase, `MC/A` strip non-alpha, `MC/N` strip non-numeric. Hard to override a conversion already set in the dictionary.

### EVAL expressions
`EVAL "expression"` creates a temporary calculated field — usable as output, selection, sort, or break.
- Arithmetic `+ - * /` with parentheses; comparison `= > < >= <= <>`
- **Rolling date windows**: `DATE()` returns today in UniVerse internal date format, so `WITH MRDATE >= EVAL "DATE()-14"` gives a self-updating last-14-days filter (date fields compare on internal values regardless of their display CONV).
- Concatenate with `:` ; substring with `[start,len]` appended; `LEN(...)`; `IF ... THEN ... ELSE ...`; constants (`EVAL "'-'"` outputs a literal dash placeholder); can call existing subroutines; `CONV(field,'code')` inside an EVAL converts format
- Can reference D-type fields of the current file, but NOT its I-types — copy the I-type's algorithm from the dictionary (Field Definition column) into the EVAL instead.
- **Cross-file lookup**: `EVAL "TRANS(FILENAME,RECORD.ID.FIELD,TARGET.FIELD,'X')"` — like a VLOOKUP into another file. Control code `X` = leave blank if not found (standard).
- If an EVAL will be reused widely, ask IT to promote it to a dictionary I-type.

### Report qualifiers (end of sentence)
`ID.SUP` (hide record ID, or place it: `ID.SUP <FIELD>` pattern in examples puts a chosen field first) | `HDR.SUP` (no page heading) | `COL.SUP` (no column headings) | `COUNT.SUP` (no record count) | `DET.SUP` | `LPTR` (send to printer/spool — without it, output displays on screen; add LPTR only when the proc is finished)

### Headings and footers
`HEADING "text"` / `FOOTING "text"`. **Use HEADING or HDR.SUP, never both** — HEADING overrides HDR.SUP, so including both is redundant (some legacy procs do this; don't copy it). Company standard heading:
```
*PZZZ019
DATA DEFAULT,,,<rows per page>,<chars per row>,,,,,,<PROC ID>
*PZZZ072
DATA <PROC ID>,<Title to appear>,<chars per row>
```
then inside the sentence: `HEADING "<<PZZZ072.HEADING>>"`.
*PZZZ019 overrides spool defaults (height 66, width 132) and sets the spool job description; `999999` for page length/width is best when exporting (prevents column stacking and repeating headers). **Recommended on every proc.**

### Tab delimiters (TD procs)
Start an output spec line with `[ ` (open square bracket then space) to insert a tab column break. TD procs: must run to Pub/Lists, cannot use parameters, description must end "TD".

### Parameters (inline prompts) — SO and FW procs only
- `<<A,Question text?>>` — ask the question, use the answer where it appears
- `<<Question text?>>` — reuse the answer from the identically-worded question asked earlier (same run, or earlier in the user's session)
- Convention: group all prompts as `* `-prefixed comment lines at the top so the questions are asked up front and visible, then reference `<<Question text?>>` in the sentence.
- Blank answers typically mean "no restriction" when used with LIKE patterns.

### Looping
`LOOP` above the prompts and body; at the end:
```
<<A,Go again?>> IF = "Y" THEN REPEAT
<<Go again?>> IF = "y" THEN REPEAT
```

### Username security
```
IF @LOGNAME = 'username' THEN GO OKAY
DISPLAY            (blank DISPLAYs = spacing)
DISPLAY Sorry, you are not authorised to run this report.
GO AWAY
OKAY:
```
One IF line per authorised user. `GO AWAY` terminates; `OKAY:` is the label where authorised users resume.

## Useful stock selections
| File | Code | Result |
|---|---|---|
| MGROWER | `WITH ACTIVITY = 'C'` | active accounts only |
| MBLOCKS | `WITH ACTIVITY = ''` | active blocks only (two quotes, NO space) |
| GRAPERECPT | `WITH ACTIVITY = ' '` | active weighnotes only (two quotes WITH a space) |
| VINTINTAKE | `WITH VINTAGE = MVYEAR` | current vintage flags only |
| MGROWER/MBLOCKS | `WITH GENERIC NE 'Y'` | exclude generic records |
| VINTINTAKE | `WITH GROWER.GENERIC NE 'Y'` | exclude generic records |
| any of MGROWER/MBLOCKS/VINTINTAKE/GRAPERECPT | `WITH OWN.BLOCK = 'Y'` / `NE 'Y'` | company vineyards only / external growers only |
| MGROWER/VINTINTAKE/GRAPERECPT | `WITH RTYPE = 'G'` / `'C'` | growers only / carriers only |
| CHIT | `WITH MACTIV = 'R'` | requested (incomplete) chits only |

## Running & operational context (for interpreting procs and advising users)
- Procs are maintained in USER.PROCS (Create - Maintain User Procedures); run via USER.P (to spool display or pub/lists) or PZZZ391 (pub/lists only; no F6 search; no prompts).
- Spool display requires DEFAULT printer = DUMMY101, disposition HQ.
- `$R` typed in the editor runs the proc-so-far for checking; `Q` quits the preview. `$i` inserts rows.
- TD procs → Pub/Lists → retrieved via FTP → opened in Excel as tab-delimited. FW procs → spool display → exported → Excel fixed-width import.
- Around 600 saved procs exist; browse via USER.P.LIST with an ID prefix.
