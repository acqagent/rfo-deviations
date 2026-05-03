# RFO Class Deviation Corpus

A dated, machine-readable archive of every agency-issued FAR class deviation linked from the [acquisition.gov FAR Overhaul deviation guide](https://www.acquisition.gov/far-overhaul/far-part-deviation-guide).

The Federal Acquisition Regulation is undergoing a top-to-bottom rewrite — the *Revolutionary FAR Overhaul (RFO)* — driven by Executive Order 14275 (April 2025) and OMB memo M-25-26. Each civilian executive agency has issued (and continues to issue) class deviations from the existing FAR while the rewrite is staged in. This repo collects those deviation PDFs into versioned, attribution-friendly snapshots so you can analyze them without scraping yourself.

---

## Source code

The PDFs and manifest are produced by **[acqagent/far-collector](https://github.com/acqagent/far-collector)** — see that repo for the scraper, the LLM-based extractor, and the DuckDB schema.

## Releases

Each release is a point-in-time snapshot tagged by the date the collector ran. The zip contains the full PDF set under `pdfs/` plus a `_manifest.csv` index.

| Tag | Scraped | PDFs | Compressed size |
|---|---|---|---|
| [v2026-05-02](../../releases/tag/v2026-05-02) | 2026-05-02 | 1,191 | 395 MB |
| [v2026-04-27](../../releases/tag/v2026-04-27) | 2026-04-27 | 1,133 | 379 MB |

Direct download:
```
gh release download v2026-05-02 -R acqagent/rfo-deviations
```
or via the web UI under the **Releases** tab.

## XLSX summaries (in repo, by snapshot date)

Two derived workbooks ship alongside `manifest.csv` for each snapshot date.

### `far_class_deviations-<date>.xlsx`

Per-agency Part 52 P&C tracker. **34 tabs**: a README tab plus one tab per civilian agency (33 agencies). Each agency tab carries every `52.*` provision and clause (~702 rows) with the agency's class-deviation effective date stamped on rows whose parent FAR Part is covered by an agency memo.

> Schema change in v2026-05-02: prior versions of this filename held a flat per-memo index. Starting 2026-05-03 the file is the per-agency clause-level workbook described above (generated from this corpus by Claude Opus 4.7 max effort). Per-agency tab schema:
>
> | Column | Description |
> |---|---|
> | Type / Number / Part | FAR 52.* identifier and its parent Part |
> | Pre-RFO Title / Date | Original FAR title and effective date |
> | RFO Title | Title under the RFO; `[Reserved]` = removed by Council |
> | `<Agency>` Deviation Effective Date | Earliest agency memo's effective date for the parent Part; per-clause overrides when a memo names a 52.x clause explicitly |
> | Disposition | FAR Council baseline action |
> | Notes | Agency-specific commentary when the memo flags something non-standard |

### `far_provisions_clauses-<date>.xlsx`

Master 52.* provision/clause list for that snapshot date — used as the blank template that the tracker is built on top of.

## Manifest schema (`manifest.csv`)

Tracked in the repo so you can browse the dataset structure without downloading the zip.

| Column | Type | Notes |
|---|---|---|
| `on_disk_filename` | string | `<url_hash>_<original_filename>`, the path inside the zip's `pdfs/` directory |
| `url_hash` | string | First 16 hex chars of `sha256(source_url)`; collision-safe filename prefix |
| `original_filename` | string | The bare filename from the source URL (preserved as published by the agency) |
| `agency` | string | Canonical short label: `DHS`, `GSA`, `NASA`, `Treasury`, … (DoD intentionally excluded — DFARS lives elsewhere) |
| `part_number` | int | FAR Part number this deviation covers; `-1` if multipart and unparsed |
| `is_dod` | int | `1` if the filename contains a DoD/DFARS marker; otherwise `0` |
| `source_url` | string | Direct acquisition.gov URL the PDF was fetched from |
| `pdf_size_bytes` | int | Size of the PDF on disk |

A given PDF can appear in multiple rows when it covers multiple FAR Parts (one row per `(filename, part_number)`). The `url_hash`/`on_disk_filename` is unique per URL.

## Scope and exclusions

- **Civilian agency deviations only.** DoD class deviations are published separately by OUSD(A&S) and follow a different cadence; they're out of scope here.
- **Source links can rot.** Some entries in the deviation guide point at PDFs that 404 from acquisition.gov; those are absent from the zip but still listed in the deviation guide. The collector logs these but does not retry indefinitely.
- **No OCR.** PDFs are stored as-published. Text extraction in downstream tools relies on the embedded text layer.

## License

The PDFs are works of the United States Government and are in the public domain under [17 U.S.C. § 105](https://www.copyright.gov/title17/92chap1.html#105). The manifest, README, and packaging in this repo are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — attribution requested but not required for the underlying federal documents.

## Citation

If you use this corpus in research, please cite as:

> AcqAgent. *RFO Class Deviation Corpus*, version v2026-05-02. https://github.com/acqagent/rfo-deviations
