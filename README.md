# Handwriting Page Conversion — Skill

Convert a photographed handwritten document (assignment, notebook pages) into a clean, complete document written in **one chosen reference hand** — matching the handwriting style, stroke thickness, ink colour and paper tone of a single approved reference page, page by page, with verified text accuracy.

Use it when you want an AI to rewrite a document "in this handwriting", clone one hand across many pages, or turn photos of handwritten pages into a uniform printable PDF.

## Use it with any AI

This is a standard `SKILL.md` skill — it is not tied to any one assistant.

- **Claude / Claude Code:** copy `SKILL.md` into your skills folder (`.claude/skills/handwriting-page-conversion/SKILL.md`), or install from this repo.
- **Any other AI agent that reads skill/instruction files:** give it `SKILL.md` as its instruction file for the task.
- **Chat AI with image input:** paste the contents of `SKILL.md` into the conversation along with your reference page photo and the pages to convert, and ask it to follow the workflow exactly.

## What the skill enforces

- One approved reference page is the standard — every page is judged against it on four separate axes: text, hand identity, stroke thickness/size, and ink + paper colour.
- The source page photo is **never** fed to the generator as a style reference (that just photocopies the original hand).
- The writer's own spellings are preserved exactly — the source page is the authority on content.
- Ink-only transfer, matched-scale measurement, and per-page colour grading, instead of pasted paper blocks and eyeballed matches.
- Page-by-page verdicts — only flagged pages get fixed.

Full workflow, measurement recipes and failure modes: see [SKILL.md](SKILL.md).

---

Made by [@zavine.webs](https://www.instagram.com/zavine.webs)
