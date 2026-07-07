# Grange Dictionary Index

One CSV per Grange file (Universe table). Columns: Field Name, Type (D=data, I=interpretive, PH=phrase, X=misc), Field No (attribute location), Field Definition (algorithm for I-types), Conversion Code, Column Heading, Output Format, Depth (S=single value, M=multivalue), Assoc (multivalue association group), DESC (description), Key, Examples, Comment.

**How to use:** never guess a field name. Grep the relevant CSV, e.g.
`grep -i "volume" dictionaries/OPSMOVE.csv` or search across all files:
`grep -ril "baume" dictionaries/`. Row 1 of each CSV records the source and download date of that dictionary snapshot.

## Choosing the right file

### Grape intake (viticulture)
| File | Key | What it holds |
|---|---|---|
| MGROWER | GROWERID (Account) | Grower master: name, address, contracts, source area |
| MBLOCKS | BLOCK | Block master: name, clone, rootstock, hectares, distances |
| VINTINTAKE | VINTINTAKE.SEQ | Intake flags: baume/contract/flag/field grades, booking & crush locations, winemaker |
| GRAPERECPT | WNOTE | Weighnotes: account, variety, block, carrier, MOG, baume, delivery date, tonnages |
| GRDOCKET | MCNOTE | Crushnotes — grape receipt dockets attached to a crush note; tonnes by variety/source |
| EXTRATES | CHIT+seq | Extraction rates (litres/tonne) per crush chit |

### Tanks & vessels
| File | Key | What it holds |
|---|---|---|
| MTANK | TANK | Master of physical vessels (tank/barrel): volume, dip table, current blend/batch. Note: ALL.GENERIC / ALL.VINT / ALL.LITRES also work for intransit wine — prefer these over GENERIC/VINT/LITRES |
| TANK.HIST | TANK | History of chits that touched each tank |
| BWALLOC | TANK | Current allocation: wine (generic code/vintage/litres) held in each tank |
| WINE.IN.TANK | TANK | Working view — what wine is currently in each tank (for reports) |
| BARREL.GROUP | TANK | Barrel groups — ranges of barrel numbers treated as one tank |
| DIPTABLE | DTABLE | Dip → volume conversion tables |
| WOODMAT.TANK | TANK | Wood type (barrel/chips/planks) contributing to each tank |
| DP | BLEND/BATCH | Barrel store/stack locations per blend/batch |

### Chits / operations
| File | Key | What it holds |
|---|---|---|
| CHIT | MCNOTE | In-progress Cellar Operation Instruction (all operations on one chit, before splitting to OPSMOVE). MACTIV: R = Requested, A = Completed. Caution: late-added attrs MFGENERIC/MFALLOCVINT/MFALLOCLITRES (188-190) are blank while MACTIV='R' — TRANS to BWALLOC (GENERIC / VINT, same ALLOC assoc) instead |
| OPSMOVE | MCNOTE+seq | Completed chit movements — **the workhorse operations log** (one row per line on chit). Within one bulkwine account only |
| RECOVERIES | MCNOTE+seq | ~Last 3 months of OPSMOVE combined across ALL bulkwine accounts (same dictionary as OPSMOVE) |
| BW.STD.CST.OPS | CNOTE | OPSMOVE rolled up across sites (for standard costing) |
| OPSMOVE.SUMMARY | site*month | High-level summary of OPSMOVEs for bulk-wine movement reporting |
| BWALLOC.OPSMOVE | OPNOTE | Links BWALLOC allocations to OPSMOVE (vol in/out per chit) |
| ADD.NOTICE | MCNOTE | Additive notice — chit type for additions to wine |
| BWOPS.PRODMEAS | MCNOTE | Machine usage / time taken per chit |
| MOPS | OPCD | Operation code master (ADD, MOVE, FILTER, etc.) |
| I.OPLIST, I.OPSMOVE.OUTSTANDING, I.CP.OPSMOVE, I.CHEMLIST | — | Inversion files (pre-built lookup indexes) |
| OP.TRACE.CTL | — | Controls chits during rebuild |

### Composition / traceability
| File | Key | What it holds |
|---|---|---|
| GRAPE.DEST | TANK / CHIT | Links each tank/chit back to contributing weighnotes & purchase chits (litres/tonnes per source). The end-to-end traceability anchor. GRAPE.DEST.CHITS = same but for wine moved out (bottling/sale) |
| TANK.ORIGIN | TANK | Where wine in tank came from — simpler/less complete than GRAPE.DEST |
| SUMCOMP.TANK | TANK | Summary composition of each tank (variety:price:source:vintage : litres). SUMCOMP.TANK.SV = start-of-vintage snapshot |
| SUMCOMP.BOT | bottling ID | Summary composition of what has been bottled |
| SUMCOMP.CN | CNOTE | Summary composition of inter-winery cellar note movements |
| SUMCOMP.OP | CNOTE | Summary composition per chit (balancing only — can usually be ignored) |
| WINE.DB | VINT*GENCODE | Extra wine blend data (tirage/disgorge dates, maturation vessel, % vintage) |

### Chemistry
| File | Key | What it holds |
|---|---|---|
| CHEM | CHEMNO | Chemistry test results (pH, TA, SO2, etc.) per tank per date |
| CHEM.BOTT | CHIT*WC*TANK | Chemistry taken during a bottling run |
| MBLEND.CHEM | TBLEND.CNOTE | Chemistry attached to a blend on a cellar note |

### Bottling / bulk output
| File | Key | What it holds |
|---|---|---|
| SAP.PR.ORDER | PR.ORDNO | Work orders from JDE/SAP, used on bottling chits |
| TANKER.MOVE | — | Tanker movements: sales, purchases, inter-site transfers (incl. tirage; Karadoc tirage tanks prefixed U). Use FREIGHT.RATE for per-movement rate; avoid DE.FRT.COST and TRUCK.RATE |
| POHEADER / PODETAIL | PONUM / PONUM+LINE | Bulk wine purchase orders (header/lines). .HIST companions exist in BWMSTR |

### Master / reference data
| File | Key | What it holds |
|---|---|---|
| MBLEND | BLEND | Blend master (vintage-winery-style-NNN-type); current volume, status |
| MBATCH | BATCH.CODE | Batch master — sub-division within a blend |
| BLEND.PROP | PROPOSAL.NO | Blend proposals (recipes) before they become actual blends |
| MADD | ADDCD | Additive master (yeast, SO2, finings, etc.) |
| LOT / LOT.OUT | LOT.ID | Lot master (additives/finings) traced into wine |
| ITEM.MSTR | ITEM.CODE | Item master (JDE/Oracle product codes) |
| WINERY.LOCATION | WIN.CODE | Winery locations / sub-sites |
| WINE.STATUS | STATUS.CODE | Wine status codes (held, released, etc.) |
| DESCRIPTION | — | Reading templates / descriptions (chem templates, work orders, etc.) |
