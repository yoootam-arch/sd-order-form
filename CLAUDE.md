# CLAUDE.md — sd-order-form

Context for any Claude session working in this folder. Written 2026-08-14; every fact below was
verified live that day unless marked otherwise.

## What this is

**טופס הזמנת חומרים רדיואקטיביים** — a Hebrew (RTL) web form for ordering radioactive materials,
plus an admin panel. Two halves that deploy to two different places:

| Half | File | Runs on | Deployed with |
|---|---|---|---|
| **Frontend** | `index.html` (single file, ~1,600 lines) | Netlify | `netlify deploy` (site linked via `.netlify/state.json`) |
| **Backend** | `gas/Code.gs` + `gas/appsscript.json` | Google Apps Script Web App | `clasp push` → `clasp deploy` |

Features built so far (from git history): contact management with admin CRUD, orderer dropdown
sorted א-ת, inline-editable roles, CC auto-assignment for new orderers, early-arrival hint for
items before 14:00, a test mode that shows the original To/CC recipients, and a clear-form button.

## 🔴 The repo is PUBLIC

`github.com/yoootam-arch/sd-order-form` — **public**. This is the dominant constraint here and it
has already caused one cleanup (commit `cc38239`, *"Remove GAS code from public repo, move password
to Script Properties"*).

**Never commit:** the admin password (it lives in **Apps Script Properties**, not in code) ·
recipient email addresses · the Apps Script ID · the Netlify site ID · anything from `gas/`.

Already git-ignored, and each for a reason — do not "helpfully" un-ignore them:

```
gas/            the backend. Lives on Google's servers; deliberately out of the public repo
.clasp.json     holds the Apps Script ID (ignored 2026-08-14)
.netlify        holds the Netlify site ID
```

## ⚠️ Consequence that is easy to miss: the backend has no backup

Because `gas/` is git-ignored, **a fresh clone of this repo does not contain the backend at all.**
`gas/Code.gs` (~11.7 KB — the entire server side) exists in exactly **two** places: this PC's disk,
and Google's servers. It is in no git history, public or private. `clasp pull` can recover it from
Google, which is the real safety net — but there is no versioned copy anywhere.

## Deploying

**Backend (Apps Script):**
```bash
clasp push          # rootDir is "gas"
clasp deploy        # creates a new numbered deployment
clasp deployments   # list; @HEAD is the dev endpoint, numbered ones are stable
```
✅ Verified 2026-08-14: `clasp` 3.3.0, and the credentials in `~/.clasprc.json` (from 2026-04-19)
**are still valid** — `clasp deployments` returned 3 deployments. The live one is `@25`, labelled
*"password from script properties"*.

**Frontend (Netlify):** the site is already linked (`.netlify/state.json`); `netlify` CLI is
installed on the PC.

⚠️ **Both toolchains are authenticated on the PC only.** Working from a clone on another machine
would mean a second OAuth against the same Google account. Don't — see below.

## Working model

Edit on the PC → commit → push → deploy from the PC. The MacBook connects to the PC over
Remote-SSH and is only a screen; **do not clone this repo on the Mac.** A second clone drifts, is
invisible to the commit-reminder hook (which scans `C:\Users\yooot\projects\` on the PC), and would
need its own clasp/Netlify credentials. See `nas-homelab/docs/REMOTE_DEV_ACCESS.md`.

**Git protocol** is global (`~/.claude/CLAUDE.md`): at the end of any session that touched files
here, run `git status`, summarize, and **ask** before committing. Never auto-commit or auto-push.
Given this repo is public, the secret scan before pushing is not optional.

## Gotchas

- **The password is in Script Properties**, not in `Code.gs`. If a code path seems to be missing a
  password, that is why — read it with `PropertiesService`, never hardcode it back.
- **The UI is Hebrew RTL.** Test rendering, not just logic; a bidi break shows up only visually.
- **`index.html` is one big file** — frontend, styling and client JS together. There is no build
  step and no bundler.
- **Test mode exists** and reports the real To/CC it *would* have used — prefer it over sending
  live mail while iterating.
- There is **no README** and no test suite. Git history is the only narrative; `git log --oneline`
  is worth reading before changing behaviour.
