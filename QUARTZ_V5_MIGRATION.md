# Quartz v4 → v5 Migration Notes

Date: 2026-07-02
Branch used: `quartz-v5-upgrade` (branched off `v4`, merged back into `v4`, now deleted)
Result: `brijse.com` running Quartz v5.0.0, confirmed live via Cloudflare Pages.

This repo is a fork of `jackyzha0/quartz` itself (content lives inside the fork), not a
project that merely depends on a Quartz package. That's why the upgrade path is a real
`git merge` of an upstream tag, not an `npm update`.

## Why this was riskier than a normal version bump

Quartz v5 is an architectural rewrite, not a point release:

- `quartz.config.ts` + `quartz.layout.ts` (TypeScript) are gone, replaced by a single
  `quartz.config.yaml`.
- Plugins changed from "built into the Quartz codebase" to "separately installed
  community plugins" (`.quartz/plugins/`, gitignored, installed via CLI at build time).
- Comparing upstream tags `v4.5.2` → `v5.0.0`: 47 commits, 264 changed files.

## Steps taken

1. **Backed up** `quartz.config.ts` and `quartz.layout.ts` (needed later as input for
   `quartz migrate`, which reads them in place).
2. **Branched**: `git checkout -b quartz-v5-upgrade` off `v4`.
3. **Merged upstream v5.0.0**: `git fetch upstream v5.0.0 --tags && git merge v5.0.0`.
   Only one real conflict: `quartz.config.ts` (deleted upstream, modified locally) —
   resolved by accepting the deletion (backup already taken). `quartz.layout.ts` merged
   away cleanly with no conflict. `content/` was untouched by the merge.
4. **Restored** the backed-up `quartz.config.ts`/`quartz.layout.ts` into the working tree
   (merge had deleted them) so the migrate command below had something to read.
5. **`npm install`** — picks up v5's restructured dependency tree (179 added / 210
   removed / 169 changed packages).
6. **`npx quartz migrate`** — converts old `.ts` config to the new `quartz.config.yaml`.
   **Found a real bug here**: it printed `⚠ Failed to import TS config with tsx. Using
   defaults` and silently produced an **empty** `configuration: {}` block — meaning
   theme, page title, `ignorePatterns`, everything, would have silently reverted to
   Quartz's stock defaults if left unchecked.
7. **Manually reconstructed** the `configuration:` block by diffing the old
   `quartz.config.ts` against Quartz's own shipped `quartz.config.default.yaml` template.
   Turned out almost the entire old theme (colors, typography, footer links) was already
   just Quartz's stock defaults — the only genuine customizations were `pageTitle: "🪴
   Brij Se"` and `defaultDateType: created`. `ignorePatterns: [private, templates,
   .obsidian]` was restored too (this matters for future privacy/build-exclusion, not
   just cosmetics).
8. **Fixed four layout regressions** the migration introduced (compared against the old
   `quartz.layout.ts` component arrays):
   - `explorer`: was `display: all`, should be `desktop-only` (was `DesktopOnly(Explorer())`)
   - `spacer`: was `display: all`, should be `mobile-only` (was `MobileOnly(Spacer())`)
   - `tag-list`: was missing a `layout:` block entirely → wasn't positioned at all
   - `table-of-contents`: wrong priority (rendered *after* `backlinks` instead of before)
     and missing `desktop-only`
9. **Disabled two new-in-v5 plugins**, neither of which existed in the old config, both
   of which broke things:
   - `citations` — errored on existing content (`Cannot read non valid bibliography URL`)
   - `explicit-publish` — opt-in publish model (`publish: true` frontmatter required),
     which silently excluded **all 5** content files from the build. This one would have
     shipped an effectively empty site.
10. **`npx quartz plugin resolve`** reported "already installed" but a subsequent
    `npx quartz build` failed with `Could not resolve "../../.quartz/plugins"` — the
    barrel file `.quartz/plugins/index.ts` (auto-generated, aggregates plugin exports)
    only gets (re)written by actual install actions, and `resolve` skipped it since it
    found nothing new to install.
    **Fix: run `npx quartz plugin restore` too** — this forces the install path and
    regenerates the index. Restored 8 plugins `resolve` had missed entirely (citations,
    comments, explicit-publish, hard-line-breaks, ox-hugo, recent-notes, roam, tag-list).
11. **Verified locally**: `npx quartz build` (the exact command Cloudflare runs) — 0
    files filtered, all 5 content pages emitted. Served it (`npx quartz build --serve`)
    and visually confirmed theme, footer links, Explorer (desktop-only), Table of
    Contents, Graph View all render correctly.
12. **Committed** the config/layout work on `quartz-v5-upgrade`, then merged into `v4`
    (clean fast-forward, `v4` hadn't diverged from `origin/v4`).
13. **`git push origin v4` failed**: `HTTP 400`, `unexpected disconnect while reading
    sideband packet`. Cause: merging the v5.0.0 tag pulled in ~629 commits of Quartz's
    actual upstream history (not just the 47-commit v4→v5 diff), which combined with new
    binary images in `docs/` exceeded git's default HTTP post buffer.
    **Fix**: `git config http.postBuffer 524288000` (500MB, scoped to this repo only),
    then push succeeded.

## Cloudflare Pages — the change that actually mattered

Code being correct locally wasn't sufficient — the live deploy still failed. Build log
showed the identical `Could not resolve "../../.quartz/plugins"` error from step 10
above, because:

- `.quartz/plugins/` is (correctly) gitignored — it's meant to be installed fresh at
  build time, never committed.
- Cloudflare's **Build command** was just `npx quartz build`, with no install step first.

**Fix applied in Cloudflare dashboard** (Settings → Build → Build configuration → edit):

```
Build command (before): npx quartz build
Build command (after):  npx quartz plugin restore && npx quartz plugin resolve && npx quartz build
```

Build output directory (`public`) and root directory (blank) were unchanged. This exact
three-command sequence was verified locally from a fully clean `.quartz` state (moved
`.quartz/` aside, reran all three commands from scratch) before recommending the change,
to avoid a third failed deploy attempt.

Other Cloudflare facts confirmed along the way, worth knowing for next time:
- Production branch: `v4`. Automatic deployments: enabled. Build watch paths: `*`
  (wildcard — every file change anywhere in the repo triggers a rebuild+redeploy of
  production, not just Quartz-relevant files).
- Domains: `brijse.com`, `brijse.pages.dev`.
- Build system version 2 (a banner in the dashboard mentioned v3 is available — not
  evaluated as part of this migration).

## New plugins enabled by default that weren't in the old v4 setup

`npx quartz migrate` used Quartz's full default template plugin set rather than a
minimal one matching only what was previously configured. Left enabled (no evidence of
breakage, treated as new v5 defaults rather than bugs): `reader-mode`, `stacked-pages`,
`encrypted-pages`, `note-properties`, `bases-page`, `canvas-page`, `comments`,
`favicon`, `og-image`, `cname`, `hard-line-breaks`, `ox-hugo`, `recent-notes`, `roam`.
Two were disabled (see above): `citations`, `explicit-publish`. If something on the
live site looks unexpectedly different (e.g. a reader-mode toggle, stacked tab
navigation), it's likely one of these — check `quartz.config.yaml` and toggle
`enabled: false` if unwanted.

## Environment notes

- Quartz v5 requires **Node.js ≥ 22** (`package.json` engines field, `.node-version`
  pins `v22.16.0`). Local machine had Node v26.3.0 already, no action needed. Cloudflare
  detected and used `nodejs@22.16.0` automatically.
- Old `quartz.config.ts` / `quartz.layout.ts` were kept in the repo as reference
  (Quartz's own `migrate` convention — it doesn't delete them either).
