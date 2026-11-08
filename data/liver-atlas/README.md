# Human liver sc/sn/spatial catalog: data files

This folder holds the data behind the catalog page `liver-atlas.qmd`, which renders to `docs/liver-atlas.html`. The page reads these files at render time; nothing is fetched from the web.

## Files

**`catalog.csv`** is the catalog itself, one row per study. The R chunks in `liver-atlas.qmd` read it, derive the filter values (modality, context, platform, data status, donor count) and the accession links, and embed the result in the page as JSON. It has 19 columns:

| Column | Content |
|---|---|
| Title, Authors, DOI, URL, Study Identifier | Bibliographic identity. URL is always `https://doi.org/<DOI>` and Study Identifier repeats the DOI. |
| Published Date, Year | First online date and its year, not the print issue date. |
| Journal Or Server, Journal Quartile | Venue and its best SCImago (SJR 2025) quartile. Preprint servers are `Not reported`. |
| Modality | `scRNA-seq`, `snRNA-seq`, `spatial Omics`, joined by ` + `. |
| Biological Context | Tokens from healthy, tumor, cirrhosis, MASLD, ALD, viral, cholestasis, acute liver failure, transplant, fetal, joined by ` + `. |
| Human Liver Individuals Profiled | Number of distinct people whose liver was profiled, then a description. `Ambiguous` when the paper does not settle it. |
| Spatial Omics Platform | Named platforms separated by `; `, or `Not applicable; no spatial omics in qualifying assay`. |
| Public Data Availability | Starts with `Yes;` (open), `Controlled;` (access on application), `No public repository stated` or `Not established in title/abstract`, followed by repository and accessions. |
| Evidence Status, Cohort Platform Evidence Basis | Whether the record was checked against the full text or only the title and abstract. |
| Study Focus, Main Results | One-sentence aim and up to four result bullets, written for the catalog. |
| Summary | The abstract, verbatim from PubMed or the preprint server. |

**`dictionary.csv`** defines each column, its allowed values and the rules behind them. The page shows it under "How each field was defined".

**`verification.csv`** is the audit trail, one or more rows per study: DOI, date checked, the sources consulted (PubMed, PMC, GEO, ENA, EGA, NGDC, and others), what was changed, and any issue left open. Rows for studies dropped during the audit are kept here with the reason. The page does not read this file.

The CSV files are UTF-8 with a byte-order mark and CRLF row endings, so they open cleanly in Excel. Fields with line breaks (Main Results, Summary) are quoted.

## How the data were obtained

**First version (September 2026).** The original 83 records were screened and extracted with Elicit from a literature search, using the eligibility rules in `dictionary.csv`. About a third were checked against the title and abstract only.

**Audit and expansion (2026-11-08).** An external review found wrong fields and missing studies, so the catalog was rebuilt with AI agents (Claude) in four steps:

1. **Audit of the original rows.** Every field of the 83 records was checked against PubMed, the full text where readable, and the data repositories. Preprints were replaced by their published versions, duplicates merged, and records that did not qualify (conference abstracts, dataset entries, re-analyses of public data, non-human or xenograft-only data) dropped. 72 records remain from the first version.
2. **Search for missing studies.** Separate searches of PubMed, bioRxiv, medRxiv and GEO covered MASLD and alcohol-related disease, healthy, fetal and other liver disease, liver tumors, and spatial transcriptomics in any context. Each candidate was checked for eligibility and written as a full record.
3. **Independent verification.** Each new record was re-checked by a second agent that saw the draft but not the reasoning behind it: identity against PubMed, eligibility, modality and platform from the methods, donor counts from the paper and GEO sample records, and every accession opened in its repository to confirm it exists, is human liver from that study, and whether it is open, controlled, raw or processed only. About half the drafts needed a correction; two were rejected. A random sample was then re-checked once more.
4. **Quartiles.** Every journal's quartile was read from its SCImago page.

**Eligibility.** A study is included when it generated new single-cell, single-nucleus or spatial transcriptomic data from human liver tissue (adult, pediatric or fetal; including liver metastases, fine-needle aspirates, perfusates and precision-cut slices). Re-analyses of public data, organoids or cell lines only, blood only, bulk RNA-seq, scATAC-seq alone, and FISH or IHC used only for validation are excluded. When two papers share one dataset, the catalog cites the paper that generated it.

**Conventions worth knowing.**

- Normal tissue next to a tumor, profiled for comparison, is tagged `healthy + tumor`.
- Acute-on-chronic liver failure is tagged `cirrhosis + acute liver failure`.
- A deposit registered but not yet released, or a cited accession that does not resolve, counts as `No public repository stated`, since there is nothing to download or apply for.
- When processed data are open and raw reads are controlled, the study counts as `Yes`, with the controlled part noted.

## Known limits

- 53 records rest on the title and abstract only, because the full text was paywalled or behind a bot check. 26 records have data availability `Not established`.
- Some NGDC (GSA-Human, OMIX) and CNGB accessions could not be opened automatically. Their access level comes from the paper's data statement.
- Donor overlap between studies from the same group is noted in `verification.csv` but not resolved. Do not add donor counts across studies without checking it.
- The search was broad but not exhaustive. A few candidates could not be screened because their full text was inaccessible.

## Updating

Edit `catalog.csv` following `dictionary.csv`, add a row to `verification.csv` for what you checked, then render the page with `quarto render liver-atlas.qmd`. The page is frozen (`execute: freeze: true`), so it only re-reads the CSV when rendered explicitly.
