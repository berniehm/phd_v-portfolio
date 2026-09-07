# Getting the project onto Bernadette's GitHub

**The problem:** Claude Code keeps pushing to Victor's account instead of Bernadette's.
**The cause:** almost certainly the GitHub CLI on that machine is signed in as Victor. Claude Code uses `gh` to create and push repos, so it inherits whoever `gh` is authenticated as — regardless of who is sitting at the keyboard.

Two routes below. **Route A avoids the problem entirely and takes about fifteen minutes.** Route B fixes it properly and is worth doing afterwards.

---

## Route A — upload through the browser

No terminal, no CLI, nothing to debug. Bernadette does all of this signed in as herself.

### 1. Create the repo

github.com → **New repository**

- Name: `phd-portfolio`
- **Private** — this matters, see the note at the end
- Do **not** tick "Add a README"

### 2. Prepare the folder

In her project folder, delete these before uploading:

- `node_modules/` — thousands of files, must never go in a repo
- `dist/` or `build/` — generated output
- `.env` if one exists

What is left should be a few dozen files.

### 3. Upload

On the empty repo page: **uploading an existing file** → drag the whole folder in → Commit.

GitHub's uploader preserves folder structure, so `src/components/...` stays where it belongs.

### 4. Add the two config files

Still in the browser, **Add file → Create new file**.

`.gitignore`
```
node_modules
dist
.env
.DS_Store
```

`netlify.toml`
```toml
[build]
  command = "npm run build"
  publish = "dist"
```

### 5. Connect Netlify to the repo

This replaces the drag-and-drop deploy with a real pipeline — push a change, site rebuilds.

Netlify → the existing site → **Site configuration → Build & deploy → Link repository** → GitHub → authorise → pick `phd-portfolio`.

- Build command: `npm run build`
- Publish directory: `dist`

Deploy. The URL stays the same.

### 6. Add Victor as a collaborator

Repo → **Settings → Collaborators → Add people** → his GitHub username → **Write** access.

Write access is what lets him edit `register.ts` in GitHub's web editor. Read-only would not.

**Done.** He updates content in the browser, Netlify rebuilds, neither of them touches a terminal.

---

## Route B — fix the CLI properly

Worth doing, because she will want git working under her own name for everything else, and because commit history is part of the portfolio value.

### Find out who the machine thinks she is

```bash
gh auth status
git config --global user.name
git config --global user.email
```

If any of those come back as Victor, that is the whole problem.

### Switch accounts

```bash
gh auth logout
gh auth login          # choose GitHub.com, HTTPS, browser — sign in as Bernadette
git config --global user.name  "Her Name"
git config --global user.email "her@email.com"
```

The email must match one on her GitHub account, or commits will not be attributed to her — they will show as a faded avatar with no profile link, which defeats the point for a portfolio.

### Clear the cached token

This is the step people miss, and it is why the wrong account keeps coming back.

- **macOS:** Keychain Access → search `github` → delete the `github.com` entries
- **Windows:** Credential Manager → Windows Credentials → remove `git:https://github.com`
- **Linux:** `git credential-cache exit`

### Point the folder at her repo

```bash
cd path/to/project
git remote -v                    # if this shows Victor's URL, that is the culprit
git remote set-url origin https://github.com/HER-USERNAME/phd-portfolio.git
git push -u origin main
```

### Check before trusting it

```bash
git log --format='%an <%ae>' -3
```

Her name and email on the recent commits means it is fixed.

**If they share the machine,** set the identity per-repo instead of globally — drop `--global` from the two `git config` commands inside her project folder, so his repos keep his identity.

---

## What the repo should contain

```
phd-portfolio/
├── .gitignore
├── netlify.toml
├── package.json
├── package-lock.json
├── tsconfig.json
├── vite.config.ts
├── index.html
├── README.md              ← Milestone 4
├── CASE_STUDY.md          ← Milestone 4
├── .github/workflows/
│   └── typecheck.yml      ← catches a broken register.ts before deploy
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── types.ts
    ├── styles.css
    ├── data/
    │   ├── register.ts    ← the file Victor edits
    │   └── demo.ts        ← synthetic content for the public build
    └── components/
        ├── Header.tsx
        ├── FilterBar.tsx
        ├── BlockGrid.tsx
        ├── BlockCard.tsx
        ├── DetailPanel.tsx
        ├── ProgressBar.tsx
        ├── DebtList.tsx
        └── Tex.tsx
```

`typecheck.yml`, worth adding early — it stops a stray comma in `register.ts` from blanking the live site:

```yaml
name: typecheck
on: [push]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx tsc --noEmit
```

---

## Keep the repo private

`register.ts` holds unpublished research — Paper 1 is under review, Papers 2 to 6 are unpublished, and Papers 8 to 10 involve other people's work and other people's names. None of that should sit in a public repo.

That does not cost Bernadette the portfolio piece. The plan already separates the two:

- **Private repo, private deploy** — the real content
- **Public demo** — same codebase, `VITE_DATASET=demo`, synthetic content, deployed as a second Netlify site from the same private repo

The demo URL is what goes on her CV. She can also make the repo public later, once the papers are published and the demo dataset is the only content — but that is a decision for 2028, not now.

---

## Order to do this in

1. Route A, steps 1 to 6 — the site is Git-connected and Victor can edit content
2. Route B — her identity fixed, so future commits are hers
3. Add the Edit link to each item's panel
4. Then Milestone 3, the ready queue

Nothing after step 1 is urgent. Getting off drag-and-drop deploys is the thing that unblocks everything else.
