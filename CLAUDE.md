# yasine.org — project notes

This repo is the source for the `yasine.org` homepage and its tools. Read this
before making structural changes — it captures things that aren't obvious
from the code alone.

## Site map

| Host | Serves from | Source |
|---|---|---|
| `yasine.org` | `public_html/` | **This repo**, root. |
| `tools.yasine.org` | `public_html/tools/` (same physical folder as `yasine.org/tools/`) | This repo's `tools/` files. |
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

Do not try to upload files to Hostinger directly from a Claude Code session
via the Hostinger MCP connector's `generateUploadURLV1`/TUS flow — it hands
back a signed URL on `*.hstgr.io` that this sandboxed environment's network
policy blocks (403 at the egress proxy). That's an org-level policy, not a
bug; don't spend time retrying it. hPanel Git deploy is the working path.

## Homepage

`index.html` is a small hand-styled page (dark/light theme toggle, an
avatar with tools arranged in a circle around it). Tools come from one
`TOOLS` array near the bottom of the `<script>` block:
`{ name, desc, url }`. The circle layout auto-spaces however many entries
are in the array — no positions to hand-tune.

## Adding or updating a tool

1. Drop the tool's HTML file in `tools/`, kebab-case filename (no spaces —
   they break URLs).
2. Add one entry to `TOOLS` in `index.html`.
3. Commit, push to `main` (or merge a PR into it). Done.

For a tool hosted elsewhere and not mirrored in this repo (like Events),
give it an absolute `url` — the renderer detects `http(s)://` and adds
`target="_blank" rel="noopener"` automatically.

**When retiring a tool:** remove both its file under `tools/` *and* its
entry in `TOOLS` in the same change. A file deleted from the repo but left
in the array becomes a dead link once deployed — that happened once
already (Travel Advisor and the ETF guide were removed from the repo
without updating the array; fixed in the commit right before this note).

## Open question — server-side data cleanup

As of the last check, the old nested `public_html/tools/tools/` directory
(a leftover from an early manual upload, holding `Travels-store.php` /
`Travels-store.json` and some original space-named tool files) appears to
be gone from the live server. If that removal was intentional, nothing to
do. If not: `Travels-store.json` held real user data (saved trip research)
that was never in git, so it isn't recoverable from this repo — check
Hostinger backups/snapshots if it's needed back. Either way, verify this
with the site owner rather than assuming; don't try to reconstruct or
second-guess it here.

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
- Before shipping a new personal tool, check it for content that
  discloses specifics you wouldn't want public (exact dates, locations,
  financial figures) — fine for a private tool, worth genericizing on
  anything meant to be publicly linked.
