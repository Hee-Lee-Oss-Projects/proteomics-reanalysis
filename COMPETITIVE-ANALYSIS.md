# Competitive & Improvement Analysis — `proteomics-reanalysis`

Scope reviewed: `PLAN.md` v0.1.0 (2026-06-28). Reproducible, openly-licensed reanalysis
pipelines for **open** cancer proteomics (PRIDE/ProteomeXchange, CPTAC open tier, MassIVE),
provenance-first, derived results + code only. Competitor claims are grounded in cited
sources (web research, June 2026).

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong on governance, licensing, and provenance, and it correctly
identifies analytical (not data-availability) irreproducibility as the real problem. The gaps
are concentrated in the **bioinformatics/statistical substance** — the part that actually
decides whether a reanalysis is correct. Concrete findings:

**A. The biggest under-specification: search engine / FDR / inference choices are named but not pinned to a defensible methodology.**
- The plan lists MaxQuant, FragPipe/MSFragger, OpenMS, DIA-NN, Percolator, MSstats but never
  states *which is the reference engine per acquisition mode*, nor how engine choice itself is
  treated as a variable. This is the single largest reproducibility lever in the field: the
  same raw file run through different engines (or engine versions) yields materially different
  protein lists, and **a lower FDR is typically observed in DDA than DIA searches** even on the
  same sample ([tear-fluid DIA/DDA comparison](https://www.sciencedirect.com/science/article/pii/S2667145X25000240)).
  Recommendation: declare a canonical engine per mode, and treat "engine concordance" (run ≥2
  engines, report overlap) as a first-class output, not an afterthought.
- **FDR at "typically 1% PSM/peptide/protein" is stated but the *protein-group* FDR problem is not.**
  Protein-level FDR is notoriously unstable and is an active research problem — see the
  [ProteomicsDB protein-group FDR reanalysis](https://www.mcponline.org/article/S1535-9476(22)00245-6/fulltext),
  which exists *specifically because* naive protein FDR is inaccurate at scale. The "FDR
  validation harness" must validate **protein-group/picked-FDR**, not just PSM-level decoy
  counting, or it will pass runs that are actually mis-calibrated. Entrapment/two-species
  ground-truth tests should be named.
- **Protein inference / parsimony / shared-peptide handling is entirely absent.** Whether the
  pipeline reports protein groups, gene-level inference, or razor-peptide assignment changes the
  count and identity of "differentially abundant" proteins. This must be a pinned, documented
  decision in the `Pipeline` data model.

**B. DDA vs DIA reproducibility nuances are flattened.** The plan treats DIA as "a second
pipeline" but does not capture that DIA quantitative fidelity depends heavily on the
downstream stat method and replicate count — at low replicate numbers, only DIA + limma/ROTS
produced high quantitative fidelity in benchmarking
([label-free benchmarking](https://pubs.acs.org/doi/10.1021/acsomega.0c04030)). The numeric-tolerance
policy should therefore be **per-acquisition-mode and replicate-aware**, not a single global band.

**C. Missing-value handling and batch effects are not mentioned at all — a major omission for a *cancer* differential-abundance project.**
- Missing values are *the* dominant nuisance in MS proteomics; imputation choice directly changes
  which proteins are called significant, and ten distinct imputation families behave differently
  ([imputation evaluation](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11728106/),
  [imputation + batch-effect comparison](https://www.medrxiv.org/content/10.1101/2025.08.14.25333694.full.pdf)).
- Cancer cohorts are multi-site/multi-batch by construction; **batch effects introduce
  non-biological variation that can mimic biology** if uncorrected
  ([FragPipe-Analyst](https://pmc.ncbi.nlm.nih.gov/articles/PMC10942459/)). A reanalysis that
  omits batch modeling can *manufacture* divergence from the original and then "honestly report"
  a divergence that is an artifact of the reanalysis itself. **Imputation method, normalization,
  and batch-correction model must be pinned fields in `statConfig`** and surfaced in every report.
- Differential-expression rule choice matters: ensemble/high-performing-rule approaches
  measurably outperform any single default
  ([Nat Commun DE optimization](https://www.nature.com/articles/s41467-024-47899-w)).

**D. Reference proteome versioning is under-specified.** UniProt is pinned by "release version,"
good — but the plan does not specify the **reference proteome choice** (canonical vs canonical+isoform,
one-protein-per-gene, contaminant set such as cRAP/MaxQuant contaminants, decoy generation
method). These choices change identification counts as much as engine version does. The
`referenceDbs[]` model should carry organism, proteome ID, isoform policy, contaminant-DB
identity+version, and decoy strategy — each checksummed.

**E. CPTAC open-vs-controlled handling is correct in spirit but slightly imprecise.** The plan
says "CPTAC has both open and controlled components; verify per resource" — accurate. To
sharpen: CPTAC **proteomics** lives in the Proteomic Data Commons and is "freely available to
the public, subject to the Data Use Agreement," while the **genomic** side (GDC) requires
**dbGaP** authorization ([CPTAC PDC](https://proteomic.datacommons.cancer.gov/pdc/),
[CPTAC-3 Open Data on AWS](https://registry.opendata.aws/cptac-3/)). The allow-list entry should
record the *specific PDC study + its DUA terms*, and flag that even "open" CPTAC carries a Data
Use Agreement (not pure CC0) — the plan's blanket "open = no credentials" note in Security needs
this caveat.

**F. Metadata quality (SDRF-Proteomics) is missing — and it is the natural backbone for this project.**
The plan never references **SDRF-Proteomics**, the PSI-official (May 2023) sample-and-data
relationship standard that defines the minimum experimental-design metadata enabling *automated
reprocessing* ([SDRF spec](https://www.psidev.info/sdrf-sample-data-relationship-format),
[bigbio repo](https://github.com/bigbio/proteomics-sample-metadata)). quantms already consumes
SDRF. A provenance-first reanalysis project that does not adopt/produce SDRF is reinventing a
weaker manifest and will not interoperate. **Strong recommendation: make SDRF-Proteomics the
required input/output metadata format**, with the run manifest layered on top.

**G. Statistical-validity gaps beyond FDR.** No mention of multiple-testing correction across
proteins (BH/q-value), minimum-peptides-per-protein thresholds, handling of fold-change vs
adjusted-p cutoffs, or power/replicate adequacy checks. The methods-review gate should have an
explicit statistical checklist, not just "sound and defensible."

**H. Compute scale is hand-waved.** "Size datasets to session budgets" and "prefer smaller pilot
datasets" is the only sizing guidance. A single large DIA cancer cohort can be tens of thousands
of instrument files (quantms reprocessed **29,354 files / 13,132 samples**;
[quantms, Nat Methods 2024](https://www.nature.com/articles/s41592-024-02343-1)). The donated
lane's interactive, human-runs-their-own-agent model is poorly matched to multi-day search jobs;
the plan needs explicit per-dataset compute envelopes, a "max tractable size for donated lane"
threshold, and clarity that real reruns will mostly be funded-lane or external HPC.

**I. Metric weakness — "independently re-executed to the same result" lacks an operational tolerance and a defined granularity.** The plan *acknowledges* bit-reproducibility is impossible
across hardware and defers the tolerance to M0, which is reasonable — but the headline metric is
stated as "100% ... within the declared tolerance" before that tolerance exists, so the metric is
currently uncheckable. Also "headline results" is undefined; fix the **assertion unit** and the
**headline-result set** together, or the gate is subjective.

**J. Concordance-vs-original is treated as a clean signal but is confounded.** "Divergence from
the original is recorded honestly" is good, but divergence has *many* causes (engine, DB version,
imputation, batch model, original-paper errors). Without a factor-attribution step, a divergence
statement is not actionable. Recommend a **divergence decomposition** ("differs because: DB
version X, imputation Y…") as part of the report template.

**Minor:** Snakemake is mentioned as alternative but nf-core/quantms is Nextflow — leaning
Nextflow is correct for ecosystem reuse. "Random seeds" are listed in the manifest, but most MS
search engines are not seed-controlled; the manifest should distinguish *seed-controllable*
stages from *non-deterministic* ones. No mention of `ppx`/`pridepy` for programmatic, checksummed
dataset retrieval ([ppx](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8457024/)) — an obvious
intake building block.

---

## 2. Competitive landscape

### Direct / closest: large-scale public-data reanalysis pipelines

**nf-core/quantms (+ pmultiqc, MSstats, SDRF)** — *the* incumbent and the most important
competitor. Cloud/HPC Nextflow pipeline that directly reprocesses any ProteomeXchange dataset,
supports DDA-LFQ, TMT (DDA-plex), and DIA-LFQ, auto-installs tools, emits standard formats, and
generates QC via pmultiqc; demonstrated at massive scale (83 datasets / 29,354 files / 13,132
samples / 16,599 proteins).
[Nat Methods 2024](https://www.nature.com/articles/s41592-024-02343-1) ·
[PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11399091/) · [nf-core/quantms](https://nf-co.re/quantms) ·
[pmultiqc demo](https://pmultiqc.quantms.org/PXD053068/multiqc_report.html).
*Strengths:* proven scale, nf-core reproducibility conventions, containerized, SDRF-native,
active community, already the de-facto standard for public reanalysis. *Weaknesses/gaps for us:*
**not cancer-focused**; no per-dataset license/PII/re-identification gate; no germline-variant
re-identification control; no "delivered to a named beneficiary/steward" outcome model; provenance
is pipeline-config-level, not an auditable per-assertion manifest with concordance-vs-original
honesty reporting; reanalyses are bulk/atlas-style, not curated-and-reviewed deeds.

**recount3 (RNA-seq, analog/benchmark)** — 750,000+ samples uniformly reprocessed by one pinned
pipeline (Monorail). [recount3](https://rna.recount.bio/) ·
[Genome Biology 2021](https://link.springer.com/article/10.1186/s13059-021-02533-6). The aspirational
"uniform reprocessing commons" model; proteomics has nothing of this completeness. *Gap:* it is
RNA, not proteomics, and not cancer-curated — a template to emulate, not a competitor.

### Repository-level reanalysis infrastructure

**MassIVE.quant** — repository infrastructure storing raw data, experimental-design metadata,
workflow scripts, intermediate files, and *alternative reanalyses* under reanalysis IDs (RMSV…).
[Nat Methods 2020](https://www.nature.com/articles/s41592-020-0955-0) ·
[PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC7541731/). *Strengths:* purpose-built for storing
multiple reproducible reanalyses with provenance, acquisition/tool-agnostic. *Weaknesses:* a
storage/structure resource, not a curated cancer-reanalysis program; no opinionated containerized
pipeline, no license/re-id gate, adoption modest.

**ProteomeXchange / PRIDE / ProteomeCentral** — the data backbone: 64,330 datasets through June
2025 (PRIDE = 77%), FAIR-committed, with internal **reanalysis "builds"** (e.g., PTM reanalyses
integrated into UniProtKB) and reanalysis accessions (**PRXD**).
[PX 2026, NAR](https://academic.oup.com/nar/article/54/D1/D459/8315797) ·
[PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC12807779/). *Strengths:* canonical source,
authoritative, increasingly does its own reanalysis. *Weaknesses:* reanalysis builds are
PTM/atlas-oriented, not cancer-differential-abundance with concordance reporting; PRIDE is the
*source of record* we depend on, and a natural **partner**, not a rival.

**jPOST** — ProteomeXchange member with an environment explicitly accelerating reuse/reanalysis of
public MS data. [jPOST reuse](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11701591/). Similar
posture to PRIDE; partner-shaped.

### Analysis platforms & toolchains (substrate, not deeds)

**FragPipe/MSFragger, MaxQuant, DIA-NN, OpenMS, Percolator, MSstats** — the engines our pipelines
wrap. [FragPipe-DIA](https://www.nature.com/articles/s41467-023-39869-5) ·
[DIA-NN](https://github.com/vdemichev/DiaNN) · [FragPipe-Analyst](https://pmc.ncbi.nlm.nih.gov/articles/PMC10942459/).
*Strengths:* best-in-class identification/quant. *Weaknesses:* they are tools, not reproducibility
programs — version drift, default-parameter opacity, and per-study config are exactly the
irreproducibility sources we exist to tame. Some (MaxQuant, DIA-NN) have license terms on binary
redistribution — the plan correctly flags "invoke vs redistribute."

**ProteomicsDB** — re-annotates/serves quantification across many datasets, and drove the
protein-group FDR reanalysis work.
[protein-group FDR](https://www.mcponline.org/article/S1535-9476(22)00245-6/fulltext). *Strengths:*
deep reprocessing + serving layer. *Weaknesses:* a database product, not open curated cancer deeds.

### Metadata / FAIR standards (to adopt, not beat)

**SDRF-Proteomics (PSI standard) + ReDU (metabolomics analog)** — the controlled-vocabulary
sample-metadata standards enabling automated reprocessing.
[SDRF](https://www.psidev.info/sdrf-sample-data-relationship-format) ·
[bigbio repo](https://github.com/bigbio/proteomics-sample-metadata) ·
[multiomics SDRF, Nat Commun](https://www.nature.com/articles/s41467-021-26111-3). ReDU is the
metabolomics metadata-capture system. [ReDU context](https://pmc.ncbi.nlm.nih.gov/articles/PMC8494749/).
These are complements: adopting SDRF is table-stakes for interoperability.

### Reproducibility-crisis efforts (validation of the problem)

The `reproteomics` codeathon ([repo](https://github.com/omicscodeathon/reproteomics)), the
dual-search reproducibility strategy ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC7896416/)),
and the PRIDE→Expression Atlas DIA-reuse pipeline
([Sci Data 2022](https://www.nature.com/articles/s41597-022-01380-9)) all confirm the crisis and
that "download-and-reproduce historically is impossible" — but none deliver a *cancer-focused,
license/PII-gated, per-assertion-provenanced, steward-delivered* reanalysis program.

---

## 3. Gaps we can fill

1. **Cancer-curated reanalyses with concordance-vs-original reporting.** quantms/recount produce
   atlas-scale matrices; nobody produces a *reviewed, per-dataset cancer reanalysis* that states
   honestly how it agrees or diverges from the original publication and *why*.
2. **License + PII + germline-re-identification gate as a hard, recorded, per-accession control.**
   No incumbent treats variant/SAV-peptide germline leakage as a default-deny compliance gate —
   and DIA can identify single-amino-acid variants
   ([DIA SAV](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11587465/)), so the risk is real, not
   theoretical. This is a genuine differentiator and a public-good safety contribution.
3. **Per-assertion provenance binding** (every published number resolves to a manifest), enforced
   as CI — stricter than pipeline-level config capture.
4. **Independent re-execution by a different operator** as a blocking ship gate — a stronger
   reproducibility claim than "the pipeline is deterministic."
5. **Engine/parameter sensitivity surfaced as a product**, not hidden — multi-engine concordance,
   imputation/batch-model sensitivity bands.
6. **A reusable, opinionated reprocessing *template*** (datasheet + manifest + SDRF + tolerance
   policy) other groups can drop onto a new PXD, lowering the barrier below even quantms.
7. **"Delivered to a named beneficiary"** outcome discipline — a steward/requestor actually
   adopting a reanalysis, vs. dumping matrices into a portal.

---

## 4. Differentiators to win (incl. vs nf-core/quantms)

- **Use quantms, don't fight it.** quantms is the better *engine harness*; our edge is the
  **governance, curation, provenance, and delivery wrapper** around it. Recommended posture:
  adopt nf-core/quantms (or wrap it) as the analysis substrate and differentiate on layers it
  lacks. Competing on raw pipeline breadth is a losing race.
- **Cancer focus + clinical-misuse guardrails.** A reanalysis program scoped to cancer with an
  explicit "research artifact / not medical advice" boundary and a gated high-risk education
  track. quantms is organism/disease-agnostic and ships no such guardrails.
- **Compliance-first, per-accession.** License snapshot + access-class + PII/re-id determination
  recorded before any download; CPTAC open-tier-only with DUA captured; COSMIC/OncoKB embed-ban.
  This is absent from every competitor and is exactly the Elyos guardrail surface.
- **Provenance honesty as the brand.** Per-assertion manifests + mandatory concordance/divergence
  statements + divergence decomposition. The promise: *every number is reconstructable and its
  agreement with the literature is stated.*
- **Independent re-execution** as a non-skippable gate (different operator/machine).
- **Donated-compute model + open licensing (MIT/CC-BY).** Spare human-run agent capacity does the
  curation/code/QC/doc work; funded lane only for compute-heavy reruns under a hard cap. This is a
  *sustainability and cost* differentiator no academic pipeline has.

---

## 5. Claude API leverage

**Where Claude genuinely helps (high-value, low-risk):**
1. **PRIDE/SDRF metadata harmonization (draft, human-verified).** Public proteomics metadata is
   notoriously messy; SDRF curation has historically been a manual crowdsourcing effort over
   datasets ([SDRF curation](https://www.nature.com/articles/s41467-021-26111-3)). Claude can
   *draft* SDRF-Proteomics rows from free-text PRIDE descriptions and methods sections, map terms
   to controlled vocabularies/ontologies, and flag ambiguities — a large force-multiplier on the
   slowest step. **Human curator verifies every term before it is authoritative.**
2. **Pipeline code + tests + manifest/linter scaffolding.** Claude writes/maintains the TypeScript
   provenance-manifest schema, validators, the FDR-validation harness scaffolding, the allow-list
   linter, Nextflow glue, and golden-fixture tests — exactly the agent-neutral tooling the plan
   centers on. Numeric *results* still come from the pipeline, never the model.
3. **QC report + reproducibility-report drafting.** Claude drafts the human-readable narrative over
   pmultiqc/MSstats outputs and the divergence-decomposition section, and writes datasheets and
   methods docs — strong on the documentation surface that the field chronically under-produces.

**Other useful uses:** license-text triage (surface terms for a human to decide), refusal-guardrail
checks on incoming task descriptions, drafting steward/requestor outreach, and explaining
parameter choices in plain language for the education track.

**Where Claude must NOT decide (hard boundaries):**
- **No numeric/statistical results from the LLM.** All identifications, quantities, FDR, fold
  changes, and p-values come from the pinned pipeline; Claude never estimates, "fills in," or
  smooths a number. Fabricated results are a project-ending failure under the plan's honesty rule.
- **Metadata curation is advisory only** — every Claude-proposed SDRF term/sample mapping is
  human-verified before it gates an analysis.
- **License / access-class / open-vs-controlled determinations are human-verified.** Claude may
  surface the license text and a recommendation; the named License+Compliance reviewer makes the
  binding call (especially CPTAC open-vs-controlled and DUA terms).
- **Re-identification / germline-variant clearance is human-only** (default-deny; reviewer
  clearance required).
- **No medical/clinical interpretation** — ever, and never without the high-risk oncologist +
  advocate gate.

---

## 6. Ten concrete optimizations

1. **Adopt nf-core/quantms as the analysis substrate** and position Elyos as the governance +
   provenance + curation + delivery layer; don't rebuild engine plumbing.
2. **Make SDRF-Proteomics a required input/output**, with the run manifest layered on top — gains
   interoperability with quantms/PRIDE and turns metadata harmonization into a shippable artifact.
3. **Pin the full reference stack explicitly:** organism + UniProt proteome ID + isoform policy +
   contaminant DB (cRAP) version + decoy method — each checksummed in `referenceDbs[]`.
4. **Upgrade the FDR harness to protein-group/picked FDR with entrapment/two-species ground truth**,
   not just PSM-level decoy counting.
5. **Add imputation, normalization, and batch-correction as first-class pinned `statConfig` fields**
   and report a sensitivity band for each (the choices that most change "significant" proteins).
6. **Run ≥2 search engines per dataset and publish engine-concordance** as a headline output —
   turns the field's worst irreproducibility source into a selling point.
7. **Replace the single global numeric tolerance with per-stage, per-acquisition-mode,
   replicate-aware tolerance bands**, published with each reanalysis.
8. **Ship a divergence-decomposition template** that attributes each concordance gap to a specific
   factor (DB version / engine / imputation / batch model / original-paper error).
9. **Use `ppx`/`pridepy` for checksummed programmatic intake** so input provenance is automatic and
   reproducible from accession alone ([ppx](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8457024/)).
10. **Publish per-dataset compute envelopes** (files, CPU-hours, RAM, wall-clock) and a
    "max-tractable-for-donated-lane" threshold; route anything larger to the funded lane or external
    HPC up front.

---

## 7. Parallel & perpendicular spin-offs

- **Reusable open reprocessing template** — a "drop-in PXD" kit (allow-list entry + SDRF +
  container-digest pins + manifest + tolerance policy + datasheet) that any lab can apply to a new
  dataset. The most leverage-rich spin-off; lowers the reanalysis barrier below quantms's.
- **MCP server serving harmonized protein matrices** — expose finished reanalyses (protein/peptide
  matrices + manifests + SDRF) over an MCP endpoint so downstream agents/tools query *provenanced*
  cancer-proteomics results directly. Natural Elyos-API integration and a public-good data service.
- **Tie to `ewing-expression-reanalysis`** — a shared multi-omics reanalysis spine: same
  provenance manifest, license gate, and concordance-reporting machinery across RNA (expression)
  and protein (proteomics) for the same cancer type; cross-omics concordance becomes a deliverable.
- **Tie to `ml-oncology-benchmarks`** — reproducible reanalyses are *exactly* the clean,
  provenanced baselines ML benchmarks need; export reanalysis matrices as benchmark inputs with
  guaranteed provenance.
- **Tie to `cancer-dataset-datasheets`** — every reanalysis emits a datasheet; co-develop the
  datasheet schema so it doubles as the dataset-intake record across the cancer portfolio.
- **Engine/parameter sensitivity atlas** — a standalone public resource quantifying how much
  engine/FDR/imputation choices move cancer differential-abundance calls (a citable methods
  contribution distinct from any single reanalysis).
- **SDRF auto-drafting service** — the human-verified Claude SDRF-curation workflow, generalized as
  a tool the broader PRIDE/nf-core community could use (partnership hook).

---

## 8. Open questions for the maintainer

1. **Build on quantms or build parallel?** Wrapping nf-core/quantms is strongly recommended — is
   there an appetite to depend on it, or a reason to stay fully independent?
2. **Is SDRF-Proteomics adopted as the metadata standard?** (Strongly recommended; affects schema
   design now.)
3. **Which canonical engine per acquisition mode**, and is multi-engine concordance in scope for M1
   or deferred?
4. **What protein-FDR method** (picked/entrapment) will the harness validate, and what is the
   ground-truth fixture?
5. **What is the M0 numeric tolerance** per stage/mode — and how is "headline result" defined so the
   100%-re-execution metric is mechanically checkable?
6. **CPTAC open-tier:** confirmed in-scope with DUA recorded per study? Does the DUA's terms
   conflict with CC-BY redistribution of *derived aggregate tables*?
7. **Realistic donated-lane compute ceiling** — what dataset size is actually runnable interactively
   vs. funded-lane only?
8. **Imputation/batch-correction defaults** — pinned project-wide, or per-dataset with justification?
9. **Steward target** — is PRIDE/nf-core/quantms community the most likely first steward, given we'd
   be building on their stack?

---

## Summary

`proteomics-reanalysis` is a governance- and provenance-strong plan whose weak flank is
bioinformatics substance. The **top 3 competitors** are **nf-core/quantms** (the incumbent
public-data reanalysis pipeline — proven at 29,354 files / 13,132 samples, SDRF-native, but
disease-agnostic and lacking any license/PII/re-identification gate), **MassIVE.quant** (repository
infrastructure that stores multiple reproducible reanalyses with provenance, but is storage not
curation), and **ProteomeXchange/PRIDE** (the FAIR data backbone now running its own PTM reanalysis
"builds" — more partner than rival). recount3 is the aspirational uniform-reprocessing analog from
RNA-seq.

The **single strongest differentiator** is the **compliance-and-provenance wrapper around a
best-in-class engine**: cancer-scoped, per-accession license + PII + *germline-variant
re-identification* gating, per-assertion provenance, mandatory independent re-execution, and honest
concordance-vs-original reporting — none of which the incumbents offer. The winning move is to
**build on quantms rather than rebuild it**.

**Top 3 Claude-API ideas:** (1) draft SDRF-Proteomics metadata harmonization from messy PRIDE
descriptions (human-verified before authoritative); (2) write/maintain the provenance-manifest
schema, validators, FDR-harness scaffolding, and tests; (3) draft QC/reproducibility reports,
divergence decompositions, and datasheets. Hard line: **no numeric/statistical results, no
license/re-id determinations, no medical interpretation from the LLM.**

**Two most important plan-correctness findings:** (1) the **search-engine / FDR / protein-inference
methodology is named but not pinned** — no canonical engine per mode, no protein-group-FDR
validation (only PSM-level decoy counting is implied), and no protein-inference policy, which is
the field's largest irreproducibility lever; (2) **missing-value imputation and batch-effect
correction are entirely absent**, even though they directly determine which proteins are called
differentially abundant and can make the project's own reanalysis *manufacture* a divergence it
then "honestly reports." Both should become pinned `statConfig` fields with published sensitivity
bands, and SDRF-Proteomics should be adopted as the metadata backbone.
