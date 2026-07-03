---
type: page
created: 2026-07-03
updated: 2026-07-03
tags: [ai, tooling, claude-code]
---

# My AI Setup

The author's current AI/agent working setup, in two parts:

**Working principles** — five rules for how an AI coding agent should operate: ask
rather than assume (and record the assumption when running unattended instead of
blocking), match solution complexity to the problem, don't touch unrelated code but do
surface issues found in passing, flag uncertainty explicitly (small low-risk
experiments over false confidence), and proactively suggest better approaches. Sourced
from a Reddit post extending Andrej Karpathy's original 4-rule CLAUDE.md.
([[wiki/sources/ai-working-principles-karpathy]])

**This vault itself** — `my-2nd-brain`, a raw/ → agent-compiled wiki/ pattern, also
based on a different Karpathy idea (the "LLM wiki" gist). Sources are curated by the
author; the agent fetches, ingests, and maintains the wiki; views (timelines,
comparisons, slides) are built on request. Five invariants govern it: raw/ is
immutable, every claim cites a source, summaries are paraphrased not copied, the user
curates while the agent maintains, and shareable:true views are frozen snapshots.
([[wiki/sources/my-2nd-brain-getting-started]])

## Sources
- [[wiki/sources/ai-working-principles-karpathy]]
- [[wiki/sources/my-2nd-brain-getting-started]]

## Related
- [[wiki/pages/ai-fluency]] — the broader focus area this setup serves
