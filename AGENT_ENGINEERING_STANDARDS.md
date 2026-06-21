# Agent Engineering Standards

## Purpose
Capture the canonical engineering standards every Knitweb AI agent follows, adapted from Kun Chen's agent-profile policy rules.
Written as human-readable prose now; each is intended to become a content-addressed policy-Knit (Agent --follows--> policy) once the lens "content-addressed agent memory" epic lands, at which point this file becomes a generated export of the underlying policy CIDs, not a hand-maintained source.

## The 8 standards
1. No em dash; use a plain dash. (markdown-style-guide: no-em-dash)
2. Never auto-add the agent name as a commit co-author. (commit-hygiene: no-agent-coauthor)
3. Never hand-edit CHANGELOG.md or any auto-generated file. (no-autogen-edits)
4. In long Markdown, one full sentence per physical line; preserve structure. (markdown-style-guide: one-sentence-per-line)
5. For technical decisions, do not weight development cost; prefer quality, simplicity, robustness, scalability, long-term maintainability. (robustness-first)
6. For bug fixes, first reproduce the bug E2E on the real end-user path before fixing. (e2e-debugging)
7. During E2E, be obsessive about pixel-perfect UI; fix anything that looks off even if unrelated. (ui-pixel-perfect)
8. Hold one bar for engineering excellence; fix lint, test failures, and flakiness even when not caused by the current work. (excellence-bar)

## Becoming policy-Knits
Under the lens epic each standard is a Preference/Policy CID adopted via a `follows` Knit; profiles bundle subsets (a UI profile includes ui-pixel-perfect + e2e-debugging; a substrate profile need not).
Until that lands, treat this file as the authoritative human-readable standard for any agent in Knitweb repos.

## Notes
This file follows its own rule 4 (one sentence per line) and rule 1 (no em dash); vocabulary knitwebs/Knits, never "loom".
> reviewer: KEEP, P2. A doc (not a speculative tracking issue) is legitimate signal even though docs has 0 open issues; it is the named reference the lens epic and the pulse policy work both point at. Trimmed the meta-prose. This is the one item we deliberately put in a quiet repo because it is durable reference content, not backlog noise.