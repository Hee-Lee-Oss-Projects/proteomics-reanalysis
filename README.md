# proteomics-reanalysis

> Most published cancer-proteomics results are effectively irreproducible by a third party. The raw data is increasingly deposited openly (PRIDE/ProteomeXchange now hold tens of thousands of datasets),   ·  **Risk tier:** med  ·  **Status:** planning

Most published cancer-proteomics results are effectively irreproducible by a third party. The raw data is increasingly deposited openly (PRIDE/ProteomeXchange now hold tens of thousands of datasets), but the *analysis* that turned spectra into protein/peptide quantities and biological conclusions is usually under-specified: exact tool versions, search parameters, FDR settings, contaminant/decoy databases, normalization choices, and statistical models are missing, inconsistent, or buried in a methods paragraph. Re-running a study from its deposited raw files is often impossible, and small parameter differences can change which proteins are called differentially abundant. This is a documented reproducibility crisis in the field.

**Definition of shipped:** | Container/environment fully pinned by immutable digest (no floating tags) | n/a | 100% of pipelines | | Datasets passing the license + PII + re-identification gate before any processing | n/a | 100% (no processing of an un-cleared dataset) | | Reproducibility outcome reported h

This is a **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Get started: https://github.com/Hee-Lee-Oss-Projects/hee-lee-oss-downloads

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo Hee-Lee-Oss-Projects/proteomics-reanalysis --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
