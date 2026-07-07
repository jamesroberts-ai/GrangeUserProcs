# Grange System Overview

Grange is Treasury Wine Estates' winemaking system, built on the Rocket UniVerse multi-value database. It tracks wine through its entire lifecycle: grape receipt → crush → fermentation → blending → maturation (tank/barrel) → bottling or bulk sale.

## Three core ideas
1. **TANKS** are physical vessels. Their contents change over time (volume, blend, batch, composition).
2. **CHITS** (Cellar Operation Instructions / Cellar Notes) record every action done to wine — additions, movements, transfers, tests. Each chit generates one or more OPSMOVE records when completed.
3. **COMPOSITION** is tracked back to source — every litre of wine in a tank can be traced to the weighnote / grape receipt / purchase it came from (via GRAPE.DEST and the SUMCOMP.* files).

Grange also manages master/reference data (blends, batches, additives, operation codes, locations), purchase orders, inter-site tanker movements, chemistry tests, and bottling orders (linked to SAP/JDE).

## Value stream (physical step → Grange artefacts)
1. **Grape growing & harvest** → GRDOCKET dockets, weighnote numbers (WNOTE)
2. **Crush & press** → crush CHIT (MCNOTE), GRDOCKET crushnote aggregates dockets, EXTRATES (L/tonne), CHEM readings, OPSMOVE on completion, GRAPE.DEST + SUMCOMP.TANK recalculated
3. **Fermentation** → addition chits (CHIT/ADD.NOTICE), OPSMOVE per addition, MADD & LOT consulted
4. **Blending / racking** → BLEND.PROP recipes, move chits, OPSMOVE (from/to tanks), BWALLOC.OPSMOVE, MBLEND/MBATCH consulted, TANK.HIST accumulates
5. **Maturation** → DP barrel locations, WOODMAT.TANK oak composition, BARREL.GROUP, periodic CHEM, DIPTABLE for volume from dip
6. **Finishing** → finishing chits, OPSMOVE, pre-bottling CHEM panel, MBLEND.CHEM, WINE.STATUS → released
7. **Bottling or bulk sale** → SAP.PR.ORDER work orders, bottling chit / TANKER.MOVE, CHEM.BOTT, SUMCOMP.BOT, GRAPE.DEST.CHITS composition of output, ITEM.MSTR & WINERY.LOCATION consulted
8. **Reporting & reconciliation** → OPSMOVE.SUMMARY (site×month), RECOVERIES (~3 months all sites), BW.STD.CST.OPS (cross-site), SUMCOMP.TANK.SV start-of-vintage snapshot

Forward flow: Weighnote # → MCNOTE + Tank → Blend/Batch on Tank → Bottling Work Order / Consignment. At every step the CHIT is the unit of change, OPSMOVE is the event record, and GRAPE.DEST + SUMCOMP.TANK are recalculated to preserve traceability. GRAPE.DEST anchors end-to-end auditability: from any tank you can list every weighnote (→ GRDOCKET → vineyard block) or purchase chit (→ POHEADER) that contributed litres/tonnes, pro-rata.

Vintage cycle: stages 1–2 compress into ~6–10 weeks per year; maturation runs 6 months to 5+ years. At each 30-June vintage roll, SUMCOMP.TANK.SV snapshots every tank; rebuilds only recalculate the current vintage.

## Key relationships (join keys for TRANS lookups)
| From | Rel | To | Via | Meaning |
|---|---|---|---|---|
| CHIT | 1→many | OPSMOVE | MCNOTE | Chit generates one OPSMOVE per movement line on completion |
| OPSMOVE | many→1 | MTANK | MFTANK / MTTANK | Each opsmove has a FROM tank and a TO tank |
| OPSMOVE | many→1 | MOPS | MOPCD | Operation code (ADD, MOVE, FILTER, PUMP...) |
| OPSMOVE | many→1 | MADD | MADDCD | Additive master when the op is an addition |
| OPSMOVE | many→1 | MBLEND | DEST.BLEND / FROM BLEND | Blend code either side of the move |
| OPSMOVE | many→1 | MBATCH | BATCH.CODE | Batch within the blend |
| MTANK | 1→many | TANK.HIST | TANK | Every chit that touched a tank |
| MTANK | 1→1 | BWALLOC | TANK | Current allocation per tank |
| MTANK | many→1 | DIPTABLE | TTABLE | Dip → volume conversion |
| GRAPE.DEST | many→1 | MTANK | TANK (@ID) | Contributing weighnotes/purchases per tank |
| GRAPE.DEST | many→1 | GRDOCKET / POHEADER | SOURCE.DOC | Source = weighnote or purchase chit |
| CHEM | many→1 | MTANK | CTANK | Chemistry readings on a tank |
| CHEM | many→1 | MBLEND | CBLEND | Readings tagged to the blend in tank at test time |
| SAP.PR.ORDER | 1→many | CHIT (bottling) | PR.ORDNO | JDE/SAP work orders referenced on bottling chits |
| SAP.PR.ORDER | refs | ITEM.MSTR / MBLEND | PROD.ITEM, PROD.BLEND | Produced & component items/blends |
| TANKER.MOVE | many→1 | WINERY.LOCATION | FLOCN / TLOCN | From/to winery locations |
| TANKER.MOVE | many→1 | POHEADER | ORDERNO | Purchase-type tanker moves link to a PO |
| MBLEND | parent | MBATCH | BLEND + BATCH.CODE | Blend → its batches |
| BLEND.PROP | proposes | MBLEND | BLEND | Accepted proposal becomes a blend |
| POHEADER | 1→many | PODETAIL | PONUM | PO header → lines |

## Universe concepts
- **Chit / Cellar Note (MCNOTE)**: a cellar work order, keyed AAAyynnnn = User * Vintage yr * sequence. Every action on wine starts as a chit. Old-style chits start in one screen and finish in another; new-style are all in one screen. CHIT holds them until complete; OPSMOVE holds the completed lines.
- **Blend code**: `vv w s bbb t` = vintage / winery / style / 3-digit number / type. Carries a wine's identity across tanks.
- **MPF (Master Planning Family)**: TWE's human term for the **generic code** (GENERIC / ALL.GENERIC fields). When labelling generic-code columns for people, use "MPF" — not "Blend" or "Generic".
- **@ID**: default primary key of every Universe file (first row of every dictionary).
- **Multivalues**: Depth column M in dictionaries. Repeating groups within one record. The Assoc column groups multivalued fields that align — value n of one field corresponds to value n of the others in the same association. Use WHEN to filter within multivalues; BY.EXP to explode and carry down single values.
- **I-types**: computed on demand from the algorithm in the Field Definition column. Cannot be referenced inside an EVAL — copy the definition instead.
- **Inversion files (I.*)**: pre-built indexes (I.OPLIST, I.CHEMLIST, I.MB.TANK = which tanks a blend is in).
- ***.HIST files**: archive companions (POHEADER.HIST, MBATCH.HIST, GRDOCKET.HIST — in BWMSTR).
- **ALL.GENERIC / ALL.VINT / ALL.LITRES** (BWALLOC-derived fields on MTANK/WIT): work for both in-tank AND intransit wine, whereas plain GENERIC/VINT/LITRES don't cover intransit. Prefer the ALL.* fields.
- **Rebuild**: only applies to the current vintage; SUMCOMP.TANK.SV is the rebuild baseline.
- **Process levels**: e.g. A = Raw Juice, G = Fermenting Wine, X = net to bottle (PROCESS.LEVEL.CODES). Whites across crusher PLEVEL = A, reds PLEVEL = G.
- **DE.TYPE** on OPSMOVE = despatch type (e.g. Bot = bottling chit).

## Which date field means what
**Chits**: MRDATE = requested date; MADATE = actual work started; in MTANK also OP.MADATE. CREATE.DATE/CREATE.TIME = when the chit was written (CHIT file).
**Chemistry (CHEM)**: CRDATE ≈ chit written / requested; RDATE ≈ sample/received; CDATE ≈ results. The exact sequence varies — the source workbook itself marks some orderings as uncertain, so verify against the CHEM dictionary and real data before relying on a specific interpretation.

## Glossary (cellar slang)
- **Lees** — what settles to the bottom of a tank if left standing
- **Fuging** — centrifuging
- **Desludge** — removing solid buildup in a centrifuge
- **Sludge** — solid build-up around the centrifuge bowl, removed every few minutes
- **Disto** — what is sent away for distillation, often including sludge
- **Floaties** — what rises to the top of a tank, especially after nitrogen is bubbled through
