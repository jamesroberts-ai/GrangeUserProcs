---
name: grange-user-procedures
description: Interpret and write "User Procedures" (user procs) for the Grange winemaking system at Treasury Wine Estates — RetrieVe/LIST query reports on the UniVerse multi-value database. Use this skill whenever the user mentions a user procedure, user proc, Grange report/extract, LIST statement, or a Grange file such as CHIT, OPSMOVE, MTANK, MGROWER, VINTINTAKE or GRAPERECPT; whenever they paste code containing LIST/WITH/COL.HDG/FMT/EVAL/PZZZ patterns and ask what it does; and whenever they ask to write, draft, modify, review, debug or explain a data extraction from Grange — even if they only describe the report they want (e.g. "a report of incomplete chits by tank") without saying "user proc".
---

# Grange User Procedures

User procedures ("user procs") are saved RetrieVe LIST queries in Grange, TWE's UniVerse-based winemaking system. This skill covers two tasks: **interpreting** an existing proc in plain English, and **drafting** new or modified procs in the house style. Output for drafted procs is always plain-text code.

## Bundled references — read before answering

1. `references/syntax-reference.md` — the full syntax: sentence anatomy, selection/sort/break clauses, EVAL and TRANS, qualifiers, PZZZ019/072 heading blocks, parameters, LOOP, security, stock selections. **Read this first for any interpret or write task.**
2. `dictionaries/INDEX.md` — which Grange file holds what, with keys. Then grep the specific `dictionaries/<FILE>.csv` to verify field names.
3. `references/system-overview.md` — how Grange hangs together (tanks/chits/composition), join keys for TRANS lookups, date-field meanings, multivalue concepts, glossary. Read when choosing between files, tracing relationships, or interpreting domain terms.
4. `references/examples.md` — two annotated real procs and the house-style checklist. Read before drafting anything.

## Non-negotiable discipline: verify every field name

Grange field names are inconsistent across files (the same data has different names in different files). **Never guess a field name.** For every field used in a draft, and every unfamiliar field in an interpretation, grep the file's dictionary CSV:

```
grep -i "keyword" dictionaries/OPSMOVE.csv        # search one file
grep -ril "baume" dictionaries/                   # find which files hold something
```

The CSV columns give you everything needed: Type (D/I/PH), Depth (S/M — drives WITH vs WHEN, BY vs BY.EXP), Assoc (which multivalues align), default Column Heading, Output Format, Conversion, and DESC. If a needed field genuinely can't be found, say so and show the nearest candidates rather than inventing one. Dictionary snapshots carry a download date in row 1 — mention it if staleness could matter.

## Task A — Interpreting a proc ("what does this do?")

1. Read the syntax reference. Identify the structure: header comments (owner, purpose, schedule), precursors (RUN/TERM, PZZZ blocks, prompts), the sentence, footer comments (history).
2. Grep the dictionary CSV for each field to get its real meaning (DESC column), not a guess from the name.
3. Explain in plain English: what file it queries, which records it selects, what each output column is, how it's formatted/sorted/summarised, where the output goes (SO/FW/TD and PZZZ settings), and anything operational (account switching via PZZZ214, schedules, security). Distinguish what the proc's comments actually state from what you're inferring — say "the comments state X" vs "this suggests Y".
4. Note quirks worth flagging: commented-out code and what re-enabling it would do, placeholder `EVAL "'-'"` columns, mismatched sort/break fields, missing PZZZ019, lines over 108 chars, or a trailing underscore on the last line (which would break the sentence).

## Task B — Writing or modifying a proc

1. **Clarify the requirement** if ambiguous — but make reasonable Grange-informed assumptions and state them rather than interrogating the user.
2. **Always ask who the Owner should be** for the header comment if not stated. Never assume or invent an owner. (This is the one question that must always be asked.)
3. **Choose the source file** using `dictionaries/INDEX.md` and the system overview — one file per sentence; use `EVAL "TRANS(...)"` for lookups into others (join keys in system-overview.md).
4. **Verify every field** against the dictionary CSV. Check Depth: multivalued fields need WHEN (to filter within record) or BY.EXP (to explode/carry down) decisions — surface this to the user when it affects their output.
5. **Draft in the house style** (full checklist in examples.md):
   - `* Owner = <name>` / `* <purpose>` header comments
   - Precursors: grouped prompts, RUN/TERM if needed, `*PZZZ019` + `*PZZZ072` heading block by default for FW/TD extracts, with `HEADING "<<PZZZ072.HEADING>>"` in the sentence
   - The sentence: uppercase, ≤108 chars/line, ` _` continuations, one output field per line with aligned COL.HDG and FMT (always on EVALs), bare `_` lines to group blocks, report qualifiers last, **no underscore on the final line**
   - `* Written <date>, for <requester>` footer; dated change comments when modifying
6. **Propose a proc ID** (≤15 chars, prefix convention e.g. initials + dot + keyword) and a description (≤40 chars ending SO/FW/TD) — and recommend the right type: SO for small on-screen checks, FW for spool-display Excel exports, TD (with `[ ` delimiters) for clean Pub/Lists exports. Remind that TD procs can't use parameters.
7. **Output the draft as plain text** in a code block. For long procs or when the user wants a file, also save as `.txt`. Suggest testing with `FIRST 10` and `$R` before saving, and adding LPTR only once finished.

When modifying an existing proc, preserve the owner's original style and comments; append dated change comments rather than silently rewriting history — and remind the user to get the owner's permission before changing someone else's proc.

## Domain cautions

- **Always single-quote literal selection values**, including numbers: `WHEN ALL.LITRES > '100000'`.
- **HEADING overrides HDR.SUP** — use one or the other, never both.
- **Generic code is called "MPF" (Master Planning Family)** in TWE human terms — label GENERIC / ALL.GENERIC columns "MPF", not "Blend".
- MVYEAR self-updates to the current vintage; hard-coded years freeze. Choose deliberately.
- MBLOCKS active = `''` (no space); GRAPERECPT active = `' '` (with space). Easy to get backwards.
- On MTANK/WINE.IN.TANK prefer ALL.GENERIC / ALL.VINT / ALL.LITRES over the plain fields (intransit wine coverage).
- OPSMOVE is per-bulkwine-account; cross-account needs RECOVERIES (~3 months), BW.STD.CST.OPS, or per-account looping via `RUN UT.BP PZZZ214 <acct>` (see examples.md).
- I-types cannot be referenced inside an EVAL — copy their Field Definition from the dictionary into the EVAL.
- **CHIT late-added allocation attributes (MFGENERIC 188, MFALLOCVINT 189, MFALLOCLITRES 190) are blank on incomplete (MACTIV='R') chits** — confirmed in production Jul 2026 for MFGENERIC and MFALLOCVINT. A tank's MPF isn't really chit data anyway: even MTANK's GENERIC I-type derives it from BWALLOC. For incomplete chits, look up the from-tank's current allocation instead: `EVAL "TRANS(BWALLOC,MFTANK,GENERIC,'X')"` for MPF and `EVAL "TRANS(BWALLOC,MFTANK,VINT,'X')"` for vintage — GENERIC (attr 2) and VINT (attr 12) share BWALLOC's ALLOC association, so the pair align value-for-value. Caveat: this returns the tank's *current* allocation (multivalued if the tank holds several MPFs), not necessarily what the chit will despatch.
- A field's attribute number hints at its vintage: high numbers far from their association's core block are later additions and may only populate for certain chit types or at completion — verify population with `LIST <FILE> WITH <FIELD> FIRST 10` before relying on them.
- CONV set in the dictionary is hard to override in the proc; FMT is overridable.

## Maintaining the dictionary snapshots

When the user supplies freshly downloaded Grange dictionary workbooks (xlsm/xlsx with 'File name = ' header sheets), regenerate the CSVs with the bundled script, then repack the skill:

```
python3 scripts/update_dictionaries.py <workbook1> [workbook2 ...] --out dictionaries/
```

Latest download date wins where files overlap. Afterwards: check for near-duplicate file names from workbook typos and merge them, and update `dictionaries/INDEX.md` if new Grange files have appeared.
