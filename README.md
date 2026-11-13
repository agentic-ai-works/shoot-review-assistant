# Shoot Review Assistant

A UGREEN AI NAS Skill that gives photographers a safe first-pass review of one shoot, entirely on the NAS.

After a shoot there are hundreds of near-identical frames. Name a folder and, optionally, a subject, and the Skill:

1. takes an inventory of the folder (photos, videos, formats);
2. reports duplicate images as the NAS dedupe scan finds them;
3. finds frames of your subject (photo search, classification, OCR);
4. proposes a shortlist and waits for you to confirm it;
5. matches the confirmed files to library photos (anything unmatched is reported and left out);
6. creates a "Selects" album from exactly the confirmed photos;
7. optionally applies tags (it looks the tag up first and asks before creating a missing one).

**Safety:** read-only until you approve. It never deletes, moves or renames files. It calls only the NAS experts, never an external service or model, so photos stay on the device.

## Files

- `SKILL.md`: the Skill (Uliya Skill standard: name and description frontmatter, goal, input, capability evidence, orchestration, completion and failure behaviour)
- `references/workflow.md`: the step-by-step flow and edge cases
- `references/disclosure.md`: third-party and AI-assistance disclosure

There are no scripts, binaries or manifests. To package it, zip `SKILL.md` and `references/`.

## Status

Written and checked by hand against the UGREEN Skill format rules and capability catalogue. **Not yet tested on a real NAS.**

## AI disclosure

Drafted with help from Claude (Anthropic) and reviewed by the author. No third-party components are bundled. The UGREEN Skill Developer Package was used as documentation only.

Built for the UGREEN Hackathon 2026 on AI NAS.

## Licence

MIT, see `LICENSE`.
