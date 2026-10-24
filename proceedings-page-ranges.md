---
name: proceedings-page-ranges
description: Find and verify the physical PDF page range for every research paper in conference proceeding PDFs under pdfs/.
---

# Proceedings paper page ranges

Use this skill when asked to identify, extract, or verify the physical PDF page numbers occupied by research papers in one or more conference proceedings PDFs, especially files in `pdfs/`. This task produces a page-range inventory; it does not require writing paper summaries.

## Deliverable

For every input PDF, create or update a Markdown report beside the PDF under `pdfs/analysis/<pdf-basename>/page-ranges.md` (or the corresponding analysis folder if one already exists). Preserve existing content and paper analyses. Include:

- source PDF path and PDF page count;
- one row per research paper, in proceedings order, with title, inclusive physical PDF page range (`first–last`), page count, printed page range if available, and status;
- unresolved entries and a concise reason, rather than silently omitting or guessing them;
- a validation summary with counts of confirmed, uncertain, and unresolved papers, and any gaps, overlaps, or non-paper sections.

Physical PDF page numbers are 1-based positions in the PDF file, including cover and front matter. They are not the page numbers printed on the page. Always label them explicitly as `physical PDF pages`.

## Procedure

### Pass 1: Inventory and contents extraction

1. Enumerate every `.pdf` in the requested `pdfs/` directory recursively. Process each PDF independently. If the user named specific PDFs, process those. Record the exact file set so none is skipped.
2. Determine each PDF's physical page count. Extract text page by page, keeping explicit physical page markers (for example, `--- PDF page 12 ---`). Use `pdftotext -layout` or a PDF library; render pages when text extraction is absent or suspicious.
3. Locate the table of contents or equivalent authoritative paper list. Extract every research paper title and its printed starting page. Do not count session headings, author/organization lists, committees, sponsors, indexes, or references as papers. Record separately front matter, keynotes, abstracts, appendices, and other sections when they affect boundaries. Include keynotes only when the user or proceedings' contents treats them as paper entries; label them distinctly.
4. Do not confuse an extracted paper's printed page number with its physical PDF page. Establish the printed-to-physical mapping from visible page labels and page content. The offset may change across sections; verify it locally rather than assuming a global offset.

### Pass 2: Establish ranges

5. For each paper, find its first physical page by matching its title and opening content against the contents entry. The title page may have no printed page number. Inspect neighboring pages when necessary.
6. Find its last physical page by locating the next paper's verified first page and inspecting the transition. The range is inclusive. Exclude intervening proceedings material such as a blank separator, session page, or another contribution when present. For the final paper, identify its actual ending before the next non-paper section or end of file.
7. Use the contents' printed end page when available as an independent clue, but verify it against actual PDF pages. Do not infer a physical range solely by subtracting or adding one constant offset.
8. Mark each range `confirmed`, `uncertain`, or `unresolved`. A range is confirmed only when title/start and end/next-boundary have been checked against the PDF. If the scan is unreadable or evidence conflicts, preserve candidate ranges and explain the conflict.

### Pass 3: Independent verification

9. Re-read the contents entries and walk the PDF in physical page order, independently checking every research-paper start. Do not merely reuse the first-pass range list. Confirm the title on the first page, and inspect the last page and the following page for each boundary. If the next page starts another paper, verify the transition; if it does not, check for an appendix, references, or another section belonging to the current paper.
10. Run consistency checks for every PDF:
    - all contents-listed research papers are represented exactly once;
    - no apparent research paper in the PDF is missing from the contents-derived list (investigate unlisted contributions);
    - each range is valid, inclusive, and within the PDF page count;
    - ranges are in proceedings order and do not overlap;
    - every gap between ranges is accounted for as front matter, a separator, a contents-omitted contribution, or another named section;
    - adjacent papers do not share a page unless the proceedings actually place both on that page;
    - range length agrees with printed pagination where that comparison is possible;
    - output totals reconcile with the itemized statuses.
11. For every mismatch, re-inspect the relevant PDF pages (render them if needed), correct the inventory only when evidence supports a correction, and state remaining uncertainty. Do not manufacture completeness: partial results must be labelled partial and list exactly which papers or PDFs remain unresolved.

## Output format

Use this compact table, adapting columns only when useful:

| # | Paper title | Physical PDF pages (inclusive) | Pages | Printed pages | Status / evidence |
|---:|---|---:|---:|---:|---|
| 1 | Example title | 14–25 | 12 | 1–12 | confirmed; title on PDF p. 14, next paper begins p. 26 |

Follow the table with `## Verification` listing page count, contents location, printed-to-physical mapping evidence, papers confirmed/uncertain/unresolved, and the consistency-check results. Include a `## Unresolved items` section whenever anything is uncertain or incomplete. Cite physical page numbers in all verification notes.

## Failure handling

- Continue processing other PDFs if one fails.
- If contents cannot be located confidently, report that PDF as incomplete and identify what is needed (for example, a clearer scan or the physical pages containing its contents). Do not invent entries.
- If OCR is needed, mark affected entries `unresolved (OCR needed)` and keep any reliable partial inventory.
- If page extraction, rendering, or file access fails, record the concrete error in the report and summarize it to the user.
- Never alter source PDFs.
