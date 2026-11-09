---
name: handwriting-page-conversion
description: Convert a photographed handwritten document (assignment, notebook pages) into a clean, complete document written in ONE chosen reference hand — matching the handwriting style, stroke thickness, ink colour and paper tone of a single approved reference page, page by page, with verified text accuracy. Use when asked to rewrite a document "in this handwriting", clone a hand across many pages, or turn photos of handwritten pages into a uniform printable PDF.
---

# Handwriting Page Conversion

Rewrite every page of a handwritten document in one target hand. The target hand is defined by ONE approved reference page (the "standard page"). Every other page must match it on four independent axes: **text, hand identity, stroke thickness/size, and colour (ink + paper)**. A page is not done until all four axes pass.

You need: an image generator that accepts a reference image, an image viewer that can zoom, and a scripting runtime with an image library (e.g. Python + PIL/numpy) for measurement, compositing and grading.

## Core rules (learned the hard way — do not skip)

1. **Never feed the source page photo to the generator as a style/layout reference.** It will photocopy the source — the original hand comes back unchanged and it *looks* converted. Generate each page fresh from: the standard-page image + an exact transcription of the source page's text. Nothing else.
2. **Preserve the writer's own spellings and errors.** Transcribe exactly what the source says, including misspellings ("equillibrium", "singal", "coesist"). The source page is the authority on content, even over reference books and your own knowledge. Verify numbers, table values and formulas digit by digit against the source at zoom.
3. **Verify the four axes separately.** A page can pass text and fail hand (too neat, round, bold, font-like — the generator's default). Text-only verification trivially "passes" on a photocopied page, which is how bad versions ship.
4. **Work page by page.** Build one page, present it, get the user's verdict (or their "build all, I'll judge later") before assembling anything. Fix only the pages the user flags — never silently swap extra pages.
5. **Don't redraw graphs, diagrams or tables if you can avoid it.** Redraws break geometry (curves that don't meet, duplicated axis ticks, wrong endpoints). Carry printed/diagram pages over as-is. For a small label flaw on an otherwise correct drawing, patch pixels: erase with a donor patch of verified-blank paper from the same page, redraw the nicked segment in the sampled ink colour. Locate ink by dark-pixel row/column profiles — never eyeball coordinates off a resized preview.

## Workflow

### 1. Lock the standard
- Pick the best page (or generate one demo page and get it approved). This page fixes: letterforms, median line height (measure it, e.g. ~40 px at working resolution), stroke weight, ink colour (sample it, e.g. dark blue-black RGB ≈ [58,58,65]) and paper tone (e.g. RGB ≈ [208,206,209]).
- Every later page is judged against this page at **matched scale** — thickness comparisons at different zoom levels lie.

### 2. Transcribe
- Read each source page at full size, then zoom-read every suspect word. Write the exact transcription, preserving spellings, punctuation, underlines, boxes, headings and blank regions (if the original leaves half the page blank, so does the rewrite).
- List the page's "trap words" (long/unusual words, values, formulas) explicitly — these get letter-spelled guardrails in the generation prompt and a personal zoom check afterwards.

### 3. Generate fresh
- Prompt: the standard-page image as the ONLY image reference + the exact transcription + a thin-first style spec: *"fine ballpoint, light pressure, thin irregular strokes, dark blue-black ink, never thick / bold / heavy / neat"*. Text-only prompts drift to the model's neat default hand — the reference image must be in the call.
- If your generator supports resuming/continuing from a previous generation's state, resuming from the standard page's own generation and replacing only the text gives the closest hand match. Note: such resume handles can expire (~1 hour) — then fall back to fresh generation from the reference image; never chain resume-from-resume.
- First attempts often slip single words. Reject and retry with the word spelled out letter by letter in the prompt.

### 4. Measure and fix size/thickness (the composite pass)
Generations usually come back oversized and too heavy. Fix as pixels, not by regenerating blindly:
- Detect the page border (e.g. a coloured margin frame: green = G exceeds max(R,B) by >20; find border lines from projection profiles, outermost strong line per side; allow manual override for faint borders).
- Clear the interior with **per-row paper fill sampled from the page itself** (median paper colour per row).
- Transfer **ink only**: build an alpha mask from darkness and thin the strokes with a darkness^1.35 curve. Never paste a rectangle of paper+ink — a pasted paper block leaves ghost outlines of the old writing and a visible paper seam that prints as stray ink.
- Scale the ink block so its median line height lands slightly UNDER the standard's (target ~35 px against a 40 px standard — the eye reads matched-scale pages as oversized).
- Restore the original border frame, re-check border integrity, and print the final median line height as proof.
- **Do not composite table pages or ruled structures** — long rules corrupt border detection and the transfer drags lines around. Ship clean table generations raw.

### 5. Grade the colour
Measure paper and ink on the finished page (exclude border pixels from the ink sample) and remap per channel with an affine transform anchored on the standard page: this page's paper → standard's paper, this page's ink → standard's ink. (Mind the sign: ink is darker than paper, so the denominator is negative — clamping it inverts the grade.) Verify by re-measuring: paper and ink should land within a few RGB points of the standard. Grade every generated page; leave carried-over prints and the standard page itself untouched.

### 6. Assemble and disclose
- Assemble the PDF only from approved pages, in order, page count verified.
- Ship a contact sheet of all pages so consistency can be judged at a glance.
- Disclose known flaws plainly (a dropped phrase, an approximate curve). Never present a fix as done until it has passed a visual crop check — a numeric probe is not an eye.

## Failure modes this skill exists to prevent
- Photocopied pages shipped as "converted" (source fed to the generator; text-only QA).
- Neat font-like hand on some pages, target hand on others (no single standard; text-only prompts).
- Pages matching in style but 20–30% larger / thicker than the reference (no matched-scale measurement).
- Bright-white paper and hard-black ink drifting away from the reference photo's soft tones (no grade step).
- Ghost lettering in the background that prints as ink (paper-block compositing instead of ink-only transfer).
- Redrawn graphs with broken geometry (regenerating what should have been carried over or pixel-patched).
- Scope creep: rebuilding pages the user never flagged.
