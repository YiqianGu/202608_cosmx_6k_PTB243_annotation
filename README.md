# PTB 24.3 — CosMx 6K spatial web viewer (IN-HOUSE REFERENCE)

Interactive deck.gl viewer. Regenerate with `scripts_inhouse/5_export_web_viewers.R`,
deploy with `scripts_inhouse/deploy_viewer_inhouse.sh`.

**This is the in-house-reference build.** The combined-reference build lives in
`analysis/web_viewers/` and deploys to a different repo; both are kept live so
they can be compared.

## What changed vs the combined-reference viewer

The snRNA reference was rebuilt from the 7 in-house `PTB-` samples only
(31,399 cells), dropping the 4 external Pyle samples, because the CosMx slide
and the pooled reference did not represent the same tissue.

* `FB11` (mesothelial, 24 in-house cells) and `BC7` (33) fell below the
  50-reference-cell floor and were removed. The 1,801 query cells that carried
  `FB11` were reassigned — see `analysis_inhouse/seurat/09_fate_of_dropped_subclusters.csv`.
* `TC9` (recently-activated T) was **exempted** and keeps its 77 Pyle cells
  (112 total), because it is the population of interest downstream.
* The query object is byte-identical to the combined-reference run (same QC,
  same 2000 anchor genes in the same order — asserted at runtime), so **every
  label difference is attributable to the reference alone**.
  Full delta: `analysis_inhouse/seurat/09_*`.

## Annotation layers

| Layer | What it is |
|---|---|
| Major Cell Type | 9 classes + `unassigned` / `QC_removed`, Round-1 transfer |
| Subcluster | 10 labels, Round-2 per-lineage transfer |
| T-cell activation | label transfer, with module-positive T cells relabelled |
| Tissue piece | within-block region (A = small, B = large) |

The **T-cell activation** layer is *not* label transfer. T cells scoring above a
cutoff on a 7-gene immediate-early module (EGR1, EGR2, NR4A1/2/3, IER3, KDM6B)
are relabelled `Early activation T cell` (0 cells); every other cell keeps its
transferred label, so the two layers can be compared directly. EGR3, NFKBID and
VDR were requested but are not on the 6K panel. Cutoff provenance:
`analysis_inhouse/early_activation/00_module_score_histogram.pdf` and
`00_cutoff_rules.csv`.

## Highlighting

Click the **◎** on any legend row to draw an enlarged ring around those cells.
It is an overlay, not a change to the cell geometry, so it works in **both**
point and polygon mode, and several labels can be ringed at once. The
**Highlight size** slider (1–10x) controls prominence.

The **cell-ID box** accepts a pasted list (comma / space / newline separated) and
reports how many matched — this consumes the ID lists the analysis scripts emit,
e.g. `analysis_inhouse/early_activation/09_EAT_cell_ids.txt`. Label highlights
are carried in the URL hash (`#hl=...&hls=...`), so a highlighted view is
shareable; pasted ID sets are not, as they would blow the URL length.

## Two things to know before interpreting

**1. One tissue block, two pieces.** The slide holds a single block placed as two
physically separate pieces (large = FOV 1–281, small = FOV 282–310, ~2.98 mm
apart). They are **not two specimens**, so `Tissue piece` is a within-block
regional split, not a group comparison.

**2. `PR0` / Proliferating is unreliable.** It failed marker validation on this
slide: only 27.7% detect any cycling transcript and 43.9% are CD3+, and its top
DEGs are `GZMA`/`CD2` rather than cycling genes. It absorbs T cells rather than
marking cycling ones. **Do not read PR0 as a proliferation measure**, and treat
the T-cell fraction as an underestimate.

Cells failing QC (`nCount < 100` or negprobe fraction > 0.02) appear as
`QC_removed` so the tissue outline stays complete.

An orange **⚠** on a legend label means this slide's own DEGs do not support the
reference call. Hover for the evidence. Full table:
`analysis_inhouse/palette/subcluster_annotations.csv`.

## Test locally

```bash
cd analysis_inhouse/web_viewers && python3 -m http.server 8000
# open http://localhost:8000
```

Opening `index.html` via `file://` will not work — the viewer fetches
`data/*.json` over HTTP.

## Deploying

`scripts_inhouse/deploy_viewer_inhouse.sh` pushes this folder to GitHub Pages.
First-time setup (creating the repo, adding the remote, enabling Pages) is
documented in that script's header and is **not** automated — this is
patient-derived data, so making it public is a deliberate decision.

