---
date: 2026-07-03
tags: [setup, quartz, cloudflare, infrastructure, first-project]
pages_read: []
pages_written: []
views_used: []
---

## Question
Set up `maeste/my-2nd-brain` (a raw/ → agent-compiled wiki/ pattern based on Karpathy's
"LLM wiki" idea) as a first coding project, with teaching along the way. Handover notes
said the vault should live somewhere near the existing `~/obsidian/brijse` Quartz site
but distinct from it — that decision changed significantly over the session (see below).

## Answer
Two nested projects ended up happening in one session:

**1. Where the vault lives (decided, reversed the original handover plan).** User chose
to install directly into `~/obsidian/brijse/content/` — the *same* public GitHub repo
that builds `brijse.com` — rather than a separate vault. Confirmed the repo is public,
walked through the privacy tradeoff explicitly (git-public vs. site-published are
different exposure levels), and landed on: `wiki/` builds and publishes (it's the
curated, teaching-facing output), while `raw/` (would republish full copies of others'
Substack articles), `conversations/`, `.lint/`, `.claude/`, `CLAUDE.md`/`AGENTS.md`, and
`inbox.md` are excluded from the Quartz build via `ignorePatterns` — present in git,
not rendered as pages.

**2. Quartz v4 → v5 migration (the actual bulk of the session).** Installing at
`content/` surfaced that the site was still on Quartz v4.2.2 while v5.0.0 existed
upstream — a full architectural rewrite (TS config → YAML, plugins became separately
installed community packages), not a version bump. Did the migration on an isolated
branch, caught and fixed several real bugs the tooling introduced (`quartz migrate`
silently fell back to empty config; explorer/spacer/tag-list/table-of-contents layout
regressions; two new-in-v5 plugins that broke the build or silently filtered out all
content). Verified locally with the exact Cloudflare build command before merging.
First push failed on a git buffer limit (merging the tag pulled in ~629 commits of
real upstream history); fixed via `http.postBuffer`. First Cloudflare deploy still
failed — `.quartz/plugins/` is gitignored by design and Cloudflare's build command
never ran the plugin-install step. Fixed the Cloudflare **Build command** to
`npx quartz plugin restore && npx quartz plugin resolve && npx quartz build`, verified
locally from a clean `.quartz/` state first, then confirmed live on `brijse.com`.

Wrote two reference docs, both committed: `QUARTZ_V5_MIGRATION.md` (the detailed
historical record of what was done and why) and `SITE_INSTALL_WATCHOUTS.md` (a
forward-looking checklist for the next tool upgrade or migration).

Then installed `my-2nd-brain`'s actual scaffold (`raw/`, `wiki/`, `CLAUDE.md`, skills,
slash commands) into `content/`, fixed the `ignorePatterns` leaks found via local build
(the `AGENTS.md` symlink crashed Quartz's parser entirely; `inbox.md` leaked through
initially too). Verified the visibility split works as intended.

## Open
- User is holding off on committing/pushing the vault scaffold itself until there's
  real content — currently just `git commit`-ready locally, not pushed.
- Agility Stories (the originally planned test content) is now *not* the plan — user
  feels stories should stay intact rather than get digested into wiki summaries, and is
  considering a separate, un-digested "stories" section of the site instead, at some
  later date.
- Actual first real content for the vault: user said they'd bring it "tomorrow" —
  nothing concrete chosen yet.
- Cosmetic: `v4` branch name left as-is (not renamed to `main`) — coordination with
  Cloudflare's Production branch setting made it not worth doing right now, revisit if
  ever motivated.
