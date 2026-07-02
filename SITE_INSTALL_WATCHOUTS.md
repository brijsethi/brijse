# Watch-outs for future installs/upgrades on this site

Lessons from installing Quartz v5 and `my-2nd-brain` (2026-07-02). Read this before
upgrading either tool again, or installing a different second-brain tool. See
`QUARTZ_V5_MIGRATION.md` for the detailed blow-by-blow of the v5 migration specifically
— this doc is the generic checklist for *next* time.

## Structural facts about this repo, easy to forget

- **This repo is a fork of Quartz itself** — your content lives inside the same repo as
  Quartz's own source code (`quartz/`, `quartz.config.yaml`, etc.), not in a separate repo
  that merely references a Quartz package. Any tool you install needs to coexist with
  that, and any Quartz upgrade is a real `git merge` of an upstream tag, not `npm update`.
- **`content/` is the live build root.** Anything placed there — new folders, dotfiles,
  symlinks — gets scanned by Quartz as page content unless excluded via `ignorePatterns`
  in `quartz.config.yaml`. This bit us twice in one afternoon (`AGENTS.md` symlink,
  `inbox.md`). Whenever a tool adds new top-level files/folders under `content/`, assume
  they'll try to become public pages until proven otherwise.
- **Current `ignorePatterns`**: `private`, `templates`, `.obsidian`, `raw`,
  `conversations`, `.lint`, `.claude`, `CLAUDE.md`, `AGENTS.md`, `inbox.md`. If a future
  tool version adds new scaffold files/folders, check this list needs extending before
  assuming they're hidden.
- **Public repo.** Anything git-committed is visible in GitHub history forever, even if
  later excluded from the Quartz build or deleted. "Excluded from the site" and "private"
  are not the same thing here — decide the actual privacy need before committing.

## Gotchas specific to Quartz's plugin/build system (v5+)

- **Symlinks crash the markdown parser.** `AGENTS.md` (symlinked to `CLAUDE.md` by
  `my-2nd-brain`'s installer) caused a hard parse error until excluded. Any tool that
  creates symlinks inside `content/` will likely need the same treatment.
- **Cloudflare's build command doesn't install Quartz plugins on its own.**
  `.quartz/plugins/` is gitignored by design (installed fresh at build time). The build
  command must be:
  ```
  npx quartz plugin restore && npx quartz plugin resolve && npx quartz build
  ```
  not just `npx quartz build`. If Quartz's plugin system changes again in a future
  version, re-verify this is still the right sequence — check `docs/cli/*.md` in the
  Quartz repo itself for the current CLI reference.
- **Automated config migration tools can silently produce wrong output.**
  `npx quartz migrate` printed a warning but still generated a broken config (empty
  `configuration: {}`, several layout bugs) that would have been easy to miss. Always
  diff the tool's output against the old config by hand, don't just trust a "success"
  message.

## Process that actually caught real bugs — repeat it

1. **Test locally with the exact Cloudflare build command before pushing anything.**
   This caught two build-breaking bugs (a plugin that silently filtered out all content,
   another that crashed on existing files) before they ever reached the live site.
2. **Simulate a truly clean environment**, not just "it works on my machine with stuff
   already cached" — e.g. temporarily move `.quartz/` aside and rerun from scratch, since
   that's what Cloudflare's fresh clone actually does.
3. **Check `git status`/`git diff` of generated config against the old one**, field by
   field, rather than assuming a migration tool preserved everything.

## Environment

- Quartz v5 needs **Node.js ≥ 22**. Local machine's default `node` was already v26,
  Cloudflare auto-detects from `.node-version`/`package.json` engines — but if trying a
  different tool/hosting setup, check this explicitly.
- The shared local Python (`~/.local/bin/python3.11`, managed by `uv`) refuses direct
  `pip install` — it's shared across projects. Any tool needing Python packages
  (`my-2nd-brain`'s `inbox-fetcher` needs `trafilatura`, `requests`, `python-slugify`,
  Python 3.10+) should get its own `uv venv .venv` scoped to wherever it runs, not a
  global install.
- Large `git push` after merging an upstream tag can fail (`HTTP 400`, buffer limit) —
  merging pulls in the tag's *entire* upstream history, not just the diff you expect.
  Fix: `git config http.postBuffer 524288000` (scoped to the repo, not global).

## Cloudflare specifics worth re-checking if anything changes

- Production branch: `v4` (kept as-is, not renamed to `main` — cosmetic only, not worth
  the coordination risk right now).
- Build watch paths: `*` — every file change anywhere in the repo triggers a full
  production rebuild+redeploy, not just Quartz-relevant changes.
- Build output: `public`, root directory: blank (repo root).
