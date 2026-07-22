# PLAN — proteomics-reanalysis

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated (with an opt-in funded lane for compute-heavy reruns)

Reproducible, openly-licensed reanalysis **pipelines** for open-access cancer proteomics —
starting from public **PRIDE / ProteomeXchange** datasets — where every result is reconstructable
from a pinned container + recorded parameters + the public source accession, and **every
quantitative assertion carries provenance** back to a specific dataset, software version, and run
manifest.

## Executive summary

Most published cancer-proteomics results are effectively irreproducible by a third party. The raw
data is increasingly deposited openly (PRIDE/ProteomeXchange now hold tens of thousands of
datasets), but the *analysis* that turned spectra into protein/peptide quantities and biological
conclusions is usually under-specified: exact tool versions, search parameters, FDR settings,
contaminant/decoy databases, normalization choices, and statistical models are missing,
inconsistent, or buried in a methods paragraph. Re-running a study from its deposited raw files is
often impossible, and small parameter differences can change which proteins are called
differentially abundant. This is a documented reproducibility crisis in the field.

This project builds **reproducible reanalysis pipelines** — containerized, version-pinned,
workflow-managed (Nextflow/nf-core-style or Snakemake) — that take an open PRIDE dataset and
produce a fully provenance-tracked reanalysis: identifications and quantities with controlled FDR,
a machine-readable **run manifest** (dataset accession, container digest, tool versions, every
parameter, input checksums), and a methods report a third party can re-execute to the same result.
The deliverables are the **pipelines, the run manifests, and the reanalysis methods/reports** — not
republished raw data and **not** medical advice.

The single most important constraint is **data scope and licensing** (see
[Data, licensing & compliance](#data-licensing--compliance)). **Only open-access / aggregate /
de-identified data is in scope.** Controlled-access repositories (dbGaP, EGA, individual-level
biobanks) and **any identifiable patient data are OUT OF SCOPE and never acceptable** — they require
authorized access and IRB oversight this project does not have. Each source's license is verified
per dataset (PRIDE/ProteomeXchange, TCGA, GEO open; UniProt CC-BY; **COSMIC and OncoKB are
non-commercial** and gated accordingly). A proteomics-specific re-identification risk —
**variant/proteogenomic peptides can leak germline-identifying information** — is treated as a
first-class compliance control, not an afterthought.

Risk tier: **medium** for the core research/engineering work (driven by license/PII rigor and
bioinformatics/statistical accuracy). **Any patient-facing or patient-advocate-facing output is a
separate, opt-in track at risk tier `high`**: education only, carrying a "not medical advice"
notice, and **blocked from release until an oncologist and a patient advocate have signed off** —
this review is a non-skippable gate, not a courtesy.

## Problem & beneficiaries

**The problem.** Open deposition solved *data availability* but not *analytical reproducibility*.
A researcher who downloads a cancer-proteomics dataset from PRIDE typically cannot reconstruct the
authors' results because: (1) the software stack (e.g., MaxQuant 1.6.x vs 2.x, FragPipe/MSFragger
versions, DIA-NN versions) is unpinned or unstated; (2) search parameters, modifications, enzyme
specificity, FDR thresholds, and the exact protein sequence + contaminant + decoy databases are
incompletely reported; (3) downstream normalization, imputation, batch correction, and statistical
modeling (e.g., MSstats / limma settings) are not shared as runnable code; and (4) environments
drift, so even the original code stops producing the original numbers. The result is wasted effort,
non-comparable studies, and conclusions that cannot be independently checked.

**Beneficiaries.**
- **Cancer proteomics researchers** who want to reuse, compare, or build on a public dataset
  without reverse-engineering its analysis.
- **Methods/bioinformatics developers** who need reproducible baselines and reference reanalyses to
  benchmark new tools fairly.
- **Reproducibility / open-science efforts** (e.g., curators, data stewards, journals) that need
  re-executable analyses tied to deposited data.
- **Students and trainees** learning reproducible proteomics from real, runnable cancer-data
  examples.
- **The broader public-good research commons** — every reproducible reanalysis raises the trust
  floor under cancer biology built on these datasets.
- **(Opt-in, high-risk track only)** patient advocates and the interested public, via *education
  only* plain-language explainers of what a reanalysis does and does not show — never treatment
  guidance.

**Verified need.** The *general* need (analytical irreproducibility in cancer proteomics) is real,
documented, and widely acknowledged. However, a **named partner / steward who will adopt, host,
cite, or formally request a specific reanalysis is TO BE SECURED.** Accordingly every task starts
`verifiedNeed: false`; it flips to `true` only when a named dataset author, lab, curator, journal,
or reproducibility initiative confirms they want and will use a specific reanalysis. Securing this
last-mile beneficiary is a first-class M0/M1 objective and a precondition for declaring any deed
*shipped* (per the Hee-Lee Oss "delivered, not merged" bar).

**Partner org.** TO BE SECURED. Candidate stewards: the **EMBL-EBI PRIDE** team / ProteomeXchange,
an academic proteomics core or reproducibility initiative, the **nf-core/proteomics** community,
a cancer-data consortium working with open data (e.g., CPTAC-adjacent open resources), or a journal
running reproducibility checks. Original dataset authors are pursued as per-reanalysis requestors.

## Goals and non-goals

**Goals**
- Publish **reproducible, containerized reanalysis pipelines** for open cancer-proteomics data, with
  pinned tool versions and a complete, machine-readable run manifest.
- Make **every reanalysis re-executable to the same result** by an independent party from the
  pipeline + manifest + the public source accession alone.
- Attach **provenance to every quantitative assertion** (which dataset, which container digest,
  which parameters, which databases, which FDR) — un-provenanced results are never published.
- Enforce a **license/PII/re-identification gate** so only open-access, de-identified data is ever
  processed, and every source's license is verified and recorded per dataset.
- Produce **transparent reproducibility reports**: concordance with the original publication where
  one exists, and an honest account of where and why results diverge.
- Provide reference reanalyses useful as **benchmarks/baselines** for the community.
- Secure at least one **partner/steward** who adopts, hosts, or cites a reanalysis.
- *(Opt-in track)* if patient-facing education is pursued, ship it **only** through the high-risk
  oncologist + patient-advocate review gate, framed as education with a "not medical advice" notice.

**Non-goals**
- **Not** processing or re-hosting any controlled-access or identifiable patient data (dbGaP, EGA,
  individual-level biobanks, clinical records). Hard refusal under Hee-Lee Oss guardrails.
- **Not** re-publishing or mirroring the raw mass-spec data itself (we reference accessions; the
  repository remains the source of record). Small derived/aggregate result tables only, where the
  source license permits.
- **Not** medical, diagnostic, prognostic, or treatment advice. No patient-specific interpretation.
- **Not** generating biomarker or "actionable target" claims for clinical use; reanalysis outputs
  are research artifacts, clearly labeled.
- **Not** a hosted compute service or a general LIMS; this is pipelines + manifests + reports.
- **Not** proteogenomic variant calling that could surface germline-identifying variants unless the
  re-identification control explicitly clears it (default: variant peptide identity is suppressed).
- **Not** laundering non-commercial reference data (COSMIC/OncoKB) into outputs that violate their
  terms.
- **Not** inventing results or "smoothing over" non-reproducibility; divergences are reported.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Baselines are zero at project start unless noted.

| Metric | Baseline | Target (first 12 months) |
| --- | --- | --- |
| Open PRIDE/ProteomeXchange datasets reanalyzed end-to-end with a published pipeline + run manifest | 0 | ≥ 5 |
| Share of published quantitative assertions carrying a resolvable provenance record (dataset + container digest + params) | n/a | 100% (hard CI gate; un-provenanced results are not published) |
| Reanalyses **independently re-executed to the same result** (by a second operator/machine, within the declared numeric tolerance) | 0 | 100% of published reanalyses (independent re-exec is part of Definition of Shipped) |
| Container/environment fully pinned by immutable digest (no floating tags) | n/a | 100% of pipelines |
| Datasets passing the license + PII + re-identification gate before any processing | n/a | 100% (no processing of an un-cleared dataset) |
| Reproducibility outcome reported honestly (concordance **or** documented divergence vs. the original) | n/a | 100% of reanalyses publish a concordance/divergence statement |
| Partner/steward adopting, hosting, citing, or requesting a reanalysis | 0 | ≥ 1 committed steward; ≥ 1 documented reuse/citation |
| Per-reanalysis methods reproducibility-audit pass (independent reviewer reproduces from manifest) | n/a | ≥ 95% of audited steps reproduce; 100% of *headline* results reproduce or are flagged |
| Controlled-access / identifiable-data incidents | 0 | **0** (hard guardrail; any incident halts the project) |
| Patient-facing outputs released **without** oncologist + advocate sign-off | 0 | **0** (hard high-risk gate) |

We explicitly **do not** count PRs merged, lines of pipeline code, or number of proteins
"identified" as success. A pipeline that runs but is not independently reproducible, or a result
without provenance, is a **failure** under this plan.

## Scope

**In scope**
- Reanalysis pipelines for **open-access** cancer proteomics from PRIDE/ProteomeXchange (and, where
  open and license-cleared, complementary TCGA/GEO/CPTAC-open layers for annotation).
- DDA (data-dependent) reanalysis first (e.g., MaxQuant / FragPipe-MSFragger / OpenMS), then DIA
  (e.g., DIA-NN) as a second pipeline.
- FDR-controlled identification and label-free / labeled quantification with documented statistics
  (e.g., MSstats / limma) — all version-pinned and containerized.
- A machine-readable **run manifest + provenance model** and a **dataset allow-list** with per-source
  license/PII/re-identification determinations.
- Reproducibility reports (concordance/divergence vs. original), reference baselines, datasheets for
  each reanalysis, and methods documentation.
- A small, license-permitting set of **derived aggregate result tables** (e.g., protein-level
  summaries) published alongside the pipeline.
- Partner/steward outreach and per-dataset requestor engagement (original authors, curators).
- *(Opt-in, gated)* patient/advocate-facing **education** about reproducibility, behind the
  high-risk review gate.

**Out of scope (explicit)**
- **Any controlled-access data** (dbGaP, EGA, individual-level biobanks) or **any identifiable
  patient data.** Hard refusal; no exceptions without authorized access + IRB, which is outside this
  project.
- **Re-hosting raw mass-spec files**; the repository remains the data of record.
- **Medical/diagnostic/prognostic/treatment advice** or patient-specific interpretation.
- **Clinical biomarker claims** or "use this target/drug" assertions.
- **Proteogenomic outputs that expose germline-identifying variants** unless the re-identification
  control clears them (default-deny).
- **Embedding non-commercial reference data** (COSMIC/OncoKB) in a way that violates their license,
  or relicensing it.
- For-profit packaging or any output whose primary beneficiary is a commercial entity.
- Inventing or "patching" results to manufacture reproducibility.

## Solution approach & architecture

A **data/method pipeline project** (containerized analysis code + documentation), not a hosted
service. The thesis: a reanalysis is only a good deed if anyone can re-run it and get the same
answer — so reproducibility infrastructure is the product, with the cancer datasets as its first
proving ground.

**Pipeline stages**
1. **Dataset intake & compliance gate** — each candidate dataset is logged in
   `datasets/allowlist.yml` with: accession (PXD/…), title, repository, instrument/acquisition
   (DDA/DIA/labeled), declared license, an explicit access classification
   (`open | controlled | unknown`), a PII / re-identification determination, and
   `status: approved | rejected | pending`. **Nothing is downloaded or processed until `approved`.**
   Controlled-access or unknown → rejected by default.
2. **Reference & database resolution** — protein sequence DB (e.g., UniProt, CC-BY, version + release
   recorded), contaminant DB, and decoy strategy are pinned and checksummed. Non-commercial
   references (COSMIC/OncoKB) are *not* embedded in outputs; if used for annotation at all, usage is
   gated by the license policy and kept out of redistributed artifacts.
3. **Containerized analysis** — the workflow runs inside pinned containers (referenced by **immutable
   digest**, never a floating tag) under a workflow manager (**Nextflow**, aligning with
   nf-core/proteomics conventions, with Snakemake considered). Stages: spectrum processing → search /
   identification → FDR control → quantification → statistical analysis. Each stage is
   deterministic-where-possible and records its inputs/outputs and checksums.
4. **FDR & statistics** — target-decoy FDR control at the documented level (typically 1% PSM/peptide/
   protein), with the decoy method and thresholds recorded; downstream differential analysis uses a
   pinned, documented model. A validation harness checks FDR is actually controlled on each run.
5. **Provenance binding** — a machine-readable **run manifest** wraps the whole run: dataset
   accession + file checksums, container digests, every tool version, every parameter, database
   versions/checksums, random seeds, and resource/time. **Every published quantitative assertion
   resolves to this manifest.** Un-provenanced outputs fail CI.
6. **Reproducibility verification** — an **independent re-execution** (second operator/machine)
   reproduces the headline results within a declared numeric tolerance before publication; the
   concordance/divergence vs. the original publication (if any) is recorded honestly.
7. **Publish** — versioned pipeline + manifest + methods report + datasheet + small license-permitted
   aggregate result tables. Raw data is *referenced by accession*, never re-hosted.

**Tech stack**
- Pipelines/tooling/validators: **TypeScript + ESM + pnpm** for the Hee-Lee Oss-facing tooling
  (manifest schema, validators, CI gates, allow-list linter); **Nextflow (DSL2)** /
  nf-core-style for the analysis workflow; analysis tools (MaxQuant, FragPipe/MSFragger, OpenMS,
  DIA-NN, Percolator, MSstats/R, Python/pandas) invoked inside pinned containers.
- Containers: **Docker / Apptainer (Singularity)**, referenced by **immutable digest**; images
  built from pinned base + pinned tool versions; SBOM recorded.
- Provenance: a JSON/YAML **run manifest** schema (PROV-aligned where practical), plus checksums
  (SHA-256) on every input/reference/output.
- Validation: a **provenance-completeness linter** + an **FDR/statistics validation harness** +
  schema validation in CI; golden fixtures use **synthetic or tiny public spectra**, never embedded
  patient-derived raw data.
- Result browsing: simple static reports (HTML/MD) over the manifest + aggregate tables; no accounts,
  no PII, no hosted compute.

**Data model (core artifacts)**
- **Dataset** (accession, repository, license, accessClass, instrument, acquisition, piiDetermination,
  reidDetermination, status).
- **Pipeline** (id, version, workflow, containerDigests[], toolVersions{}, referenceDbs[] with
  versions+checksums, fdrConfig, statConfig).
- **RunManifest** (datasetRef, pipelineRef, inputChecksums[], params{}, seeds, outputs[] with
  checksums, environment, startedAt/duration, operator).
- **ReanalysisResult** (manifestRef, aggregate result tables, FDR summary, concordance/divergence
  statement, license).
- **ProvenanceRecord** — binds each published assertion to its RunManifest at the result-table /
  comparison granularity.

**Key decisions (to ratify in M0)**
- **Workflow manager & container runtime:** Nextflow (nf-core/proteomics alignment) vs. Snakemake;
  Docker vs. Apptainer for HPC portability. (Leaning: Nextflow + Apptainer for portability; decide
  in M0.) This decision **defines the reproducibility substrate** all later work depends on.
- **The countable "assertion" unit** for the 100%-provenance gate — e.g., one published result-table
  row / one differential comparison / one summary statistic — so the gate is mechanically checkable.
- **Numeric reproducibility tolerance** — exact-match where determinism is achievable, else a
  declared, justified tolerance band per stage (search engines and some quant steps are not
  bit-reproducible across hardware). The tolerance is published with each reanalysis.
- **Re-identification stance for variant/proteogenomic peptides** — default-deny: identity-revealing
  variant peptides are not produced/published unless the re-identification policy clears the dataset
  and use.

## Data, licensing & compliance

**This is the headline gate. Read this section before any data is touched. The cancer guardrails
below are binding and override convenience.**

### Hard boundary — cancer data scope (binding)
**ONLY open-access / aggregate / de-identified data is in scope.** The following are **OUT OF SCOPE
and never acceptable** under Hee-Lee Oss guardrails:
- **Controlled-access repositories** — **dbGaP, EGA**, and any individual-level biobank or registry
  requiring an access committee / Data Use Agreement / IRB. These need authorized access + IRB
  oversight this project does not have.
- **Any identifiable patient data** — clinical records, individual-level genotypes, or any dataset
  where a person could be re-identified.
- **Proteogenomic variant outputs that leak germline-identifying information.** Mass-spec data can,
  via variant ("single-amino-acid-variant") peptides, encode germline SNP information functionally
  equivalent to a genetic identifier. Even from *open* raw files, producing/publishing
  identity-revealing variant peptides is treated as a re-identification risk and is **default-denied**
  (see re-identification control below).

If a task even proposes touching controlled-access or identifiable data, it is **refused and
flagged** per Hee-Lee Oss refusal guardrails — not deprioritized.

### Approved data sources (open-access only; verified per dataset)
Every dataset must be entered in `datasets/allowlist.yml` with a recorded license + access class +
PII/re-id determination and an `approved` status before any download:
- **PRIDE / ProteomeXchange** (EMBL-EBI) — the primary source: open-access proteomics depositions.
  ProteomeXchange/PRIDE public datasets are openly available; **the exact license/usage terms are
  verified per accession** (many are effectively CC0/CC-BY; do not assume — record the per-dataset
  statement). Only `public` datasets; never a `private`/reviewer-access dataset.
- **UniProt** — protein sequence reference databases — **CC-BY-4.0** (attribution recorded; release
  version pinned).
- **TCGA / GEO** open-access layers — **open** (verify the specific accession is open, not
  controlled); used for annotation/cross-reference only.
- **CPTAC open resources** — used only where the specific resource is open-access (CPTAC has both
  open and controlled components; verify per resource).
- **COSMIC** and **OncoKB** — **NON-COMMERCIAL** licenses. Treated as restricted: may be consulted
  for annotation only under their terms, **never embedded in or redistributed with outputs**, never
  relicensed. Default: avoid in redistributed artifacts; if unavoidable for annotation, the license
  policy task governs and the output records the restriction. (Hee-Lee Oss is non-profit/public-good, but
  outputs are openly licensed for *all* reuse including commercial, which is incompatible with NC
  source data — hence the embed ban.)

**Caveats we will not gloss over:** "open" must be verified for the *specific accession and copy*
used, not assumed; a dataset can be open while a derived annotation source is not; and a
nominally-open dataset can still carry re-identification risk through variant peptides. Each
allow-list entry records this analysis with a cited license URL + snapshot.

### Provenance model
- **Every published quantitative assertion** resolves to a **run manifest** recording dataset
  accession + input checksums, container digests, tool versions, all parameters, reference-DB
  versions/checksums, FDR settings, seeds, and operator/environment.
- Un-provenanced assertions are **never published** — enforced as a CI gate over the countable
  assertion unit fixed in M0.
- Divergence from the original publication is recorded, not hidden.

### Privacy / PII / re-identification stance
- **Open-access, de-identified data only.** No identifiable patient data, ever.
- **Re-identification control (proteogenomics):** by default, the pipeline does **not** emit
  identity-revealing variant peptides or per-individual genotype-equivalent calls. Any proteogenomic
  search that could surface germline variants requires the License+Compliance reviewer to clear the
  specific dataset and use; absent clearance, variant identity is suppressed/aggregated. Sample-level
  outputs are kept aggregate where individual-level results add re-identification risk without
  research need.
- **No raw re-hosting:** raw files stay at the source repository; we publish pipelines, manifests,
  reports, and small license-permitted aggregate tables.
- **No contributor PII** beyond standard opt-in open-source attribution.

### Attribution & output license
- **Code/pipelines:** **MIT** (or Apache-2.0 if the community prefers — decided in M0).
- **Documentation / reports / datasheets / aggregate result tables:** **CC-BY-4.0**, with explicit
  attribution to the **original dataset (accession + authors)**, the repository (PRIDE/
  ProteomeXchange), and reference databases (UniProt CC-BY attribution required).
- **Non-commercial source data (COSMIC/OncoKB) is never relicensed or embedded** in CC-BY/MIT
  outputs.

### Medical-content boundary (binding)
- Reanalysis outputs are **research artifacts**, not clinical guidance. No diagnostic, prognostic,
  or treatment claims.
- **Any patient-facing or patient-advocate-facing output is risk tier `high`**, is **education
  only**, must carry a prominent **"not medical advice"** notice, must cite sources, and is
  **blocked from release until a credentialed oncologist *and* a patient advocate sign off.** This is
  a non-skippable gate.

## Quality, review & risk gates

**Risk tier: medium** for core research/engineering; **high** for any patient-facing track. Review
dimensions, all required before a deed is "done":

1. **License / PII / re-identification review (primary gate, medium).** Before any dataset is
   downloaded, the License+Compliance reviewer confirms the `allowlist.yml` entry: open-access (not
   controlled), de-identified, license verified for the specific accession, COSMIC/OncoKB not
   embedded, and the re-identification determination recorded. No processing task starts against a
   `pending`/`rejected` dataset. Any proposal touching controlled/identifiable data is **refused and
   flagged**.
2. **Bioinformatics / statistical methods review (medium).** A reviewer with proteomics-analysis
   skill confirms search parameters, FDR control, quantification, and statistical model are sound and
   defensible, and that the reproducibility report's concordance/divergence claims are accurate.
3. **Reproducibility verification (medium, blocking).** Headline results are **independently
   re-executed** (second operator/machine) from the pipeline + manifest within the declared
   tolerance before publication. A reanalysis that cannot be independently reproduced is not shipped;
   the blocker is surfaced.
4. **Oncologist + patient-advocate review (high, blocking — patient-facing track only).** No
   patient-facing/education output is released without **both** a credentialed oncologist and a
   patient advocate signing off, plus the "not medical advice" framing. Non-skippable.

**Definition of Shipped (project level).** ≥1 open PRIDE cancer dataset reanalyzed end-to-end with:
a published containerized pipeline (immutable digests); a complete run manifest; 100% provenance on
published assertions; FDR validated; an honest concordance/divergence statement; **independent
re-execution to the same result within tolerance**; license/PII/re-id gate passed; and **at least
one partner/steward that has adopted, hosted, cited, or formally requested it.** Per Hee-Lee Oss,
*delivered ≠ merged* — the reanalysis must be in a beneficiary's hands and reproducible.

**Per-deed Definition of Done.** Acceptance criteria met + CI green
(schema/provenance/FDR/lint, `pnpm build && pnpm test && pnpm lint`) + license/PII/re-id review
passed + methods review passed + independent re-execution passed + (for patient-facing) oncologist +
advocate sign-off + output published under the declared license + DCO sign-off.

## Roadmap & milestones

**M0 — Foundation & compliance spine (cold-start).**
Goal: build the rails so no data work can bypass the license/PII/re-id and provenance gates.
Exit criteria: (a) `datasets/allowlist.yml` schema defined with ≥3 candidate datasets analyzed and
≥1 `approved` (open-access verified); (b) license + PII + **re-identification** gate checklist
published; (c) reproducibility standard published (containers by immutable digest, version pinning,
run-manifest schema, numeric-tolerance policy) **and the countable "assertion" unit fixed**;
(d) containerized pipeline skeleton runs a trivial end-to-end smoke test in CI; (e) provenance-
completeness linter + FDR validation harness scaffolded in CI; (f) a **qualified License+Compliance
reviewer named** (hard exit; documented fallback if the seat is empty — M0 cannot exit and
escalation begins); (g) partner/steward outreach started and ≥1 candidate dataset requestor
contacted (status logged); (h) workflow-manager + container-runtime + output-license decisions
ratified.

**M1 — First reproducible reanalysis (proof of pipeline).**
Goal: take one approved open PRIDE cancer DDA dataset end-to-end into a provenance-tracked,
FDR-controlled reanalysis.
**Hard entry precondition:** the pilot dataset is `approved` in the allow-list (open-access verified,
de-identified, license recorded, re-id determination made) before any download.
Exit criteria: (a) one open PRIDE DDA cancer dataset reanalyzed end-to-end in pinned containers;
(b) complete run manifest produced; 100% of published assertions carry provenance and pass CI;
(c) FDR validated by the harness; (d) reproducibility report with an honest concordance/divergence
statement vs. the original publication; (e) headline results **independently re-executed** to the
same result within the declared tolerance; (f) methods review + license review passed; (g) ≥1
candidate steward/requestor in conversation. Depends on M0.

**M2 — Verification, DIA, and benchmarking (robustness).**
Goal: prove the reproducibility claim holds across operators and acquisition types.
Exit criteria: (a) a second pipeline (DIA, e.g., DIA-NN) containerized and validated; (b) a documented
independent reproducibility audit of M1 (different machine/operator) passes; (c) a concordance
benchmark vs. published results for ≥2 datasets; (d) per-reanalysis **datasheets** published;
(e) reuse/citation tracking in place. Depends on M1.

**M3 — Scale, catalog & partner adoption (shipped).**
Goal: broaden coverage and lock in real-world use.
Exit criteria: (a) ≥5 open datasets reanalyzed cumulatively with manifests + datasheets;
(b) ≥1 partner has **adopted, hosted, cited, or requested** a reanalysis (Definition of Shipped met);
(c) sustainability/refresh plan in effect (drift detection on container/tool/DB versions);
(d) ≥1 documented external reuse event. Depends on M2.

**M4 — (Opt-in) Patient/advocate education layer (HIGH risk; gated, may be deferred indefinitely).**
Goal: *if pursued*, explain — for advocates/public — what a reproducible reanalysis does and does not
show. Education only.
Exit criteria: (a) plain-language explainer drafted with sources and a prominent "not medical advice"
notice; (b) **blocking** oncologist + patient-advocate sign-off recorded before any release;
(c) no patient-specific or treatment content. Depends on M3 and is **strictly optional** — the core
project ships without it.

## Work breakdown

The itemized, schema-mapped backlog lives in [`TASKS.md`](./TASKS.md), organized by the milestones
above (M0–M4) plus a sized backlog and an opt-in funded-lane option for compute-heavy reruns. Each
task maps to a Hee-Lee Oss Task JSON and carries a type, size, risk tier, deliverable, dependencies, and
reviewer. M0 deliberately front-loads the compliance and reproducibility guardrails before any
dataset is processed.

## Governance, roles & stakeholders

- **Maintainer / Owner:** TBD — accountable for scope, the compliance gate, releases, and the
  "delivered, not merged" bar.
- **License + Compliance reviewer:** TBD — must approve every `allowlist.yml` entry; has veto over
  any dataset; owns the controlled-access refusal and the re-identification determination. **Naming a
  qualified person is a hard M0 exit criterion.** **Documented fallback if the seat is empty:** no
  dataset advances past `pending`, no download/processing begins, M0 cannot exit; the maintainer
  escalates to Hee-Lee Oss governance/board to source a qualified reviewer before any data work proceeds.
- **Bioinformatics / methods reviewers (rotation):** proteomics analysts who review search/FDR/quant/
  stats and the concordance claims. TO BE SECURED.
- **Reproducibility verifier:** the independent operator (rotated) who re-executes headline results
  on a different machine. May overlap with the methods reviewer but must be a *different* person from
  the original runner.
- **Oncologist reviewer (high-risk track):** credentialed clinician — **blocking** sign-off on any
  patient-facing output. TO BE SECURED before M4 starts.
- **Patient-advocate reviewer (high-risk track):** **blocking** sign-off on any patient-facing
  output for clarity, safety, and tone. TO BE SECURED before M4 starts.
- **Steward (last-mile owner):** the partner who adopts/hosts/cites/requests a reanalysis. **TO BE
  SECURED** — required for "shipped."
- **Partner / requestor:** original dataset authors, curators, a reproducibility initiative, or the
  PRIDE/nf-core community; a named representative is TO BE SECURED.
- **Hee-Lee Oss governance/board:** arbiter for edge cases (borderline license, borderline re-id risk)
  under the published conflict-of-interest / veto checklist.

## Dependencies & integrations

- **PRIDE / ProteomeXchange** (EMBL-EBI) — primary open-access dataset source (intake; verify
  per-accession terms; public datasets only).
- **UniProt** (CC-BY-4.0) — protein sequence reference databases (pinned releases).
- **TCGA / GEO / CPTAC (open layers only)** — optional annotation/cross-reference.
- **COSMIC / OncoKB** (non-commercial) — annotation only, under their terms, never embedded/
  redistributed.
- **Analysis tools** — MaxQuant, FragPipe/MSFragger, OpenMS, DIA-NN, Percolator, MSstats (R),
  Python/pandas — all pinned in containers (verify each tool's own license for redistribution of
  binaries vs. invoking them).
- **Workflow & containers** — Nextflow (nf-core/proteomics alignment), Docker/Apptainer; SBOM
  tooling.
- **Hee-Lee Oss pieces:** Task schema (`packages/schema`), CLI workspace/PR flow (`packages/cli`,
  `packages/core`), the **funded runner** (`packages/runner`) for any opt-in funded reruns with a
  hard budget cap, governance proposal/registry process. Donated lane by default — humans run their
  own agents.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Controlled-access or identifiable patient data is processed (incl. a dataset misclassified as open) | Low | Critical (guardrail breach, project-ending, privacy harm) | Hard out-of-scope rule; allow-list gate with explicit access classification; default-deny on `unknown`; License+Compliance reviewer veto; CI blocks any dataset not `approved`; refusal + flag per guardrails | License+Compliance reviewer |
| Re-identification via variant/proteogenomic peptides leaking germline info | Medium | High (privacy/re-id) | Default-deny variant identity output; re-identification determination per dataset; aggregate-only individual results; reviewer clearance required to enable any variant search | License+Compliance reviewer |
| Reanalysis is not independently reproducible (env drift, non-determinism) | Medium | High (defeats the project's purpose) | Containers by immutable digest; full run manifest; declared numeric tolerance; **mandatory independent re-execution** before publish; honest divergence reporting | Reproducibility verifier |
| "Open" dataset actually carries restrictive/unclear license for the specific copy | Medium | High | Per-accession license verification + cited URL + snapshot in allow-list; no assume-open; reviewer sign-off | License+Compliance reviewer |
| COSMIC/OncoKB (non-commercial) data embedded in CC-BY/MIT outputs | Medium | High (license violation) | Embed ban; annotation-only-under-terms policy; output scan for restricted content; reviewer check | License+Compliance reviewer |
| Incorrect FDR / search params / stats → wrong biological conclusions | Medium | High | FDR validation harness; methods review (mandatory); pinned, documented parameters; concordance benchmark vs. original | Methods reviewer |
| No steward/partner secured → "delivered ≠ merged" not met | High | High | Partner/requestor outreach is an M0/M1 deliverable; engage dataset authors + PRIDE/nf-core + reproducibility initiatives; log status honestly | Maintainer |
| Patient-facing content ships without oncologist + advocate sign-off | Low | Critical (high-risk guardrail) | M4 is opt-in and fully gated; blocking dual sign-off; "not medical advice" framing; core project ships without M4 | Maintainer |
| Compute cost/time for large datasets blows up (esp. funded lane) | Medium | Medium | Donated lane default; funded reruns only via `packages/runner` with a hard per-task budget cap; size datasets to session budgets; prefer smaller pilot datasets first | Maintainer |
| Tool/DB/container versions drift, breaking reproducibility over time | Medium | Medium | Immutable digests; SBOM; version drift detection in maintenance; versioned releases with changelogs | Maintainer |
| Reviewer capacity exhausted (license + methods + verification bottleneck) | Medium | High | Sampling-based methods review; rotation with response-time SLA; throughput ceiling that throttles intake when backlog exceeds it | Maintainer |
| Misuse: a reanalysis result quoted as clinical advice | Medium | High | Clear "research artifact / not medical advice" labeling on every output; no actionable-target/biomarker claims; refusal of clinical-advice tasks | Maintainer |

## Security & privacy

- **Threat surface:** small (data/method project; no accounts, no hosted user PII). Primary risks are
  *compliance* (processing out-of-scope data), *re-identification* (variant peptides), and *data
  integrity* (un-provenanced or irreproducible results) — addressed by the gates above.
- **Secrets handling:** open-access datasets need no credentials; any incidental tokens (e.g., a
  container registry) stay out of logs, receipts, and commits per Hee-Lee Oss rules. The donated lane never
  runs headless or authenticates an agent. The funded lane runs only via `packages/runner` on an
  Anthropic API key with a hard per-task budget cap and never exceeds escrow.
- **PII / re-identification:** open-access, de-identified data only; default-deny on variant
  identity; aggregate-preferred individual outputs; no raw re-hosting; controlled/identifiable data
  refused and flagged.
- **Abuse/misuse prevention:** outputs are provenance-linked and labeled research artifacts; the
  project refuses and flags any attempt to ingest controlled/identifiable data, surface germline
  identity, or repackage results as clinical advice. No surveillance, profiling, or patient-targeting
  use is supported.

## Sustainability & maintenance

- **After delivery,** the maintainer plus the secured steward own ongoing curation; pipelines are
  versioned and containers pinned by digest so a reanalysis stays reproducible even as upstream tools
  evolve. If no steward is secured, the project stays in a maintained-but-not-shipped state and the
  gap is reported honestly rather than declared done.
- **Drift detection:** a maintenance process watches for tool/DB/container changes that would break
  reproducibility; stale pipelines become `maintenance` tasks with a fresh re-execution check.
- **Reviewer sustainability:** methods review runs on sampling (not exhaustive re-check); reviewers
  work a rotation with a response-time SLA; a documented throughput ceiling throttles new dataset
  intake when the review/verification backlog exceeds it, so gates never silently degrade.
- **Outcome tracking:** quarterly report on datasets reanalyzed, provenance completeness (must stay
  100%), independent-reproduction pass rate, concordance/divergence outcomes, partner adoptions, and
  reuse/citations.
- **Funded-lane discipline:** any compute-heavy rerun via `packages/runner` carries a hard budget cap
  and a public cost note; never exceeds the task's escrow.

## Open questions

- Who is the committed steward / adopting partner, and which dataset author(s) will act as
  per-reanalysis requestors? (TO BE SECURED — blocks "shipped.")
- Final workflow manager + container runtime: Nextflow + Apptainer vs. alternatives? (Decided in M0.)
- Output code license: MIT vs. Apache-2.0? (Decided in M0; docs/aggregate tables are CC-BY-4.0.)
- The exact numeric reproducibility tolerance per pipeline stage where bit-reproducibility is not
  achievable across hardware — what band is defensible and how is it justified per stage?
- Will any reanalysis ever need a proteogenomic/variant search, and under what re-identification
  clearance? (Default-deny until cleared.)
- Is the opt-in M4 patient-facing track pursued at all, and if so who staffs the blocking oncologist
  + advocate sign-off?
- Which datasets are most valuable to reanalyze first (community demand vs. tractable size for
  donated sessions)?

## References

- Project proposal: `governance/proposals/proteomics-reanalysis.md` (TO BE CREATED)
- Hee-Lee Oss work rules: `CLAUDE.md`
- Good Deed Definition & risk tiers: `docs/good-deed-definition.md`
- Task JSON schema: `packages/schema/src/schemas.ts`
- Portfolio roadmap (cancer track guardrails): `planning/ROADMAP.md`
- PRIDE Archive / ProteomeXchange (EMBL-EBI) — open proteomics data deposition
- UniProt (CC-BY-4.0) — protein sequence reference databases
- COSMIC, OncoKB — non-commercial license terms (annotation only; never embedded)
- TCGA / GEO / CPTAC — open vs. controlled access distinctions; dbGaP / EGA — controlled access (out of scope)
- nf-core/proteomics; Nextflow; Docker/Apptainer — reproducible workflow conventions
- MaxQuant, FragPipe/MSFragger, OpenMS, DIA-NN, Percolator, MSstats — analysis tooling (versions pinned per pipeline)

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified during planning and have been **applied** to
the plan above (and to `TASKS.md`). Each is concrete and already reflected in the documents.

1. **Cancer data scope made the headline gate.** The Data/licensing section leads with the binding
   open-access/aggregate/de-identified-only rule and an explicit out-of-scope list (dbGaP, EGA,
   biobanks, identifiable data) — applied as a hard refusal, not a preference.
2. **Proteogenomic re-identification risk treated as first-class.** Added a default-deny control for
   variant/SAV peptides that can leak germline-identifying information, with reviewer clearance
   required to enable any variant search — a risk generic proteomics plans miss.
3. **Reproducibility operationalized, not asserted.** Containers must be referenced by **immutable
   digest** (no floating tags), and a full **run manifest** binds every result to dataset + digest +
   params + DB versions + seeds.
4. **Independent re-execution made a blocking ship gate.** Headline results must be reproduced by a
   *different operator/machine* within a declared tolerance before publication — added to Definition
   of Shipped, metrics, and a dedicated task.
5. **Numeric-tolerance policy added.** Acknowledges that search engines/quant are not bit-reproducible
   across hardware; requires a declared, justified per-stage tolerance published with each reanalysis.
6. **COSMIC/OncoKB non-commercial embed ban.** Explicit policy that NC reference data is annotation-
   only-under-terms and **never embedded/redistributed/relicensed** in CC-BY/MIT outputs, with an
   output scan to enforce it.
7. **Per-accession license verification (no assume-open).** Even PRIDE datasets have their specific
   license verified and snapshotted; `unknown` access class is default-rejected.
8. **License-snapshot provenance.** License URL + snapshot recorded per dataset (mirrors the house
   pattern in sibling projects) so license claims are auditable later.
9. **Countable "assertion" unit fixed in M0.** Makes the 100%-provenance CI gate mechanically
   checkable rather than aspirational (e.g., per result-table row / per comparison).
10. **FDR validation harness as a CI gate.** Adds an automated check that target-decoy FDR is actually
    controlled on each run, not just configured.
11. **Patient-facing work isolated into an opt-in HIGH-risk track (M4).** Core project ships without
    it; M4 is strictly optional and fully gated.
12. **Oncologist + patient-advocate sign-off made a blocking, non-skippable gate** for any
    patient-facing output, with a "not medical advice" notice required — per the cancer guardrails.
13. **Honest verifiedNeed = false everywhere** until a named requestor confirms a specific reanalysis;
    partner/steward securing is an explicit M0/M1 deliverable and a ship precondition.
14. **License+Compliance reviewer naming is a hard M0 exit** with a documented empty-seat fallback and
    escalation path (mirrors the strongest sibling plans).
15. **Reviewer separation enforced.** The reproducibility verifier must be a *different person* from
    the original runner — prevents self-validation.
16. **No raw-data re-hosting.** Pipelines/manifests/reports + small license-permitted aggregate tables
    only; the repository stays the data of record (reduces license and storage risk).
17. **Outcome-based metrics with anti-vanity clause.** Explicitly rejects PRs/lines/protein-count as
    success; counts reproducible, provenance-complete, adopted reanalyses.
18. **Concordance/divergence honesty requirement.** Every reanalysis must publish how it compares to
    the original (including documented divergence) — non-reproducibility is reported, never hidden.
19. **Funded-lane discipline wired in.** Compute-heavy reruns run only via `packages/runner` with a
    hard per-task budget cap and a public cost note; donated lane is the default.
20. **Golden fixtures use synthetic/tiny public spectra only** — never embedded patient-derived raw
    data — so CI never ships restricted data.
21. **Version drift detection in maintenance.** A process flags tool/DB/container changes that would
    break reproducibility, turning stale pipelines into maintenance tasks.
22. **Throughput ceiling + reviewer rotation/SLA** to keep the license/methods/verification gates from
    silently degrading under load (sustainability).
23. **Risk register expanded** to include re-identification, NC-data embedding, dataset
    misclassification, env drift, and clinical-misuse — each with a named owner.
24. **Schema-faithful TASKS.md** with deliverables constrained to the enum (`pr`/`document`;
    never an unsupported "dataset" output for raw data) and a funded-task example carrying
    `fundedBudgetUsd` per the schema's conditional requirement.
25. **Explicit refusal hooks** for clinical-advice tasks and any controlled/identifiable-data proposal
    (refuse + flag), wired into the quality gates, risks, and security sections per Hee-Lee Oss guardrails.

---

## Review sign-off

A completeness + correctness review was performed against the PLAN_SPEC 17-section structure, the
Hee-Lee Oss CLAUDE.md work rules, the Good Deed Definition + risk tiers, the ROADMAP cancer guardrails,
and the Task JSON schema. Findings and fixes:

- **Section completeness:** all 17 required H2 sections are present and in the mandated order; the
  Data/licensing section leads with the binding cancer guardrails. ✔
- **Cancer guardrails:** open-access/aggregate/de-identified-only is stated as binding and applied as
  a hard refusal; dbGaP/EGA/biobanks/identifiable data are explicitly out of scope; per-source
  license verification (PRIDE/TCGA/GEO open; COSMIC/OncoKB non-commercial) is recorded; provenance is
  required on every assertion. ✔
- **High-risk gate:** patient-facing content is isolated to opt-in M4 at `riskTier: high` with a
  **blocking** oncologist + patient-advocate sign-off and "not medical advice" framing. ✔
- **Schema validity (TASKS.md):** all enums (`type`, `lane`, `priority`, `riskTier`, `deliverable`,
  `tokenEstimate`, `status`) checked against `schemas.ts`; required fields present; the funded-lane
  example includes `fundedBudgetUsd` per the schema's `if/then`; `deliverable` never set to an
  unsupported value; `verifiedNeed: false` throughout. Fixed during review: ensured the example JSON
  uses `additionalProperties`-clean fields only and a real `outputLicense`. ✔
- **Honesty:** no fabricated partner/requestor; `verifiedNeed: false` and `requestor: TO BE SECURED`
  everywhere; reproducibility outcomes (including divergence) reported honestly. ✔
- **Correctness fix applied:** reconciled the reproducibility-verifier role so it is explicitly a
  *different person* from the original runner (was implicit); aligned the metric "independent
  re-execution = 100% of published reanalyses" with the Definition of Shipped and M1 exit criteria.
- **Correctness fix applied:** clarified that COSMIC/OncoKB NC data is incompatible with the project's
  open output license and therefore embed-banned (not merely "cited carefully").
- **Residual open items (tracked in Open questions):** steward/requestor securing; workflow/runtime +
  code-license decisions; per-stage numeric tolerance; whether M4 is pursued. These are honest
  unknowns, not gaps in the plan.

Outcome: **plan is internally consistent, schema-faithful, and compliant with Hee-Lee Oss guardrails.**
Remaining blockers are external (secure reviewer + steward) and are tracked as hard M0/M1 exits.
