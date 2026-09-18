# AWS Certification Notes

Personal study notes for AWS certifications (currently just [`saa-c03/`](saa-c03/), the SAA-C03 Associate exam). Each cert lives in its own root folder, organized by AWS service domain, with numbered domain folders mirroring how AWS documents its own services (compute, networking, storage, databases, ...).

## Content sources

- **Base content**: seeded from the [Stellex Group SAA-C03 Cheat Sheet](https://www.stellexgroup.com/blog/aws-solutions-architect-associate-saa-c03-cheat-sheet).
- **Slide deck**: `AWS Certified Solutions Architect Slides v48.pdf` at the repo root, split and converted page-range-by-page-range into markdown under `converted/` (e.g. `converted/AWS Certified Solutions Architect Slides v48_pages_181-210.md`). When a user request references "the slides" without a page number, grep `converted/` for the relevant heading/keyword to find the right page range — page ranges don't map predictably to topics.
- Files may carry a `## To research` section for exam-scope gaps the slides/cheat sheet never answered; `saa-c03/open-research-items.md` is the aggregated checklist across all such gaps — when a gap gets filled in a file, remove its row there too.

## Editing conventions for topic files

- Organize content by **logical topic section** (e.g. `## Storage Tiers`, `## Access Control`, `## Replication`), not as a flat dump of "stuff from the slides." When pulling new content in from the slide deck, place each bullet under the section it topically belongs to, splitting a single slide page across multiple sections if needed.
- Every file ends with a `## Notes` section (with an HTML placeholder comment `<!-- Your own notes go here. -->` for the user's own additions). When slide-sourced content is added or reorganized, keep a short provenance note here (e.g. "content sourced from slide deck, pages 121-270") so it's traceable later — update the wording as content moves between reorganizations.
- Keep bullets terse but complete: pull out concrete exam-relevant specifics (numbers, thresholds, formulas, named scenarios) rather than leaving a generic one-liner. Prefer several short, specific bullets over one long paragraph.
- Bold the key term at the start of a bullet introducing a named concept/feature (e.g. `- **Weighted** — splits traffic by relative weight...`).
- Cross-link related files/sections with relative markdown links (e.g. Glacier tier detail in `s3.md` links to `glacier.md` rather than duplicating it).
- Don't create a new sibling "advanced" file for overflow content (e.g. the old `s3-advanced.md`) — merge related material into the primary topic file's sections instead, and delete the sibling once merged (verify nothing else still references it first).
- When a section covers a service/concept commonly confused with another on the exam (e.g. two services that solve a similar-sounding problem), add a `> Exam-wording cue:` blockquote right after it, mapping distinctive question phrasing to the correct answer. Skip it when there's no real ambiguity to resolve — don't add one reflexively to every section.
- When asked to expand a section "based on the slides," go find the actual slide source in `converted/` rather than relying on general AWS knowledge — the point of these notes is to capture what this specific deck emphasizes.
