To see
- `npx quartz build --serve`, run from the repo root (`~/obsidian/brijse`). It builds the site and serves it locally — by default at `http://localhost:8080`. It also auto-watches for file changes and hot-reloads the browser, so you can leave it running while you edit.
- If port 8080 is already busy, you can pick another: `npx quartz build --serve --port 3000`.

To push

- **Command line**: `git add -A && git commit -m "message"` then `git push origin v4`
- **GitHub Desktop**: stage changes, write a commit message, "Commit to v4," then "Push origin" — same effect, just a GUI