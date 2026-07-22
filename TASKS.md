# TASKS — proteomics-reanalysis

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated (opt-in funded reruns)

## How these tasks map to Hee-Lee Oss

Each task below becomes a Hee-Lee Oss **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- `id` — stable slug ID from the tables (e.g. `proteomics-reanalysis-allowlist-002`).
- `title` — the table's Title.
- `project` — `proteomics-reanalysis`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (per table).
- `lane` — `donated` for all current tasks; the **funded** rerun option adds `fundedBudgetUsd`
  (required by the schema's `if lane=funded then fundedBudgetUsd`).
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer","proteomics","open-science","reproducibility"]`.
- `riskTier` — `low | medium | high`. License/PII/re-id and methods-accuracy tasks are `medium`;
  any patient-facing task is `high`.
- `urgent` — boolean; `false` for all current tasks.
- `deliverable` — `pr | dataset | document | translation`. We deliver **`pr`** for code/pipelines and
  **`document`** for reports/datasheets/specs. We do **not** emit `dataset` (raw mass-spec data is
  never re-hosted; small aggregate tables ride along with the `document`/`pr` deliverable).
- `tokenEstimate` — `small | medium | large` (Size column).
- `status` — `open | in-progress | review | delivered | done`; all start `open`.
- `context`, `objective`, `acceptanceCriteria[]`, `resources[]`, `output` — per task.
- `requestor` — **TO BE SECURED** until a named dataset author / curator / steward confirms.
- `verifiedNeed` — **`false`** until a named beneficiary confirms they want a specific reanalysis
  (general need is real; per-task delivery need is unproven).
- `outputLicense` — **`MIT`** for code/pipelines; **`CC-BY-4.0`** for documentation, reports,
  datasheets, and small aggregate result tables. Non-commercial source data (COSMIC/OncoKB) is never
  relicensed or embedded.

---

## Milestone M0 — Foundation & compliance spine (cold-start)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| proteomics-reanalysis-reviewer-001 | Name/secure the License+Compliance reviewer (blocking, non-skippable gate role) | research | small | low | document | — | Maintainer |
| proteomics-reanalysis-allowlist-002 | Dataset allow-list schema + license/PII/re-identification gate (blocking gate) | design-spec | small | medium | document | — | License+Compliance |
| proteomics-reanalysis-repro-standard-003 | Reproducibility standard (immutable container digests, version pinning, run-manifest schema, numeric-tolerance policy, countable-assertion unit) | design-spec | medium | low | document | — | Methods |
| proteomics-reanalysis-skeleton-004 | Containerized pipeline skeleton + CI smoke test (Nextflow/nf-core-style) | code | medium | low | pr | repro-standard-003 | Methods |
| proteomics-reanalysis-provenance-005 | Provenance-completeness linter + run-manifest validator (CI gate) | code | medium | medium | pr | repro-standard-003 | Methods |
| proteomics-reanalysis-reid-policy-006 | Proteogenomic re-identification policy (variant-peptide default-deny) | design-spec | small | medium | document | allowlist-002 | License+Compliance |
| proteomics-reanalysis-outreach-007 | Partner/steward outreach + candidate dataset shortlist + requestor contact | research | small | low | document | — | Maintainer |

**Acceptance criteria — key tasks**

- **reviewer-001 (name the License+Compliance reviewer — blocking role)**
  - [ ] A named, qualified person (proteomics + data-license/privacy competence) accepts the
        License+Compliance reviewer seat **before** any dataset is downloaded or processed.
  - [ ] The role's veto authority over `allowlist.yml` entries and the controlled-access refusal are
        documented.
  - [ ] Empty-seat fallback recorded: no dataset advances past `pending`, M0 cannot exit, maintainer
        escalates to Hee-Lee Oss governance to source a reviewer.

- **allowlist-002 (allow-list schema + license/PII/re-id gate)**
  - [ ] Schema captures: accession (PXD/…), repository, declared license + URL + snapshot,
        `accessClass: open | controlled | unknown`, instrument/acquisition, `piiDetermination`,
        `reidDetermination`, and `status: approved | rejected | pending`.
  - [ ] **Objective gate rule:** PASS only if `accessClass: open` is established from a cited source
        statement AND license verified for the specific accession AND PII/re-id determinations made;
        `controlled` or `unknown` = REJECT (no default-allow). dbGaP/EGA/biobank/identifiable = hard
        REJECT.
  - [ ] COSMIC/OncoKB flagged non-commercial: annotation-only-under-terms, **never embedded** in
        outputs.
  - [ ] License snapshot recorded (committed copy + SHA-256 + archived URL).
  - [ ] Produces a committed, reviewable PASS/REJECT artifact per dataset.

- **repro-standard-003 (reproducibility standard)**
  - [ ] Mandates **immutable container digests** (no floating tags); pinned tool + reference-DB
        versions with checksums.
  - [ ] Defines the **run-manifest** schema (dataset accession + input checksums, container digests,
        tool versions, all params, DB versions/checksums, seeds, operator, environment, duration).
  - [ ] Fixes the **countable "assertion" unit** the 100%-provenance CI gate measures.
  - [ ] States the **numeric-tolerance policy** (exact where deterministic; declared, justified
        per-stage band otherwise) to be published with each reanalysis.
  - [ ] Output documentation licensed CC-BY-4.0.

- **provenance-005 (provenance linter + manifest validator)**
  - [ ] CI fails the build if any published assertion lacks a resolvable run-manifest entry (over the
        countable unit fixed in repro-standard-003).
  - [ ] Validates run-manifest schema; rejects floating container tags and missing checksums/versions.
  - [ ] Ships golden fixtures using **synthetic/tiny public spectra only** (no patient-derived raw
        data); `pnpm build && pnpm test && pnpm lint` green; DCO signed-off; code MIT.

**M0 Definition of Done:** License+Compliance reviewer named (blocking role filled before any data
work); allow-list schema + license/PII/re-id gate + re-identification policy published; reproducibility
standard published with the run-manifest schema, numeric-tolerance policy, and countable-assertion
unit fixed; containerized skeleton runs a CI smoke test; provenance linter + manifest validator green
with synthetic fixtures; ≥3 candidate datasets analyzed and ≥1 `approved`; ≥1 partner-outreach thread
opened and ≥1 candidate requestor contacted; workflow/runtime + output-license decisions ratified.

---

## Milestone M1 — First reproducible reanalysis (proof of pipeline)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| proteomics-reanalysis-gate-pilot-008 | Triage + gate one pilot open PRIDE cancer DDA dataset | research | small | medium | document | allowlist-002, reid-policy-006, reviewer-001 | License+Compliance |
| proteomics-reanalysis-pilot-009 | End-to-end DDA reanalysis of the pilot dataset (containerized) | code | large | medium | pr | skeleton-004, provenance-005, gate-pilot-008 | Methods, License+Compliance |
| proteomics-reanalysis-fdr-010 | FDR/statistics validation harness | code | medium | medium | pr | skeleton-004, repro-standard-003 | Methods |
| proteomics-reanalysis-report-011 | Reproducibility report + methods doc (concordance/divergence vs. original) | writing | medium | medium | document | pilot-009, fdr-010 | Methods |
| proteomics-reanalysis-reexec-012 | Independent re-execution of pilot headline results (different operator/machine) | research | medium | medium | document | pilot-009, report-011 | Reproducibility verifier |

**Acceptance criteria — key tasks**

- **gate-pilot-008 (gate the pilot dataset)**
  - [ ] Pilot is a **public** PRIDE/ProteomeXchange cancer DDA dataset, sized to fit donated-session
        budgets.
  - [ ] Passed allow-list gate with committed PASS artifact: `accessClass: open` (cited),
        license verified + snapshotted, PII + re-id determinations recorded (variant identity
        default-denied).
  - [ ] Confirmed **not** controlled-access and **not** identifiable; COSMIC/OncoKB not embedded.

- **pilot-009 (end-to-end DDA reanalysis)**
  - [ ] Runs end-to-end in **pinned containers (immutable digests)** under the workflow manager:
        spectrum processing → search → FDR → quantification → stats.
  - [ ] Produces a complete **run manifest** (dataset accession + checksums, digests, tool versions,
        all params, DB versions/checksums, seeds, operator/env).
  - [ ] 100% of published assertions carry provenance and pass the CI provenance gate.
  - [ ] Only small **aggregate** result tables are published (CC-BY-4.0); raw data referenced by
        accession, never re-hosted; pipeline code MIT.
  - [ ] `pnpm build && pnpm test && pnpm lint` green; DCO signed-off; license + methods review passed.

- **report-011 (reproducibility report)**
  - [ ] Documents the full method, every parameter, and the numeric-tolerance band used.
  - [ ] Publishes an **honest concordance/divergence statement** vs. the original publication (if one
        exists), explaining any differences rather than hiding them.
  - [ ] States clearly that the output is a **research artifact, not medical advice**.

- **reexec-012 (independent re-execution)**
  - [ ] A **different operator on a different machine** re-runs the pipeline from the manifest and
        reproduces the headline results within the declared tolerance.
  - [ ] Any non-reproducible result is flagged with root-cause notes (not silently dropped).
  - [ ] Re-execution evidence artifact committed (`outcomes/<accession>-reexec.json`).

**M1 Definition of Done:** one open PRIDE cancer DDA dataset reanalyzed end-to-end in pinned
containers with a complete manifest; 100% provenance passing CI; FDR validated; reproducibility report
with honest concordance/divergence; headline results **independently re-executed** within tolerance;
license + methods review passed; ≥1 candidate steward/requestor in conversation.

---

## Milestone M2 — Verification, DIA, and benchmarking (robustness)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| proteomics-reanalysis-dia-013 | DIA pipeline (DIA-NN) containerized + validated | code | large | medium | pr | pilot-009, fdr-010 | Methods |
| proteomics-reanalysis-audit-014 | Independent reproducibility audit of M1 (3rd machine/operator) | maintenance | small | medium | document | reexec-012 | Reproducibility verifier |
| proteomics-reanalysis-benchmark-015 | Concordance benchmark vs. published results (≥2 datasets) | research | medium | medium | document | pilot-009, dia-013 | Methods |
| proteomics-reanalysis-datasheet-016 | Per-reanalysis datasheets + reuse/citation tracking setup | writing | small | low | document | report-011 | Maintainer |

**Acceptance criteria — key tasks**

- **dia-013 (DIA pipeline)**
  - [ ] Containerized DIA reanalysis (immutable digests) producing FDR-controlled, provenance-tracked
        quantities on an approved open DIA dataset.
  - [ ] Validated by the FDR harness; run manifest complete; numeric tolerance declared.
  - [ ] Code MIT; CI green; DCO signed-off.

- **benchmark-015 (concordance benchmark)**
  - [ ] For ≥2 datasets, quantifies agreement between the reanalysis and the original published
        results (overlap of identifications/quantities; documented metric).
  - [ ] Divergences explained (parameter, version, or method differences), not hidden.
  - [ ] Output labeled a research artifact; no clinical claims.

**M2 Definition of Done:** DIA pipeline validated; an independent third re-execution of M1 passes;
concordance benchmark published for ≥2 datasets with honest divergence accounting; per-reanalysis
datasheets published; reuse/citation tracking in place.

---

## Milestone M3 — Scale, catalog & partner adoption (shipped)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| proteomics-reanalysis-scale-017 | Reanalyze datasets #3–#5 (DDA/DIA) with manifests + datasheets | data | large | medium | document | dia-013, datasheet-016 | Methods, License+Compliance |
| proteomics-reanalysis-steward-018 | Secure first confirmed adopting/citing/requesting partner | research | small | low | document | outreach-007 | Steward |
| proteomics-reanalysis-reuse-019 | Track and verify external reuse/citation events | research | small | low | document | datasheet-016, steward-018 | Steward |
| proteomics-reanalysis-drift-020 | Version-drift detection + refresh process (containers/tools/DBs) | maintenance | small | low | pr | provenance-005 | Maintainer |

**Acceptance criteria — key tasks**

- **scale-017 (datasets #3–#5)**
  - [ ] Each dataset passed the allow-list gate with a committed PASS artifact before processing.
  - [ ] Each reanalyzed end-to-end in pinned containers with a complete manifest and 100% provenance.
  - [ ] Each ships a datasheet + honest concordance/divergence statement; raw data never re-hosted.

- **steward-018 (first confirmed partner)**
  - [ ] A named dataset author / curator / reproducibility initiative / community confirms they will
        adopt, host, cite, or has formally requested a reanalysis.
  - [ ] Tasks for that beneficiary updated to `verifiedNeed: true` with `requestor` set.

- **reuse-019 (reuse tracking)**
  - [ ] ≥1 verifiable external reuse/citation event recorded with externally verifiable evidence (no
        self-reported reuse).

**M3 Definition of Done:** ≥5 open datasets reanalyzed cumulatively (manifests + datasheets); ≥1
confirmed partner that has adopted/hosted/cited/requested a reanalysis (Definition of Shipped met);
≥1 verifiable external reuse event; version-drift detection + refresh process live.

---

## Milestone M4 — (Opt-in) Patient/advocate education layer (HIGH risk; gated)

> **Strictly optional.** The core project ships at M3 without this. Every task here is `riskTier: high`,
> education only, carries a "not medical advice" notice, and is **blocked from release until a
> credentialed oncologist AND a patient advocate both sign off.**

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| proteomics-reanalysis-edu-021 | Plain-language explainer: what a reproducible reanalysis does/does not show (education only) | writing | medium | high | document | report-011, steward-018 | Oncologist + Patient advocate (blocking) |

**Acceptance criteria — edu-021 (patient/advocate explainer)**
- [ ] Education-only content; **no** diagnostic/prognostic/treatment guidance and **no**
      patient-specific interpretation.
- [ ] Prominent **"not medical advice"** notice; every claim sourced/provenanced.
- [ ] **Blocking dual sign-off:** a credentialed oncologist AND a patient advocate both approve before
      any release; sign-off evidence recorded. No release without both.
- [ ] Output licensed CC-BY-4.0; `verifiedNeed: true` only once an advocate/educator requests it.

**M4 Definition of Done (if pursued):** explainer drafted, fully sourced, "not medical advice" framed,
and released **only** after blocking oncologist + advocate sign-off — or held with the blocker
surfaced. Project does not depend on M4 to be "shipped."

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| proteomics-reanalysis-tmt-022 | Labeled-quant (TMT/iTRAQ) pipeline support | code | large | medium | pr | Extends quant coverage; pinned containers |
| proteomics-reanalysis-ptm-023 | PTM-localization reanalysis module | code | medium | medium | pr | Phospho/other PTM; methods review heavy |
| proteomics-reanalysis-browser-024 | Static results browser over manifests + aggregate tables | code | medium | low | pr | No accounts, no PII, no hosted compute |
| proteomics-reanalysis-funded-025 | Funded rerun of a compute-heavy dataset (metered, budget-capped) | code | large | medium | pr | **Funded lane** — requires `fundedBudgetUsd`; via `packages/runner` only |
| proteomics-reanalysis-i18n-026 | Translate a delivered datasheet/report (domain reviewer) | translation | small | medium | translation | Widens reuse; needs language + domain reviewer |

---

## Example task JSON

```json
{
  "id": "proteomics-reanalysis-allowlist-002",
  "title": "Dataset allow-list schema + license/PII/re-identification gate",
  "project": "proteomics-reanalysis",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer", "proteomics", "open-science", "reproducibility", "data-licensing"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "Reproducible reanalysis of open cancer proteomics (PRIDE/ProteomeXchange) is only permissible on open-access, de-identified data. Controlled-access repositories (dbGaP, EGA, individual-level biobanks) and any identifiable patient data are strictly out of scope and require authorized access + IRB this project does not have. Before any dataset is downloaded, the project needs a machine-readable allow-list and a binding license/PII/re-identification gate so no out-of-scope dataset can ever be processed. Mass-spec data also carries a proteogenomic re-identification risk (variant peptides can leak germline-identifying information), which the gate must address.",
  "objective": "Define the dataset allow-list schema and the blocking license + PII + re-identification gate that every candidate dataset must pass (with a committed PASS/REJECT artifact) before any download or processing.",
  "acceptanceCriteria": [
    "Allow-list schema captures: accession (PXD/...), repository, declared license + URL + committed snapshot (SHA-256 + archived URL), accessClass (open | controlled | unknown), instrument/acquisition (DDA/DIA/labeled), piiDetermination, reidDetermination, and status (approved | rejected | pending).",
    "Objective gate rule: PASS only if accessClass=open is established from a cited source statement AND the license is verified for the specific accession AND PII and re-identification determinations are recorded; controlled or unknown access = REJECT (no default-allow).",
    "dbGaP, EGA, individual-level biobanks, and any identifiable patient data are hard-REJECT and flagged per Hee-Lee Oss refusal guardrails.",
    "COSMIC and OncoKB are flagged non-commercial: annotation-only-under-terms and never embedded in or redistributed with outputs.",
    "Re-identification rule: variant/proteogenomic identity output is default-denied unless the License+Compliance reviewer clears the specific dataset and use.",
    "Produces a committed, reviewable PASS/REJECT artifact per dataset recording which checks ran and what fired; output documentation licensed CC-BY-4.0.",
    "pnpm build && pnpm test && pnpm lint pass for any committed schema/validator tooling; commit is DCO signed-off."
  ],
  "resources": [
    "C:\\code\\hee-lee-oss\\planning\\projects\\proteomics-reanalysis\\PLAN.md",
    "C:\\code\\hee-lee-oss\\docs\\good-deed-definition.md",
    "C:\\code\\hee-lee-oss\\planning\\ROADMAP.md",
    "PRIDE Archive / ProteomeXchange dataset license terms (per accession)",
    "UniProt CC-BY-4.0; COSMIC and OncoKB non-commercial license terms"
  ],
  "output": "A documented dataset allow-list schema plus the blocking license/PII/re-identification gate checklist, committed to the project repo, that all dataset-intake tasks must use before any download.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```

---

## Example funded-lane task JSON (schema requires `fundedBudgetUsd`)

```json
{
  "id": "proteomics-reanalysis-funded-025",
  "title": "Funded rerun of a compute-heavy open PRIDE dataset (metered, budget-capped)",
  "project": "proteomics-reanalysis",
  "type": "code",
  "lane": "funded",
  "priority": "low",
  "domain": ["cancer", "proteomics", "reproducibility"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "pr",
  "tokenEstimate": "large",
  "status": "open",
  "context": "Some open PRIDE cancer datasets are too large to reanalyze within a single donated session. For these, a metered funded rerun via packages/runner can complete the reanalysis under a hard per-task budget cap, with a public cost note. Only open-access, de-identified data; same compliance gate as the donated lane.",
  "objective": "Reproducibly reanalyze one gate-approved, compute-heavy open PRIDE cancer dataset via the funded runner within a hard budget cap, producing a complete run manifest and aggregate result tables.",
  "acceptanceCriteria": [
    "Dataset passed the allow-list gate (open-access, de-identified, license verified, re-id determined) before any processing.",
    "Run executes only via packages/runner with a hard per-task budget cap and never exceeds escrow; a public cost note is recorded.",
    "Pinned containers (immutable digests); complete run manifest; 100% provenance passing CI; FDR validated.",
    "Headline results independently re-executed within the declared tolerance; concordance/divergence reported honestly.",
    "Pipeline code MIT; aggregate tables CC-BY-4.0; raw data never re-hosted; pnpm build && pnpm test && pnpm lint green; DCO signed-off."
  ],
  "resources": [
    "C:\\code\\hee-lee-oss\\planning\\projects\\proteomics-reanalysis\\PLAN.md",
    "C:\\code\\hee-lee-oss\\packages\\runner"
  ],
  "output": "A reproducible reanalysis of one compute-heavy open PRIDE cancer dataset, with run manifest, aggregate result tables, and a public cost note, delivered as a PR.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "MIT",
  "fundedBudgetUsd": 200
}
```

---

## Task rollup

- **21 scheduled tasks** across M0–M4 (M0: 7 · M1: 5 · M2: 4 · M3: 4 · M4: 1) + **5 backlog** = **26 total**.
- Risk: most are `medium` (license/PII/re-id + methods accuracy); M4 patient-facing is `high`
  (blocking oncologist + advocate sign-off); pure-tooling/skeleton tasks are `low`.
- Lane: all `donated` except the opt-in funded rerun example (`funded`, carries `fundedBudgetUsd`).
- All start `verifiedNeed: false`, `requestor: TO BE SECURED`; flip to `true` only when a named
  beneficiary confirms a specific reanalysis.
- Hard gates before scale: License+Compliance reviewer named (M0), allow-list/license/PII/re-id gate
  published (M0), independent re-execution proven (M1), partner secured (M3).

---

## Generated task index

> Auto-generated by Hee-Lee Oss task decomposition on 2026-06-29. All 26 tasks validated against the
> Hee-Lee Oss Task JSON schema (draft-07, additionalProperties:false). Seed file (allowlist-002) retained
> as-is. Fan-out: none (no explicitly enumerated fan-out dimensions in TASKS.md).

| File | ID | Title | Type | Lane | Priority | Risk | Deliverable |
| --- | --- | --- | --- | --- | --- | --- | --- |
| tasks/proteomics-reanalysis-reviewer-001.json | proteomics-reanalysis-reviewer-001 | Name/secure the License+Compliance reviewer | research | donated | high | low | document |
| tasks/proteomics-reanalysis-allowlist-002.json | proteomics-reanalysis-allowlist-002 | Dataset allow-list schema + license/PII/re-identification gate | design-spec | donated | high | medium | document |
| tasks/proteomics-reanalysis-repro-standard-003.json | proteomics-reanalysis-repro-standard-003 | Reproducibility standard | design-spec | donated | high | low | document |
| tasks/proteomics-reanalysis-skeleton-004.json | proteomics-reanalysis-skeleton-004 | Containerized pipeline skeleton + CI smoke test | code | donated | high | low | pr |
| tasks/proteomics-reanalysis-provenance-005.json | proteomics-reanalysis-provenance-005 | Provenance-completeness linter + run-manifest validator | code | donated | high | medium | pr |
| tasks/proteomics-reanalysis-reid-policy-006.json | proteomics-reanalysis-reid-policy-006 | Proteogenomic re-identification policy | design-spec | donated | high | medium | document |
| tasks/proteomics-reanalysis-outreach-007.json | proteomics-reanalysis-outreach-007 | Partner/steward outreach + candidate dataset shortlist | research | donated | medium | low | document |
| tasks/proteomics-reanalysis-gate-pilot-008.json | proteomics-reanalysis-gate-pilot-008 | Triage + gate one pilot open PRIDE cancer DDA dataset | research | donated | high | medium | document |
| tasks/proteomics-reanalysis-pilot-009.json | proteomics-reanalysis-pilot-009 | End-to-end DDA reanalysis of the pilot dataset | code | donated | high | medium | pr |
| tasks/proteomics-reanalysis-fdr-010.json | proteomics-reanalysis-fdr-010 | FDR/statistics validation harness | code | donated | high | medium | pr |
| tasks/proteomics-reanalysis-report-011.json | proteomics-reanalysis-report-011 | Reproducibility report + methods doc | writing | donated | high | medium | document |
| tasks/proteomics-reanalysis-reexec-012.json | proteomics-reanalysis-reexec-012 | Independent re-execution of pilot headline results | research | donated | high | medium | document |
| tasks/proteomics-reanalysis-dia-013.json | proteomics-reanalysis-dia-013 | DIA pipeline (DIA-NN) containerized + validated | code | donated | medium | medium | pr |
| tasks/proteomics-reanalysis-audit-014.json | proteomics-reanalysis-audit-014 | Independent reproducibility audit of M1 | maintenance | donated | medium | medium | document |
| tasks/proteomics-reanalysis-benchmark-015.json | proteomics-reanalysis-benchmark-015 | Concordance benchmark vs. published results | research | donated | medium | medium | document |
| tasks/proteomics-reanalysis-datasheet-016.json | proteomics-reanalysis-datasheet-016 | Per-reanalysis datasheets + reuse/citation tracking setup | writing | donated | medium | low | document |
| tasks/proteomics-reanalysis-scale-017.json | proteomics-reanalysis-scale-017 | Reanalyze datasets #3-#5 (DDA/DIA) | data | donated | medium | medium | document |
| tasks/proteomics-reanalysis-steward-018.json | proteomics-reanalysis-steward-018 | Secure first confirmed adopting/citing/requesting partner | research | donated | medium | low | document |
| tasks/proteomics-reanalysis-reuse-019.json | proteomics-reanalysis-reuse-019 | Track and verify external reuse/citation events | research | donated | medium | low | document |
| tasks/proteomics-reanalysis-drift-020.json | proteomics-reanalysis-drift-020 | Version-drift detection + refresh process | maintenance | donated | medium | low | pr |
| tasks/proteomics-reanalysis-edu-021.json | proteomics-reanalysis-edu-021 | Plain-language explainer (education only, HIGH risk) | writing | donated | low | high | document |
| tasks/proteomics-reanalysis-tmt-022.json | proteomics-reanalysis-tmt-022 | Labeled-quant (TMT/iTRAQ) pipeline support | code | donated | low | medium | pr |
| tasks/proteomics-reanalysis-ptm-023.json | proteomics-reanalysis-ptm-023 | PTM-localization reanalysis module | code | donated | low | medium | pr |
| tasks/proteomics-reanalysis-browser-024.json | proteomics-reanalysis-browser-024 | Static results browser over manifests + aggregate tables | code | donated | low | low | pr |
| tasks/proteomics-reanalysis-funded-025.json | proteomics-reanalysis-funded-025 | Funded rerun of a compute-heavy dataset | code | funded | low | medium | pr |
| tasks/proteomics-reanalysis-i18n-026.json | proteomics-reanalysis-i18n-026 | Translate a delivered datasheet/report | writing | donated | low | medium | translation |

**Totals:** 26 task files · 25 generated (excl. seed allowlist-002) · validation PASS · no fan-out.
