# yasine.org — project notes

This repo is the source for the `yasine.org` homepage and its tools. Read this
before making structural changes — it captures things that aren't obvious
from the code alone.

## Site map

| Host | Serves from | Source |
|---|---|---|
| `yasine.org` | `public_html/` | **This repo**, root. |
| `tools.yasine.org` | `public_html/tools/` (same physical folder as `yasine.org/tools/`) | Partly this repo, partly server-only files — see below. |
| `events.yasine.org` | `public_html/events/` | A separate app, not in GitHub. Only linked from the homepage. |
| `testing.yasine.org` | `public_html/testing/` | Unrelated scratch space. Ignore. |

`tools.yasine.org` and `yasine.org/tools/*` are **the same directory on
disk**. Anything this repo puts in `tools/` becomes reachable at both URLs.

## Deploy

Hostinger's hPanel has native Git deploy connected to this repo (`main`
branch → `public_html`). **Merging to `main` deploys automatically** —
nothing else to run. Confirm in hPanel: **Websites → yasine.org → Git**.

`.github/workflows/pages.yml` also deploys to GitHub Pages on push to
`main`. That's a harmless secondary preview at
`santogermano.github.io/Tools` — it is **not** what serves the live site.
Don't confuse the two, and don't remove it without reason.

Do not try to upload files to Hostinger directly from a Claude Code session
via the Hostinger MCP connector's `generateUploadURLV1`/TUS flow — it hands
back a signed URL on `*.hstgr.io` that this sandboxed environment's network
policy blocks (403 at the egress proxy). That's an org-level policy, not a
bug; don't spend time retrying it. hPanel Git deploy is the working path.

## Adding or updating a tool

1. Drop the tool's HTML file in `tools/`, kebab-case filename (no spaces —
   they break URLs).
2. Add one entry to the `TOOLS` array near the bottom of `index.html`:
   `{ name, desc, url: 'tools/your-file.html' }`. That's the entire build
   step — no bundler, no manifest to regenerate.
3. Commit, push to `main` (or merge a PR into it). Done.

For a tool hosted elsewhere and not mirrored in this repo (like Events, or
Travel Advisor below), give it an absolute `url` — the renderer detects
`http(s)://` and adds `target="_blank" rel="noopener"` automatically.

## Travel Advisor — do not "fix" this by re-deploying from the repo

`tools/travel-advisor.html` in this repo is a **stale, feature-incomplete
duplicate**. The real, current version lives only on the server at
`public_html/tools/tools/Travels.html` (reachable as
`https://tools.yasine.org/tools/Travels.html`), where someone hand-extended
it with a trip-idea save feature backed by `Travels-store.php` /
`Travels-store.json` (server-side, PHP, not in git). The homepage links
directly to that live URL instead of shipping the repo's outdated copy.

If you ever want to bring Travel Advisor back under version control:
pull the *live* file's content into the repo (not the other way around),
and think carefully about `Travels-store.json` before doing anything with
it — see next section.

`tools/travel-advisor.html` itself is currently unreferenced dead weight in
the repo. It should be deleted (`git rm tools/travel-advisor.html`) — a
prior session's safety guard blocked the automated deletion, so it's still
sitting there pending a manual `git rm`.

## Files that live on the server only — never add these to git

Inside `public_html/tools/tools/` (yes, double-nested — an artifact of an
old manual upload): `Travels-store.php`, `Travels-store.json`,
`Travels-store.lock`, `Travels-map.html`, `manifest.json`, plus the original
space-named tool files (`Foody food week.html`, `Newsletter Generator.html`,
`Travels.html`).

`Travels-store.json` in particular holds live, user-generated data (saved
trip ideas), written at runtime by `Travels-store.php`. **Never commit this
file.** If it ever ends up tracked in git, a future deploy could overwrite
real saved data with a stale committed copy, or (if hPanel does a hard
reset rather than a soft pull) silently destroy it. Git deploys are
additive/non-destructive by default *only* as long as these files stay
untracked — leave them alone.

`public_html/tools/index.html` also still exists as a leftover — the old
GitHub-API-driven homepage design, now stale relative to this repo's
`index.html`. It's what currently makes `tools.yasine.org` show a different
(older) homepage than `yasine.org`. Worth cleaning up or redirecting later;
not touched by this repo's deploy since this repo has no `tools/index.html`.

## Security posture

**This file itself is deployed.** hPanel's Git deploy pulls the *whole*
repo into `public_html`, so anything committed here — including this
file — is reachable at `yasine.org/CLAUDE.md`. Don't record vulnerability
specifics, credentials, or exploit-relevant detail in any repo file,
including this one; raise those privately with the site owner instead of
writing them down here.

- No secrets, API keys, or credentials belong in any file under `tools/` or
  in `index.html`. All of them are served as-is, publicly, with no build
  step to strip anything out. Grep for common key patterns
  (`sk-`, `AIza`, `ghp_`, `xox`, `AKIA`, `api_key`, `secret`, `token`,
  `password`) before committing a new tool — and treat the same rule for
  PR descriptions and commit messages on this repo, since it's public too.
- `index.html` carries `<meta name="robots" content="noindex, nofollow">`
  since these are personal tools, not a marketing site. Remove deliberately
  if you want the homepage indexed.
- The server-only files listed above (Travel Advisor's save backend
  especially) were flagged for an access-control review with the site
  owner directly — check with them on status before assuming it's handled.
- Before shipping a new personal tool, check it for the same class of
  thing the Travel Advisor page has (a title/header disclosing specifics
  like exact travel dates) — fine for a private tool, worth genericizing
  on anything meant to be publicly linked.
