---
name: guwenzi-remote
description: Use guwenzi-tools as an agent-controlled set of atomic document, image, glyph-search, and glyph-composition tools. Read pages, preserve evidence, output text, and call refinement tools only when the visual model is uncertain.
---

# Guwenzi remote tools: agent decides the next action

## Vertical reading and direct font comparison

Inspect the live schema. For vertical columns, use split_line direction=vertical-ttb without rotating source pixels. layout_analyze writing_mode=vertical-rl orders right columns first and characters top-to-bottom; vertical-lr is left-column-first, horizontal-tb preserves ordinary horizontal text. Geometry is a proposal, not reading confirmation.

Use reading_plan when explicit section/slip/column and unit order is needed. regions carry original-pixel bbox, section_id, role, writing_mode and units; multiple sections require section_order. Separate main text, annotation, photo fragments and print columns; do not globally sort a mixed page by x. alignments retain source_ids/target_ids plus monotone anchors, including empty-side gaps and 1:n compounds. Keep every omission square, omission mark and unassigned fragment; equal counts or equal y coordinates do not prove correspondence.

Do NOT use DINO for font acceptance. glyph_verify items take source_asset_id, final font_asset_id and codepoint (including font-local PUA) and return direct rendered-font differences. Recheck every source occurrence, including Unicode and symbols, against the final combined font. A pass covers only the stated raster tolerance, not identity; needs_review retains a printable candidate and diagnostic evidence, not a task failure or permission to claim completion. Fix selected components or structural layout then recheck; avoid endless retries for scan/style differences. No public/private page upload beyond the user's authorized workflow. If new tools are absent from the live schema, report the deployment gap instead of inventing calls.


This is an atomic-tool contract, not a fixed OCR pipeline. The remote service does not contain a multimodal language model and does not decide which glyph needs research. The calling agent must inspect returned images, write the text it can read, and request more evidence only when necessary.

## Authentication and safety

```sh
python -m pip install -U guwenzi-tools
guwenzi-tools login
guwenzi-tools whoami
guwenzi-tools doctor --quick
```

Self-hosted services use `guwenzi-tools configure --endpoint URL`; never place a shared token in a prompt, source file, HTML, or evidence package. `tools --server` returns the live JSON schemas.

Every output must preserve the distinction between original file/page/image bytes, `asset_id`, `span_id`, temporary candidate IDs such as `c1`, `gwz:` glyph identity, source-font codepoints and aliases, and an author's reading or the agent's interpretation.

Never apply NFC/NFKC, traditional/simplified conversion, silent codepoint substitution, or a guessed reading. Similarity is not a probability and a selected candidate is `model_selected_unverified` until checked.

## Atomic tool map

Choose tools from the current evidence; do not execute every row:

| Need | Tool/command | Result |
|---|---|---|
| Register a whole file | `document_open` / `document open` | Original bytes, hash, page count; no reading |
| Register a page image already available | `asset_import` | Page `asset_id`; enlarged view available on demand |
| See one image more clearly | `asset ASSET_ID --view` | Enlarged derivative, no new detail |
| Ask YOLO where regions are | `layout_analyze` with `analysis_mode=regions` | Text/figure/table/caption proposals and overlay; no OCR |
| Get optional projection lines/slots | `layout_analyze` with `regions_lines` or `split_line` | Suggestions only; use only when AI needs localization |
| Take a precise local crop | `crop ASSET_ID --box L T R B` | Original-pixel crop and provenance |
| Search a doubtful glyph | `glyph_search` / `glyph search` | Candidate images and neutral `cN` IDs |
| Commit a candidate choice | `glyph_select` | Binding and source identity, still unverified |
| Preserve a glyph with no match | `glyph_register` | `gwz:sha256:` identity for the original bytes |
| Find reusable glyphs/parts | `glyph_lookup` | Versioned glyph_ref, real previews, variant pages; mode=parts exposes named parts/templates |
| Reuse or assemble printable glyphs | `glyph_compose` | Supply glyph/ids/structure; server writes KAGE, SVG/PNG and OTF/TTF/cmap/Word |
| Save the agent's reading | `transcribe` | Structured text and glyph references stored with page/block evidence |

## Fast evidence workflow (inspect live schemas first)

Use `guwenzi-tools tools --server` to check capabilities; do not assume a host has the new arguments. On performance-enabled hosts, `page_prepare` accepts `detail=image|regions|lines|slots`. Start with `detail=image` to get the page and its source text without YOLO or character cutting; native text is still source evidence, not a verified reading. In CLI 0.3.6+, `page image DOCUMENT_ID PAGE --output DIR` selects this mode. Older clients can use `call page_prepare --json @request.json`.

Use `layout_analyze` only if region localization helps: `regions` generates only regions, `regions_lines` adds lines, and `regions_slots` explicitly adds every character slot. Do not request slots simply to read a paragraph. A legacy host without these semantics may still do expensive combined preparation; treat that as compatibility fallback, not the default.

For several uncertain forms, collect their original-pixel boxes first, then issue a batch under an existing tool name:

- `crop`: `{"items":[{"asset_id":"asset:...","bbox_px":[10,20,70,90]}, ...]}`.
- `glyph_search`: `{"asset_ids":["asset:...","asset:..."],"top_k":8}`.
- `glyph_compose`: `{"items":[{"glyph":"<glyph_ref>","hint_asset_id":"asset:..."},{"ids":"⿱艹⿰日月","hint_asset_id":"asset:..."}]}`.

Policy-enabled hosts queue accepted work and split large batches internally; do not invent a total page, glyph, iteration or elapsed-time cutoff. A contact sheet may have multiple pages: inspect `contact_sheets` as needed, while full results remain available. Older hosts may still expose batch ceilings in their live schema; split the work without omitting items. CLI 0.3.7+ streams uploads without a client file-size quota and waits until completion or user cancellation by default. An explicit `status --timeout` is the caller's choice, not a server task deadline. A receipt with unknown acceptance must still never be replayed blindly.

CLI 0.3.6+ supports `batch TOOL --json @request.json --output DIR --preview-only`; the generic `call TOOL` works on earlier clients too. Batches retain per-item indices and original assets. Crop/compose report individual invalid-item errors; search validates the query assets together and an invalid input fails that search job. Inspect the labelled contact sheet together, then fetch individual originals only where needed. The sheet is a resized preview, not source pixels; never use its cell coordinates as crop coordinates. Search sheets show up to8 candidates per query; an expanded 32-candidate result may require inspecting the remaining originals before concluding no match.

Read the whole paragraph first, identify uncertainties, obtain evidence in a batch, then review and transcribe the whole region. Reuse an already-verified occurrence or an identical-image result; do not merge different forms just because they share a Unicode label. Do not repeat search or regenerate a KAGE recipe merely because the same form occurs again. An unresolved form still needs explicit uncertainty, not a lower acceptance threshold.

Enlarged views are generated on demand (`asset ASSET_ID --view`); returned original pixels and provenance remain available even if no `view` field is materialized yet. All tools remain evidence primitives: job completion is not a finished OCR transcription.

## Image-page decision pattern

For a scanned or image-only page:

1. Obtain the page image and let the agent inspect it at a readable scale.
2. If region localization is needed, call YOLO layout analysis with `regions`, not all slots. Skip it for an already readable, localized paragraph. Keep figure/table regions as ordered visual objects.
3. Inspect each proposed text region. If the agent can read a region, transcribe the whole region directly; do not download every slot.
4. If one character is uncertain, choose a crop box from the readable region and call `crop`.
5. Call glyph search only for that crop. Compare the actual candidate images, not just scores. A miss does not prove that the form lacks Unicode.
6. Recheck the source sentence, parenthetical reading, author's explicit character name and clear component structure. Use those direct signals with a Unicode/GlyphWiki lookup before declaring the form unencoded.
7. If a candidate is useful, select it as an unverified binding. If no candidate is defensible, register the original crop. For a genuinely unencoded printed form that must be typeable, first use `glyph_lookup` to reuse an existing glyph or choose components, then call `glyph_compose` with `glyph`, `ids`, or `structure`; keep known characters on their standard Unicode codepoints.
8. Submit the complete region text with structured glyph/image references through `transcribe`.

The agent, not the service, decides whether a character is known, uncertain, unencoded, or worth reconstructing. Do not turn a table into hundreds of forced OCR slots.

## Structural composition, not stroke programming

The agent selects structure and variants; the service resolves versioned component programs, computes layout, and writes all KAGE. Never author kage_json, root/table, stroke coordinates or a low-level KAGE tutorial. Check live schemas: if glyph_lookup or glyph/ids/structure is absent, report the deployment gap instead of reverting to raw programs.

Use glyph_lookup(query=character or glyph name) and inspect previews; glyph_compose(glyph=returned glyph_ref) reuses an entire shape. A public gwz code from glyph_select can enter lookup; direct composition bridges a GlyphWiki source only when its snapshot matches. DINO remains the existing server-side image search, not a font acceptance test. Neither models nor the whole database belong in each agent sandbox.

For a new arrangement, use ids="⿱艹⿰日月" or structure={op:"⿰",children:[{glyph:REF},{glyph:REF}],weights:[1,2]}. Prefer mode=parts to obtain an existing glyph's reusable components/template; do not reconstruct ordinary components stroke by stroke. The template's boxes are machine-generated. An unavailable component is an explicit gap, never a blank or guessed familiar substitute. Preserve Unicode, original evidence and printable candidates separately.

gwk: fixes a component program and its dependency closure; gwz: identifies glyph evidence. Neither establishes a reading. Reuse validated outputs. Raw KAGE artifacts are retained for reproducibility, not for the model to edit.

## Printable font delivery

For printable output, check the live compose schema for `unicode_codepoint`. Supply an already-confirmed character explicitly (e.g. `"U+655A"`); omit only when no standard codepoint can be asserted, which yields font-local PUA, not an identity claim. The existing PNG `asset` is just the visual preview. Require `print_font.status=ready`, then fetch/publish its OTF or TTF, mapping, copyable text and optional embedded-font Word sample. For batches use the top-level combined font, not concatenated single-font U+E000 values. Report partial/failed packs and encoding conflicts; do not silently remap or call images an editable font. On old hosts report the missing capability instead of inventing fields or retrying POSTs. Existing native Unicode text does not need composition merely to remain printable.

For faithful source-print reproduction, verify EVERY transcribed position, including ordinary Unicode and punctuation, against the actual final font/document rendering. Font generation `ready` is not visual acceptance. Source/font/hash coverage alone does not compare shapes. DINO cosine (even above 0.95) cannot authorize export; also inspect local missing/extra strokes, curves, joins and counters. Do not dismiss different structures as font style. Preserve source, codepoint, font/render hashes and differences; changed evidence invalidates prior approval. Revise glyph selection, components or layout, then re-render and re-check, without lowering the threshold to force a pass. A blocked candidate remains a printable draft, not a blank and not a completed faithful transcription. Report false rejections and unresolved differences; no promise of exact font recovery from a blurred scan. Use local `guwenzi-glyph verify-font` only if actually installed and listed by `--help`; do not invent a remote tool or claim this local feature has been deployed. This final typesetting check does not require slotting every character for ordinary reading.

## Projection and crop boundaries

Projection measures ink by row or column; it proposes whitespace boundaries and never recognizes a character.

- Use it for deskewed, direction-consistent printed blocks with stable line spacing, and optionally for candidate slots in fixed-pitch CJK text.
- For footnote markers, superscripts, punctuation, annotations or proportional type, use it only to locate a line; merge detached dots/components and check neighboring spacing.
- Do not let projection define single-glyph boxes for rubbings, hand copies, bamboo-slip images, touching forms or damaged/noisy material.
- The calling visual model chooses the final bbox while viewing the complete line/region. `crop` only executes `[left, top, right, bottom)` in original-image pixels and records provenance.

A useful hybrid is: layout region → deskew → row/column projection → slot proposal → component merge and neighbor constraints → visual review → exact crop.

## Native/text PDF decision pattern

For a PDF with a text layer:

- preserve and read the native text as the source layer;
- preserve embedded images, fonts, paths, and page coordinates; `detail=image` gives a lightweight render/text trace, not a complete font/path extraction. When those assets are needed, request the host's native evidence with the same mode/DPI, keeping the source PDF;
- use visual evidence to check encoding gaps or suspicious text;
- if an unencoded path/image character is unknown, crop or retrieve that individual form using the same atomic image tools;
- submit an external correction, reading, or glyph reference separately from the original source text.

Native extraction is not a reason to skip visual evidence, and visual retrieval is not a reason to overwrite the source text.

## `transcribe` semantics

`transcribe` is not an OCR command and does not invoke an AI model. It persists text that the calling agent has already read, with parts such as `text`, `binding_id`, `glyph_code`, or `asset_id`. Use it only after reading the relevant region and checking the evidence. Keep the source layer, the agent's transcription, and scholarly interpretation separate.

## Uncertainty and resumability

- Read the original page/region image before interpreting a candidate.
- A projection boundary is not proof that one crop is one character; correct it with `crop` when needed.
- Preserve failed search evidence when direct source text later resolves a character; this demonstrates that retrieval and identification are different claims.
- Use `--detach` and `status JOB_ID --wait --output DIR` for long jobs.
- A newly started instance may spend extra time preparing model/index caches. Resume the accepted job by ID instead of submitting it again; a fast repeat-cache hit is not the expected latency of every new query.
- Never repeat an uncertain POST blindly. Keep the job receipt and inspect its status first.
- Do not place base64 images or entire catalogues in the model context; fetch only the evidence needed for the next decision.

## Minimum final record

A useful result contains the original source hash, ordered text/figure/table objects, page and block citations, original and derived image hashes, any search/selection evidence, unresolved glyphs, and a clear status for every claim. Completion of a tool job never means that the text or reading is correct.
