# UniTop Rebrand Fork — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fork `TencentCloud/Octop` to `759012193/UniTop`, replace user-facing "Octop" branding with "UniTop", strip Tencent Cloud active promotion (keep LICENSE/attribution), retarget all CI/Release pipelines, and add macOS + Windows code-signing/notarization — without changing runtime identifiers (CLI `octop`, Python package `octop`, paths `~/.octop/`, Docker image name).

**Architecture:** Single GitHub fork + `unitop/branding` long-lived branch. All UniTop-specific edits are in-place on existing upstream files (no overlay directories). 10 logical commits on `unitop/branding`, each rebaseable onto `upstream/main`. Runtime behavior unchanged.

**Tech Stack:** Python 3.12, FastAPI, React 18 + Vite + TS, Wails (Go + WebView) for desktop, GitHub Actions, Apple notarization (notarytool + App Store Connect API), Azure Trusted Signing (signtool via OIDC).

**Spec:** `docs/superpowers/specs/2026-09-17-unitop-rebrand-design.md`

## Global Constraints

- All edits are **in-place** on existing upstream files. Do not create new directories like `unitop/branding/`.
- Runtime identifiers **must not change**: `octop` (Python package name, CLI command), `~/.octop/` (data path), `OCTOP_*` env vars, Docker image name, GitHub Actions environment `pypi`, `octop.cli.main:cli` entry point.
- License: **Do not modify** `LICENSE`. Add upstream attribution in `README.md` and `AGENTS.md`.
- Branch: **all work happens on `unitop/branding`**. `main` stays clean for upstream rebase.
- Commits: each task ends with exactly one commit. Conventional Commits style.
- Wails desktop stack — **not Tauri**. Edit `desktop/src/build/{darwin,windows,linux}/*.yml` and `.nsi`, not Tauri config.
- Windows code signing path: **W1 Azure Trusted Signing** (default). Switch to W2 (`.pfx`) only if user instructs in Q1.
- macOS: signing identity string `Developer ID Application: <TODO: cert CN> (${{ secrets.APPLE_TEAM_ID }})` is a literal placeholder to be replaced when `APPLE_TEAM_ID` is provisioned.
- Docker Hub publishing: **disabled by default** — only GHCR. Switch on only if Q1 sees Docker Hub account confirmed.

---

## Phase 1: Setup

### Task 1: Fork repo, clone, create branch, commit spec

**Files:**
- Create: `/Volumes/MacMini/unitop/` (clone)
- Create: `docs/superpowers/specs/2026-09-17-unitop-rebrand-design.md` (already exists locally — gets committed)
- Create: `docs/superpowers/plans/2026-09-17-unitop-rebrand.md` (already exists locally — gets committed)

**Interfaces:**
- Produces: working `git` repo at `/Volumes/MacMini/unitop` with `origin` → `759012193/UniTop`, `upstream` → `TencentCloud/Octop`, current branch `unitop/branding` with the spec+plan as its first commit

- [ ] **Step 1: Fork the upstream repo via GitHub CLI**

```bash
gh repo fork TencentCloud/Octop \
  --org 759012193 \
  --name UniTop \
  --remote \
  --description "UniTop — rebranded fork of TencentCloud/Octop (MIT)"
```

Expected: prints "✓ Created fork 759012193/UniTop" and clones to current directory.

- [ ] **Step 2: Move/rename the clone to the canonical local path**

```bash
# If gh fork cloned into ./UniTop, move it
if [ -d ./UniTop ]; then
  rm -rf /Volumes/MacMini/unitop
  mv ./UniTop /Volumes/MacMini/unitop
fi
cd /Volumes/MacMini/unitop
```

Verify: `git remote -v` shows `origin → https://github.com/759012193/UniTop.git`.

- [ ] **Step 3: Add upstream remote and fetch**

```bash
cd /Volumes/MacMini/unitop
git remote add upstream https://github.com/TencentCloud/Octop.git
git remote set-url --push upstream no_push  # prevent accidental push to upstream
git fetch upstream
git branch -u upstream/main main
```

Verify: `git remote -v` shows both remotes; `upstream` has `(push)` blocked.

- [ ] **Step 4: Copy the locally-staged spec + plan into the fork**

```bash
cd /Volumes/MacMini/unitop
mkdir -p docs/superpowers/specs docs/superpowers/plans
cp /Volumes/MacMini/unitop/docs/superpowers/specs/2026-09-17-unitop-rebrand-design.md \
   docs/superpowers/specs/
cp /Volumes/MacMini/unitop/docs/superpowers/plans/2026-09-17-unitop-rebrand.md \
   docs/superpowers/plans/
```

Note: the spec and plan already exist as raw files in `/Volumes/MacMini/unitop/docs/...` outside the fork. This step copies them inside the fork so they're tracked.

- [ ] **Step 5: Create the long-lived branch and commit the spec+plan**

```bash
cd /Volumes/MacMini/unitop
git checkout -b unitop/branding
git add docs/superpowers/
git commit -m "docs(spec): add UniTop rebrand design + implementation plan"
```

Verify: `git log --oneline -1` shows the commit on `unitop/branding`.

- [ ] **Step 6: Push the branch**

```bash
cd /Volumes/MacMini/unitop
git push -u origin unitop/branding
```

Verify: prints `branch 'unitop/branding' set up to track 'origin/unitop/branding'`.

- [ ] **Step 7: Verify the spec is readable on GitHub**

```bash
gh browse --branch unitop/branding docs/superpowers/specs/2026-09-17-unitop-rebrand-design.md
```

Expected: opens browser to the file on github.com. (URL only — close doesn't require interactive.)

---

## Phase 2: Branding (in-place edits)

### Task 2: Rebrand README + CHANGELOG + AGENTS.md

**Files:**
- Modify: `README.md`, `README_CN.md`, `CHANGELOG.md`, `AGENTS.md`
- Modify: `docs/assets/octop-banner.png`, `docs/assets/octop-logo.svg`, `docs/assets/readme-banner.png` (asset swap — see Step 7)

**Interfaces:**
- Consumes: user-supplied UniTop brand assets in `unitop-assets/` directory (banner PNG 1200×630, logo SVG, banner-zh PNG)
- Produces: README files where every visible "Octop" reads as "UniTop" except `TencentCloud/Octop` upstream attribution

- [ ] **Step 1: Survey all "Octop" occurrences**

```bash
cd /Volumes/MacMini/unitop
grep -n "Octop" README.md | head -100
grep -n "octop" README.md | head -100
```

Review output. Goal: identify (a) bare "Octop" / "octop" → UniTop candidates, (b) `TencentCloud/Octop` and `octop.cloud` to keep or replace.

- [ ] **Step 2: Replace project title and headings**

Edit `README.md`:
- Line 1 `# Octop` → `# UniTop`
- Find: `# 🐙 Octop` patterns → `# UniTop`
- Find: `## What is Octop?` → `## What is UniTop?`

Edit `README_CN.md`:
- `# Octop` → `# UniTop`
- `# 🐙 Octop` → `# UniTop`

Verify: `grep -nE "^#.*Octop" README.md README_CN.md` shows only the upstream attribution (none of the headings).

- [ ] **Step 3: Replace body-text mentions (preserving upstream URLs)**

In both `README.md` and `README_CN.md`, do a contextual replace:
- `Octop is` → `UniTop is`
- `Octop's` → `UniTop's`
- `Octop:` → `UniTop:`
- `with Octop` → `with UniTop`
- `install Octop` → `install UniTop`

**Do NOT replace** anywhere these strings appear:
- `TencentCloud/Octop` (upstream attribution — keep)
- `https://github.com/TencentCloud/Octop` (upstream URL — keep)
- `pip install octop` (PyPI package name — keep, runtime identifier)
- `~/.octop/` (data path — keep, runtime identifier)
- `octop-cloud` style strings — investigate case-by-case
- Code blocks containing `octop` as CLI command — keep

Use `sed` with word boundaries for safety:

```bash
cd /Volumes/MacMini/unitop
sed -i '' 's/\bOctop\b/UniTop/g' README.md README_CN.md
```

Then manually revert any over-replacements using `git diff`.

- [ ] **Step 4: Add upstream attribution banner at top of README**

Insert at top of `README.md` (after the title), and similarly for `README_CN.md`:

```markdown
> **Fork notice:** This is **UniTop**, a rebranded fork of
> [TencentCloud/Octop](https://github.com/TencentCloud/Octop) (MIT).
> See [AGENTS.md](./AGENTS.md) for fork policy and [LICENSE](./LICENSE) for original terms.
```

- [ ] **Step 5: Add a `## UniTop fork` entry to CHANGELOG.md**

Prepend to `CHANGELOG.md`:

```markdown
## [Unreleased] — UniTop fork

- Forked from [TencentCloud/Octop](https://github.com/TencentCloud/Octop) at upstream/main
- Cosmetic rebrand: Octop → UniTop (UI, desktop app, NAS package, README)
- Removed Tencent Cloud active promotion (WeCom QR, COS install URLs, default connectors)
- Added macOS code signing + notarization
- Added Windows code signing (Azure Trusted Signing)
- Discontinued PyPI publishing from this fork (package `octop` is owned by TencentCloud on PyPI)

## [Upstream Changelog](https://github.com/TencentCloud/Octop/blob/main/CHANGELOG.md)
```

- [ ] **Step 6: Add fork-policy section to AGENTS.md**

Prepend to `AGENTS.md`:

```markdown
# UniTop fork policy

This repository is a rebranded fork of
[TencentCloud/Octop](https://github.com/TencentCloud/Octop), licensed under MIT.

## Rebrand scope

User-facing surfaces only: project name, desktop app name, README, web dashboard,
docs. **Runtime identifiers preserved**: Python package `octop`, CLI command
`octop`, data path `~/.octop/`, Docker image, environment variables.

## Upstream attribution

We retain the upstream LICENSE file and `LICENSE` references throughout. We
do not remove `TencentCloud/Octop` references that are necessary for upstream
attribution. We do remove Tencent Cloud **active promotion** (WeCom QR,
COS-hosted installer, default connector bundles).

## Syncing upstream

```bash
git fetch upstream
git checkout main
git merge upstream/main
git checkout unitop/branding
git rebase main
```

## Original work below

---
```

Then keep the original AGENTS.md content below the `---`.

- [ ] **Step 7: Swap brand assets**

```bash
cd /Volumes/MacMini/unitop
# Verify user has placed assets in unitop-assets/ with these names
ls -la unitop-assets/ 2>/dev/null || {
  echo "ERROR: Create unitop-assets/ with files: banner.png, logo.svg, banner-zh.png, chat.png, chat-zh.png"
  exit 1
}
git rm docs/assets/octop-banner.png docs/assets/octop-logo.svg \
        docs/assets/readme-banner.png docs/assets/readme-banner-zh.png \
        docs/assets/readme-chat.png docs/assets/readme-chat-zh.png 2>/dev/null
cp unitop-assets/banner.png docs/assets/readme-banner.png
cp unitop-assets/logo.svg docs/assets/octop-logo.svg  # keep filename to minimize diff
cp unitop-assets/banner-zh.png docs/assets/readme-banner-zh.png
cp unitop-assets/chat.png docs/assets/readme-chat.png
cp unitop-assets/chat-zh.png docs/assets/readme-chat-zh.png
git add docs/assets/
```

If user has not yet provided assets, **skip this step** and note it as a blocker — the rest of Task 2 still proceeds; assets can be added later.

- [ ] **Step 8: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -nE "(^|[^a-zA-Z])Octop([^a-zA-Z]|$)" README.md README_CN.md | grep -v "TencentCloud/Octop"
```

Expected: no output (all visible Octop mentions are now UniTop, except upstream URL).

```bash
grep -c "UniTop" README.md README_CN.md
```

Expected: ≥ 5 per file.

- [ ] **Step 9: Update pyproject.toml metadata**

Edit `pyproject.toml`:

```toml
# Find:
description = "Smarter self-hosted AI assistant for multiple users and agents, built on harness-agent"
# Replace with:
description = "UniTop — smarter self-hosted AI assistant for multiple users and agents (forked from TencentCloud/Octop)"

# Find:
authors = [{name = "octop contributors"}]
# Replace with:
authors = [{name = "UniTop contributors"}]

# Find:
readme = "README.md"
# Keep as-is.
```

**Do NOT change** `name = "octop"` (line above `version`) — runtime identifier per spec §0.2.

Verify:

```bash
cd /Volumes/MacMini/unitop
grep -A1 "^name " pyproject.toml | head -3
grep "^description" pyproject.toml
grep "^authors" pyproject.toml
```

Expected:
- `name = "octop"` (unchanged)
- `description = "UniTop — ..."` (updated)
- `authors = [{name = "UniTop contributors"}]` (updated)

- [ ] **Step 10: Commit**

```bash
cd /Volumes/MacMini/unitop
git add README.md README_CN.md CHANGELOG.md AGENTS.md docs/assets/ pyproject.toml
git commit -m "docs: rebrand README/CHANGELOG/AGENTS + pyproject metadata to UniTop + swap brand assets"
git push origin unitop/branding
```

---

### Task 3: Rebrand dashboard UI (HTML title, i18n, layout, app icons)

**Files:**
- Modify: `dashboard/src/index.html`
- Modify: `dashboard/src/i18n/locales/*.json` (all locale files)
- Modify: `dashboard/src/components/Layout/Sidebar*.tsx`, top bar components
- Modify: `dashboard/public/apple-touch-icon.png`, `dashboard/src/assets/app/` — side files
- (See file survey in spec §2.A)

**Interfaces:**
- Consumes: user-provided `unitop-assets/dashboard-logo.svg`, `unitop-assets/favicon.ico`
- Produces: dashboard where `<title>` reads "UniTop", all UI strings show "UniTop" instead of "Octop", brand icons swapped

- [ ] **Step 1: Locate every "Octop" string in dashboard sources**

```bash
cd /Volumes/MacMini/unitop
grep -rn "Octop" dashboard/src/ dashboard/public/ dashboard/index.html 2>/dev/null \
  | grep -v node_modules \
  | grep -v "\.test\." \
  | grep -v "\.snap" \
  > /tmp/octop-occurrences.txt
wc -l /tmp/octop-occurrences.txt
```

Review the file. Categorize each match as: title/brand (change), comment (don't change), test fixture (update if exposed), data attribute (case-by-case).

- [ ] **Step 2: Replace `<title>` and meta tags in `dashboard/src/index.html`**

Edit `dashboard/src/index.html`:

```html
<!-- Find: -->
<title>Octop</title>
<!-- Replace with: -->
<title>UniTop</title>

<!-- Find any: -->
<meta name="description" content="...Octop...">
<!-- Replace "Octop" with "UniTop" inside the content attribute only -->
```

- [ ] **Step 3: Bulk-replace UI strings in i18n locales**

```bash
cd /Volumes/MacMini/unitop
find dashboard/src/i18n/locales -name "*.json" -print0 \
  | xargs -0 sed -i '' 's/\bOctop\b/UniTop/g'
```

Then **manually inspect and revert** any translation key changes that look wrong (translators may have produced different word choices for "Octop" that shouldn't be globally substituted). Verify by spot-checking 3 files:

```bash
cd /Volumes/MacMini/unitop
grep -l "UniTop" dashboard/src/i18n/locales/*.json | head -3 | xargs head -30
```

- [ ] **Step 4: Update brand string in Sidebar / top-bar components**

```bash
cd /Volumes/MacMini/unitop
grep -rn '"Octop"\|>Octop<\|Octop<' dashboard/src/components/Layout/ \
  dashboard/src/components/TopBar/ 2>/dev/null
```

For each match, change the displayed string to `"UniTop"`. Do NOT change `className`, `id`, `data-testid`, file names, or import paths.

- [ ] **Step 5: Swap dashboard brand assets**

```bash
cd /Volumes/MacMini/unitop
git rm dashboard/public/apple-touch-icon.png 2>/dev/null
cp unitop-assets/favicon.ico dashboard/public/apple-touch-icon.png
# Also swap any logo in dashboard/src/assets/app/
ls dashboard/src/assets/app/ 2>/dev/null
git rm dashboard/src/assets/app/logo.svg dashboard/src/assets/app/logo.png 2>/dev/null
cp unitop-assets/dashboard-logo.svg dashboard/src/assets/app/logo.svg
cp unitop-assets/dashboard-logo.png dashboard/src/assets/app/logo.png 2>/dev/null || true
git add dashboard/public/ dashboard/src/assets/app/
```

- [ ] **Step 6: Run dashboard lint/typecheck**

```bash
cd /Volumes/MacMini/unitop/dashboard
npm run lint 2>&1 | tail -30
npm run typecheck 2>&1 | tail -30
```

Expected: no new errors. Pre-existing errors (before this commit) are tolerated.

- [ ] **Step 7: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -rn '"Octop"' dashboard/src/ --include="*.ts" --include="*.tsx" \
  | grep -v node_modules | grep -v "\.test\."
```

Expected: no output (UI string literals are UniTop).

- [ ] **Step 8: Commit**

```bash
cd /Volumes/MacMini/unitop
git add dashboard/
git commit -m "feat(dashboard): rebrand UI to UniTop + swap app icons"
git push origin unitop/branding
```

---

### Task 4: Rebrand desktop build configs (Info.plist, NSIS, Taskfile, dmg bg, appicon)

**Files:**
- Modify: `desktop/src/build/darwin/Info.plist`, `Info.dev.plist`
- Modify: `desktop/src/build/windows/nsis/project.nsi`, `wails_tools.nsh`
- Modify: `desktop/src/build/darwin/Taskfile.yml`, `windows/Taskfile.yml`, `linux/Taskfile.yml`
- Modify: `desktop/src/build/darwin/dmg-background.jpeg`
- Modify: `desktop/src/build/appicon.png`, `appicon-macos.png`, `desktop/src/assets/tray-icon*.png`, `octop-mascot-*.webp`
- Modify: `desktop/src/build/config.yml`, `desktop/src/build/windows/info.json`

- [ ] **Step 1: Replace identifiers in `desktop/src/build/darwin/Info.plist`**

Edit `desktop/src/build/darwin/Info.plist`:

```xml
<!-- Find and replace: -->
<key>CFBundleName</key><string>Octop</string>
<!-- → -->
<key>CFBundleName</key><string>UniTop</string>

<key>CFBundleExecutable</key><string>Octop</string>
<!-- → -->
<key>CFBundleExecutable</key><string>UniTop</string>

<key>CFBundleIdentifier</key><string>com.tencent.octop</string>
<!-- → -->
<key>CFBundleIdentifier</key><string>com.unitop.desktop</string>

<key>CFBundleGetInfoString</key><string>Octop desktop shell</string>
<!-- → -->
<key>CFBundleGetInfoString</key><string>UniTop desktop shell</string>
```

Edit `Info.dev.plist` with the same substitutions.

- [ ] **Step 2: Replace display name in NSIS project.nsi**

Edit `desktop/src/build/windows/nsis/project.nsi`:

```nsi
; Find:
# Octop desktop NSIS installer.
; Replace with:
# UniTop desktop NSIS installer.

; Find in build-command comments:
makensis -DARG_WAILS_AMD64_BINARY=..\..\..\bin\Octop.exe project.nsi
makensis -DARG_WAILS_ARM64_BINARY=..\..\..\bin\Octop.exe project.nsi
; Replace `Octop.exe` → `UniTop.exe` in both lines.
```

The display name string flows through `${INFO_PRODUCTNAME}` from `windows/info.json` (next step) — verify with grep.

- [ ] **Step 3: Update `desktop/src/build/windows/info.json`**

Edit `desktop/src/build/windows/info.json`:

```json
{
  "name": "UniTop",
  "productName": "UniTop",
  "companyName": "759012193",
  "productVersion": "<inherit from upstream VERSION>",
  "copyright": "Licensed under MIT — forked from TencentCloud/Octop"
}
```

(Keep `"productVersion"` matching the upstream wails build, or remove if upstream omits.)

- [ ] **Step 4: Update `desktop/src/build/config.yml`**

Edit `desktop/src/build/config.yml`:

```yaml
# Find any occurrence of:
name: Octop
# Replace with:
name: UniTop

# Find any:
output: Octop-
# Replace with:
output: UniTop-
```

- [ ] **Step 5: Rename artifact prefixes in `Taskfile.yml` files**

Edit `desktop/src/build/darwin/Taskfile.yml`:

```yaml
# Find:
{{.BIN_DIR}}/{{.APP_NAME}}-desktop-darwin-{{.ARCH}}-{{.VERSION}}.dmg
# Replace:
{{.BIN_DIR}}/{{.APP_NAME}}-desktop-darwin-{{.ARCH}}-{{.VERSION}}.dmg
# (no change — APP_NAME controls the prefix; rename APP_NAME value at top of file)

# Find at top of file:
APP_NAME: Octop
# Replace with:
APP_NAME: UniTop

# Find:
rm -f {{.ARCHIVE}} {{.BIN_DIR}}/{{.APP_NAME}}-Desktop-darwin-{{.ARCH}}.zip ...
# (No string replace needed — uses {{.APP_NAME}} interpolation.)

# Find:
../portable/release/Octop-portable-darwin-%s-%s.zip
# Replace:
../portable/release/UniTop-portable-darwin-%s-%s.zip

# Find:
cp {{.PORTABLE_ZIP}} {{.BIN_DIR}}/{{.APP_NAME}}.app/Contents/Resources/Octop-darwin-{{.ARCH}}.zip
# Replace:
cp {{.PORTABLE_ZIP}} {{.BIN_DIR}}/{{.APP_NAME}}.app/Contents/Resources/UniTop-darwin-{{.ARCH}}.zip
```

Apply the analogous `APP_NAME: Octop → UniTop` change in `windows/Taskfile.yml` and `linux/Taskfile.yml`.

- [ ] **Step 6: Swap desktop brand assets**

```bash
cd /Volumes/MacMini/unitop
git rm desktop/src/build/appicon.png \
        desktop/src/build/appicon-macos.png \
        desktop/src/assets/tray-icon.png \
        desktop/src/assets/tray-icon-template.png \
        desktop/src/assets/octop-mascot-peek.webp \
        desktop/src/assets/octop-mascot-type.webp \
        desktop/src/build/darwin/dmg-background.jpeg 2>/dev/null
cp unitop-assets/desktop-icon.png desktop/src/build/appicon.png
cp unitop-assets/desktop-icon-macos.png desktop/src/build/appicon-macos.png
cp unitop-assets/tray-icon.png desktop/src/assets/tray-icon.png
cp unitop-assets/tray-icon-template.png desktop/src/assets/tray-icon-template.png
cp unitop-assets/mascot-peek.webp desktop/src/assets/octop-mascot-peek.webp
cp unitop-assets/mascot-type.webp desktop/src/assets/octop-mascot-type.webp
cp unitop-assets/dmg-background.jpeg desktop/src/build/darwin/dmg-background.jpeg
git add desktop/src/
```

- [ ] **Step 7: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -rn "Octop" desktop/src/build/ desktop/src/assets/ \
  | grep -v "\.go:" | grep -v "node_modules"
```

Expected: empty (no `Octop` strings left in build configs/assets, except Go source filenames like `desktop/src/desktop_copy.go` which don't contain Octop).

```bash
plutil -lint desktop/src/build/darwin/Info.plist
plutil -lint desktop/src/build/darwin/Info.dev.plist
```

Expected: `OK`.

- [ ] **Step 8: Commit**

```bash
cd /Volumes/MacMini/unitop
git add desktop/src/
git commit -m "feat(desktop): rename build configs to UniTop + swap icons/dmg background"
git push origin unitop/branding
```

---

### Task 5: Rebrand FnOS NAS package

**Files:**
- Modify: `fnos/docker/manifest`, `fnos/docker/ICON.PNG`, `fnos/docker/ICON_256.PNG`
- Modify: `fnos/native/manifest` (or analogous), `fnos/native/ICON.PNG`, `fnos/native/ICON_256.PNG`
- Modify: `fnos/docker/app/ui/images/icon_256.png`, `icon_64.png` (same for native)
- Modify: `scripts/build-fpk.sh`

- [ ] **Step 1: Survey FnOS branding**

```bash
cd /Volumes/MacMini/unitop
grep -rn "Octop\|octop" fnos/ scripts/build-fpk.sh \
  | grep -v "LICENSE" | grep -v "\.gitkeep"
```

Review each match. Categorize: package name (change), icon path (swap file), references in shell scripts (change display strings only).

- [ ] **Step 2: Update FnOS manifests**

Edit `fnos/docker/manifest`:

```ini
# Find:
name=Octop
# Replace with:
name=UniTop

# Find:
display_name=Octop
# Replace with:
display_name=UniTop

# Find:
app_id=com.tencent.octop
# Replace with:
app_id=com.unitop.desktop
```

Apply the analogous changes to `fnos/native/manifest` (or whatever manifest file exists).

- [ ] **Step 3: Update `scripts/build-fpk.sh`**

Edit `scripts/build-fpk.sh`:

```bash
# Find any reference to:
Octop-*.fpk
# Replace:
UniTop-*.fpk

# Find:
APP_NAME="Octop"
# Replace:
APP_NAME="UniTop"
```

- [ ] **Step 4: Swap FnOS icons**

```bash
cd /Volumes/MacMini/unitop
git rm fnos/docker/ICON.PNG fnos/docker/ICON_256.PNG \
        fnos/native/ICON.PNG fnos/native/ICON_256.PNG \
        fnos/docker/app/ui/images/icon_256.png \
        fnos/docker/app/ui/images/icon_64.png \
        fnos/native/app/ui/images/icon_256.png \
        fnos/native/app/ui/images/icon_64.png 2>/dev/null
cp unitop-assets/fnos-icon.png fnos/docker/ICON.PNG
cp unitop-assets/fnos-icon-256.png fnos/docker/ICON_256.PNG
cp unitop-assets/fnos-icon.png fnos/native/ICON.PNG
cp unitop-assets/fnos-icon-256.png fnos/native/ICON_256.PNG
cp unitop-assets/fnos-icon-256.png fnos/docker/app/ui/images/icon_256.png
cp unitop-assets/fnos-icon-64.png fnos/docker/app/ui/images/icon_64.png
cp unitop-assets/fnos-icon-256.png fnos/native/app/ui/images/icon_256.png
cp unitop-assets/fnos-icon-64.png fnos/native/app/ui/images/icon_64.png
git add fnos/
```

- [ ] **Step 5: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -rn "Octop" fnos/manifest fnos/docker/manifest fnos/native/manifest scripts/build-fpk.sh 2>/dev/null
```

Expected: empty.

- [ ] **Step 6: Commit**

```bash
cd /Volumes/MacMini/unitop
git add fnos/ scripts/build-fpk.sh
git commit -m "feat(fnos): rebrand NAS package to UniTop + swap icons"
git push origin unitop/branding
```

---

## Phase 3: Tencent Cloud strip

### Task 6: Remove WeCom QR + Discord from README (Tencent active promotion)

**Files:**
- Modify: `README.md`, `README_CN.md`
- Delete: `docs/assets/qrcode.png`

- [ ] **Step 1: Locate the WeCom QR section and Discord link**

```bash
cd /Volumes/MacMini/unitop
grep -n "WeCom\|wecom\|qrcode\|discord" README.md README_CN.md
```

You should see:
- A section titled `## 💬 WeCom Customer Group (CN)` (or similar)
- An image reference to `docs/assets/qrcode.png`
- A Discord URL: `https://discord.gg/jPas5J8Ua`

- [ ] **Step 2: Delete the WeCom QR section**

In `README.md`, find the entire `## 💬 WeCom Customer Group (CN)` section (from the heading to the next `##` or EOF) and delete it.

Repeat for `README_CN.md`.

- [ ] **Step 3: Delete the qrcode.png file**

```bash
cd /Volumes/MacMini/unitop
git rm docs/assets/qrcode.png
```

- [ ] **Step 4: Decide Discord link disposition**

Per spec §6 Q4 default: **keep** the Discord link (user can override). If keeping:

```bash
cd /Volumes/MacMini/unitop
# Verify Discord link is still present and well-formed:
grep -n "discord.gg" README.md
```

Expected: at least one hit. No change needed.

If user has overridden to remove: delete the Discord line(s) from both `README.md` and `README_CN.md`.

- [ ] **Step 5: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -n "WeCom\|wecom\|qrcode\|myqcloud" README.md README_CN.md
```

Expected: empty (no WeCom references; no COS references).

```bash
git status --porcelain | grep qrcode.png
```

Expected: no output (file deleted).

- [ ] **Step 6: Commit**

```bash
cd /Volumes/MacMini/unitop
git add README.md README_CN.md
git rm docs/assets/qrcode.png 2>/dev/null
git commit -m "docs: remove Tencent WeCom customer QR + section from README"
git push origin unitop/branding
```

---

### Task 7: Default-disable Tencent connector bundle

**Files:**
- Modify: `dashboard/src/api/modules/connectors.ts`
- Modify: `dashboard/src/pages/Agent/Connectors/connectorDefs.tsx`
- Possibly: `dashboard/src/pages/Agent/Connectors/useConnectors.ts` (if bundle initialization happens there)

**Interfaces:**
- Consumes: connector ID constants from upstream (Tencent connector IDs preserved — only their `defaultEnabled` flag changes)
- Produces: dashboard where Tencent suite connectors are not auto-enabled on first user login

- [ ] **Step 1: Locate the default-enabled connector logic**

```bash
cd /Volumes/MacMini/unitop
grep -rn "tencent-" dashboard/src/ --include="*.ts" --include="*.tsx" | head -30
grep -rn "defaultEnabled\|default_enabled\|DEFAULT_ENABLED" dashboard/src/ \
  --include="*.ts" --include="*.tsx" | head -30
```

Note all connector IDs starting with `tencent-` and where their enabled-by-default flag is set.

- [ ] **Step 2: Identify the Tencent bundle**

```bash
cd /Volumes/MacMini/unitop
grep -rn "tencent-suite\|TencentSuite\|tencent_bundle\|Tencent.*bundle" dashboard/src/ \
  --include="*.ts" --include="*.tsx" | head -20
```

If there's a `tencent-suite` bundle definition, note its location.

- [ ] **Step 3: Disable Tencent connectors by default**

In `dashboard/src/pages/Agent/Connectors/connectorDefs.tsx` (or wherever Tencent connector definitions live), for each `tencent-*` connector entry, change `defaultEnabled: true` to `defaultEnabled: false`. If a `defaultOpenConnectors` array exists, remove all `tencent-*` IDs from it.

Specific pattern to look for (adjust per actual file content):

```typescript
// Find:
{
  id: 'tencent-docs',
  ...
  defaultEnabled: true,
},
// Replace with:
{
  id: 'tencent-docs',
  ...
  defaultEnabled: false,
},
```

- [ ] **Step 4: Remove Tencent suite from default bundle**

If a default connector bundle list exists (e.g., `DEFAULT_BUNDLE`), remove `tencent-suite` from it. Find:

```bash
cd /Volumes/MacMini/unitop
grep -rn "tencent-suite\|tencent_suite" dashboard/src/ --include="*.ts" --include="*.tsx"
```

For each occurrence in a default/bundle context, comment it out or replace with `[]`.

- [ ] **Step 5: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -rn "defaultEnabled: true" dashboard/src/ --include="*.ts" --include="*.tsx" \
  | grep -i tencent
```

Expected: empty (no Tencent connector defaults to enabled).

- [ ] **Step 6: Run dashboard typecheck**

```bash
cd /Volumes/MacMini/unitop/dashboard
npm run typecheck 2>&1 | tail -30
```

Expected: no new errors.

- [ ] **Step 7: Commit**

```bash
cd /Volumes/MacMini/unitop
git add dashboard/src/
git commit -m "feat(dashboard): disable Tencent connector defaults"
git push origin unitop/branding
```

---

## Phase 4: Distribution redirects

### Task 8: Redirect README install URLs from COS to UniTop releases

**Files:**
- Modify: `README.md`, `README_CN.md`

- [ ] **Step 1: Locate all COS install URLs in README**

```bash
cd /Volumes/MacMini/unitop
grep -n "finnie-1258344699.cos.ap-guangzhou.myqcloud.com" README.md README_CN.md
```

You should see 4 occurrences per file (macOS/Linux curl, Windows irm, Windows .bat, extras variant).

- [ ] **Step 2: Replace COS URLs in README.md**

For each line containing `https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.{sh,ps1,bat}`, replace with the corresponding GitHub Releases URL:

| Original | Replace with |
|----------|--------------|
| `https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.sh` | `https://github.com/759012193/UniTop/releases/latest/download/install.sh` |
| `https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.ps1` | `https://github.com/759012193/UniTop/releases/latest/download/install.ps1` |
| `https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.bat` | `https://github.com/759012193/UniTop/releases/latest/download/install.bat` |

Use `sed`:

```bash
cd /Volumes/MacMini/unitop
sed -i '' \
  -e 's|https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.sh|https://github.com/759012193/UniTop/releases/latest/download/install.sh|g' \
  -e 's|https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.ps1|https://github.com/759012193/UniTop/releases/latest/download/install.ps1|g' \
  -e 's|https://finnie-1258344699.cos.ap-guangzhou.myqcloud.com/octop/install.bat|https://github.com/759012193/UniTop/releases/latest/download/install.bat|g' \
  README.md README_CN.md
```

- [ ] **Step 3: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -n "finnie\|myqcloud" README.md README_CN.md
```

Expected: empty.

```bash
grep -nE "759012193/UniTop/releases/latest/download/install" README.md README_CN.md | wc -l
```

Expected: ≥ 6 (3 URLs × 2 files).

- [ ] **Step 4: Confirm scripts themselves are untouched**

```bash
cd /Volumes/MacMini/unitop
diff <(git show upstream/main:scripts/install.sh) scripts/install.sh
diff <(git show upstream/main:scripts/install.ps1) scripts/install.ps1
diff <(git show upstream/main:scripts/install.bat) scripts/install.bat
```

Expected: no output for each.

- [ ] **Step 5: Commit**

```bash
cd /Volumes/MacMini/unitop
git add README.md README_CN.md
git commit -m "docs: redirect install URLs from Tencent COS to UniTop GitHub Releases"
git push origin unitop/branding
```

---

## Phase 5: CI / Release pipeline

### Task 9: Retarget all workflows + Makefile to 759012193/UniTop

**Files:**
- Modify: `.github/workflows/release.yml`
- Modify: `.github/workflows/octop-desktop.yml`
- Modify: `.github/workflows/docker-publish.yml`
- Modify: `.github/workflows/fnos-build-fpk.yml`
- Modify: `.github/workflows/ci.yml` (badges only)
- Modify: `Makefile`

- [ ] **Step 1: Survey all `TencentCloud/Octop` references**

```bash
cd /Volumes/MacMini/unitop
grep -rn "TencentCloud/Octop\|tencentcloud/octop" .github/ Makefile 2>/dev/null
```

- [ ] **Step 2: Replace repo references in workflows**

In each file from the survey, replace `TencentCloud/Octop` → `759012193/UniTop` (case-sensitive):

```bash
cd /Volumes/MacMini/unitop
sed -i '' 's|TencentCloud/Octop|759012193/UniTop|g' \
  .github/workflows/release.yml \
  .github/workflows/octop-desktop.yml \
  .github/workflows/docker-publish.yml \
  .github/workflows/fnos-build-fpk.yml \
  .github/workflows/ci.yml \
  Makefile
```

- [ ] **Step 3: Replace Docker image registry paths**

In `.github/workflows/docker-publish.yml`, find the GHCR image path:

```yaml
# Find:
image: ghcr.io/tencentcloud/octop
# OR:
images: ghcr.io/tencentcloud/octop
# Replace with:
image: ghcr.io/759012193/unitop
```

Also find Docker Hub path:

```yaml
# Find:
$DOCKERHUB_USERNAME/octop
# (Default per spec: comment out the Docker Hub login + tag-push section, since Q2 default = no Docker Hub account)
```

**Apply the spec default — disable Docker Hub:**

Find the `docker/login-action` step that uses `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN` and comment it out (or wrap in `if: false`). Also comment out the `tags:` entry that pushes to `${{ secrets.DOCKERHUB_USERNAME }}/octop`.

```yaml
# Original:
- name: Log in to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

# Replace with:
# Disabled: UniTop does not publish to Docker Hub (Q2 default).
# - name: Log in to Docker Hub
#   uses: docker/login-action@v3
#   with:
#     username: ${{ secrets.DOCKERHUB_USERNAME }}
#     password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Apply the analogous comment-out to the `tags:` line that references Docker Hub.

- [ ] **Step 4: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -rn "TencentCloud/Octop\|tencentcloud/octop" .github/ Makefile
```

Expected: empty.

```bash
grep -n "759012193/UniTop\|759012193/unitop" .github/workflows/release.yml
```

Expected: at least one hit.

- [ ] **Step 5: Validate YAML syntax**

```bash
cd /Volumes/MacMini/unitop
for f in .github/workflows/*.yml; do
  python3 -c "import yaml; yaml.safe_load(open('$f'))" && echo "OK: $f" || echo "FAIL: $f"
done
```

Expected: `OK:` for each file.

- [ ] **Step 6: Commit**

```bash
cd /Volumes/MacMini/unitop
git add .github/ Makefile
git commit -m "ci: retarget all workflows + Makefile to 759012193/UniTop"
git push origin unitop/branding
```

---

### Task 10: Disable PyPI publish job in release.yml

**Files:**
- Modify: `.github/workflows/release.yml` (the `publish` job)

- [ ] **Step 1: Locate the PyPI publish job**

```bash
cd /Volumes/MacMini/unitop
grep -n "pypi\|PyPI\|gh-action-pypi-publish" .github/workflows/release.yml
```

You should see a job with `uses: pypa/gh-action-pypi-publish@release/v1`.

- [ ] **Step 2: Disable the publish job**

Find the publish job definition and prepend `if: false` to its top-level `if:` or add one if missing:

```yaml
# Find:
  publish:
    name: Publish to PyPI
    runs-on: ubuntu-latest
    environment: pypi
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist
      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          password: ${{ secrets.PYPI_API_TOKEN }}

# Replace by adding `if: false` after the job name:
  publish:
    if: false  # Disabled: UniTop does not publish to PyPI — `octop` is owned by TencentCloud on PyPI
    name: Publish to PyPI (disabled)
    runs-on: ubuntu-latest
    environment: pypi
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist
      - name: Publish to PyPI (no-op)
        run: echo "PyPI publish disabled in UniTop fork"
```

- [ ] **Step 3: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -B1 -A5 "publish:" .github/workflows/release.yml | head -20
```

Expected: see `if: false` on or near the publish job.

- [ ] **Step 4: Validate YAML**

```bash
cd /Volumes/MacMini/unitop
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/release.yml'))" && echo OK
```

Expected: `OK`.

- [ ] **Step 5: Commit**

```bash
cd /Volumes/MacMini/unitop
git add .github/workflows/release.yml
git commit -m "ci: disable PyPI publish job (octop package owned by TencentCloud)"
git push origin unitop/branding
```

---

### Task 11: Add macOS codesign + notarization steps to octop-desktop.yml

**Files:**
- Modify: `.github/workflows/octop-desktop.yml`

**Interfaces:**
- Consumes: secrets `APPLE_TEAM_ID`, `APPLE_KEY_ID`, `APPLE_ISSUER_ID`, `APPLE_API_KEY_P8`, `APPLE_CERT_P12`, `APPLE_CERT_PASSWORD`, `KEYCHAIN_PASSWORD` (all `<TODO>` until user provisions)
- Produces: signed + notarized `UniTop-desktop-darwin-{arch}.dmg` artifacts

- [ ] **Step 1: Locate the `darwin-*` jobs**

```bash
cd /Volumes/MacMini/unitop
grep -n "darwin-arm64:\|darwin-amd64:\|runs-on: macos" .github/workflows/octop-desktop.yml
```

- [ ] **Step 2: Add the three signing steps before the upload-artifact step**

For each `darwin-*` job, insert these steps immediately before `actions/upload-artifact@v4`:

```yaml
      - name: Import signing certificate
        env:
          P12: ${{ secrets.APPLE_CERT_P12 }}
          P12_PASSWORD: ${{ secrets.APPLE_CERT_PASSWORD }}
          KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
        run: |
            KEYCHAIN=build.keychain-db
            security create-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN
            security set-keychain-settings -lut 21600 $KEYCHAIN
            security unlock-keychain -p "$KEYCHAIN_PASSWORD" $KEYCHAIN
            echo "$P12" | base64 --decode > /tmp/cert.p12
            security import /tmp/cert.p12 -k $KEYCHAIN -P "$P12_PASSWORD" -T /usr/bin/codesign
            security set-key-partition-list -S apple-tool:,apple: -s -k "$KEYCHAIN_PASSWORD" $KEYCHAIN

      - name: Codesign .app
        run: |
            # Replace <TODO: cert CN> with the cert's Common Name (e.g. "Your Name")
            SIGN_IDENTITY="Developer ID Application: <TODO: cert CN> (${{ secrets.APPLE_TEAM_ID }})"
            codesign --deep --force --options runtime --timestamp \
                     --sign "$SIGN_IDENTITY" \
                     UniTop.app

      - name: Notarize .dmg
        env:
          APPLE_KEY_ID: ${{ secrets.APPLE_KEY_ID }}
          APPLE_ISSUER_ID: ${{ secrets.APPLE_ISSUER_ID }}
          APPLE_API_KEY_P8: ${{ secrets.APPLE_API_KEY_P8 }}
        run: |
            echo "$APPLE_API_KEY_P8" | base64 --decode > /tmp/AuthKey_$APPLE_KEY_ID.p8
            xcrun notarytool submit UniTop.dmg \
              --key /tmp/AuthKey_$APPLE_KEY_ID.p8 \
              --key-id "$APPLE_KEY_ID" --issuer "$APPLE_ISSUER_ID" \
              --wait
            xcrun stapler staple UniTop.dmg
```

Note `<TODO>` placeholders for cert CN — engineer must replace after user provisions the cert.

- [ ] **Step 3: Verify YAML**

```bash
cd /Volumes/MacMini/unitop
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/octop-desktop.yml'))" && echo OK
```

Expected: `OK`.

- [ ] **Step 4: Commit**

```bash
cd /Volumes/MacMini/unitop
git add .github/workflows/octop-desktop.yml
git commit -m "ci(desktop): add macOS codesign + notarytool steps for darwin jobs"
git push origin unitop/branding
```

---

### Task 12: Add Windows Azure Trusted Signing steps to octop-desktop.yml

**Files:**
- Modify: `.github/workflows/octop-desktop.yml`

**Interfaces:**
- Consumes: secrets `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_SUBSCRIPTION_ID`, `AZURE_DLIB_PATH`
- Produces: signed `UniTop-desktop-windows-{arch}.exe` artifacts

- [ ] **Step 1: Locate the `windows-*` jobs**

```bash
cd /Volumes/MacMini/unitop
grep -n "windows-amd64:\|windows-arm64:\|runs-on: windows" .github/workflows/octop-desktop.yml
```

- [ ] **Step 2: Add Azure login + signing steps before upload-artifact**

For each `windows-*` job, insert immediately before `actions/upload-artifact@v4`:

```yaml
      - name: Azure login
        uses: azure/login@v2
        with:
          creds: '{"clientId":"${{ secrets.AZURE_CLIENT_ID }}","clientSecret":"${{ secrets.AZURE_CLIENT_SECRET }}","subscriptionId":"${{ secrets.AZURE_SUBSCRIPTION_ID }}","tenantId":"${{ secrets.AZURE_TENANT_ID }}"}'

      - name: Sign .exe
        uses: azure/trusted-signing-action@v0
        with:
          azure-signing-path: ${{ secrets.AZURE_DLIB_PATH }}
          endpoint: https://eus.codesigning.azure.net/
          files:
            - UniTop.exe
          file-digest: SHA256
          timestamp-digest: SHA256
          timestamp-rfc3161: http://timestamp.digicert.com
```

- [ ] **Step 3: Verify YAML**

```bash
cd /Volumes/MacMini/unitop
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/octop-desktop.yml'))" && echo OK
```

Expected: `OK`.

- [ ] **Step 4: Commit**

```bash
cd /Volumes/MacMini/unitop
git add .github/workflows/octop-desktop.yml
git commit -m "ci(desktop): add Windows Azure Trusted Signing steps"
git push origin unitop/branding
```

---

### Task 13: Rename release artifacts Octop-* → UniTop-* in octop-desktop.yml

**Files:**
- Modify: `.github/workflows/octop-desktop.yml`

- [ ] **Step 1: Find all `Octop-` artifact prefixes**

```bash
cd /Volumes/MacMini/unitop
grep -n "Octop-" .github/workflows/octop-desktop.yml
```

You should see references like:
- `Octop-portable-{plat}-{VER}.zip`
- `Octop-desktop-{plat}-{VER}.{ext}`
- `Octop-Dekstop-darwin-{arch}.dmg` (upstream typo — also fix capitalization)
- `Octop-portable-darwin-%s-%s.zip`

- [ ] **Step 2: Replace `Octop-` → `UniTop-`**

```bash
cd /Volumes/MacMini/unitop
sed -i '' 's|Octop-|UniTop-|g' .github/workflows/octop-desktop.yml
```

- [ ] **Step 3: Fix upstream capitalization typo (`Dekstop` → `Desktop`)**

```bash
cd /Volumes/MacMini/unitop
grep -n "Dekstop\|dektop\|UniTop-Dekstop" .github/workflows/octop-desktop.yml
sed -i '' 's|UniTop-Dekstop|UniTop-Desktop|g' .github/workflows/octop-desktop.yml
```

- [ ] **Step 4: Verify**

```bash
cd /Volumes/MacMini/unitop
grep -n "Octop-\|Dekstop" .github/workflows/octop-desktop.yml
```

Expected: empty.

```bash
grep -nE "UniTop-(portable|desktop)-" .github/workflows/octop-desktop.yml | head -10
```

Expected: at least 4 hits showing `UniTop-portable-*` and `UniTop-desktop-*`.

- [ ] **Step 5: Validate YAML**

```bash
cd /Volumes/MacMini/unitop
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/octop-desktop.yml'))" && echo OK
```

Expected: `OK`.

- [ ] **Step 6: Commit**

```bash
cd /Volumes/MacMini/unitop
git add .github/workflows/octop-desktop.yml
git commit -m "ci(desktop): rename release artifacts Octop-* → UniTop-*"
git push origin unitop/branding
```

---

## Phase 6: Verification

### Task 14: Local sanity — Docker build + Python wheel

**Files:** none (read-only verification)

- [ ] **Step 1: Build the Python wheel locally**

```bash
cd /Volumes/MacMini/unitop
make install 2>&1 | tail -20
make build 2>&1 | tail -20
ls -la dist/
```

Expected: `dist/octop-*.whl` exists (note: wheel name stays `octop` per minimal-change constraint).

- [ ] **Step 2: Smoke-install the wheel**

```bash
cd /Volumes/MacMini/unitop
python3 -m venv /tmp/unitop-venv
/tmp/unitop-venv/bin/pip install dist/octop-*.whl --force-reinstall 2>&1 | tail -10
/tmp/unitop-venv/bin/octop --version
```

Expected: `1.0.0` (or current upstream version). No import errors.

- [ ] **Step 3: Build the Docker image locally**

```bash
cd /Volumes/MacMini/unitop
docker build -f docker/Dockerfile -t unitop-test:local . 2>&1 | tail -20
```

Expected: image builds successfully. Final line should mention the image tag.

- [ ] **Step 4: Run the Docker image briefly**

```bash
cd /Volumes/MacMini/unitop
docker run --rm -p 18080:80 unitop-test:local &
DOCKER_PID=$!
sleep 30
curl -s http://localhost:18080/ -o /tmp/dash.html
echo "--- HTML head ---"
head -50 /tmp/dash.html
kill $DOCKER_PID 2>/dev/null
```

Expected: response HTML contains `<title>UniTop</title>` (from Task 3 changes).

- [ ] **Step 5: Document results**

Note any failures. If the wheel name was accidentally changed, or the dashboard title didn't update, **fix and amend** the responsible commit before proceeding.

---

### Task 15: Push test tag v0.0.1-unitop-test1, verify all 6 platforms + signed binaries open

**Files:** none (read-only verification)

**Prerequisites:** User has provisioned:
- GitHub Secrets: `APPLE_TEAM_ID`, `APPLE_KEY_ID`, `APPLE_ISSUER_ID`, `APPLE_API_KEY_P8`, `APPLE_CERT_P12`, `APPLE_CERT_PASSWORD`, `KEYCHAIN_PASSWORD`
- GitHub Secrets: `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_SUBSCRIPTION_ID`, `AZURE_DLIB_PATH`
- User has replaced `<TODO: cert CN>` placeholder in `.github/workflows/octop-desktop.yml` with the actual cert CN
- User has uploaded `install.sh`, `install.ps1`, `install.bat` as release assets (manual upload — or engineer adds them to release.yml — see Step 7)

- [ ] **Step 1: Push the test tag**

```bash
cd /Volumes/MacMini/unitop
git tag v0.0.1-unitop-test1
git push origin v0.0.1-unitop-test1
```

- [ ] **Step 2: Watch the GH Actions run**

```bash
gh run watch --exit-status=1
```

Expected: at least 4 jobs succeed (frontend, darwin-arm64, darwin-amd64, windows-amd64, windows-arm64, linux-amd64, linux-arm64 — all 6 platforms). The `release` job uploads to a GitHub Release. If any signing step fails due to missing/wrong secret, surface the GH Actions log URL to user.

- [ ] **Step 3: List release artifacts**

```bash
gh release view v0.0.1-unitop-test1 --json assets --jq '.assets[].name'
```

Expected output contains:
- `UniTop-desktop-darwin-arm64-0.0.1-unitop-test1.dmg`
- `UniTop-desktop-darwin-amd64-0.0.1-unitop-test1.dmg`
- `UniTop-desktop-windows-amd64-0.0.1-unitop-test1.exe`
- `UniTop-desktop-windows-arm64-0.0.1-unitop-test1.exe`
- `UniTop-desktop-linux-amd64-0.0.1-unitop-test1.AppImage` (or `.deb`)
- `UniTop-desktop-linux-arm64-0.0.1-unitop-test1.AppImage`
- `UniTop-portable-{plat}-{ver}.zip` × 6
- `install.sh`, `install.ps1`, `install.bat` (if uploaded)

- [ ] **Step 4: Verify macOS signature**

Download `UniTop-desktop-darwin-arm64-0.0.1-unitop-test1.dmg` (engineer or user does this on a macOS machine):

```bash
codesign -dv --verbose=4 UniTop-desktop-darwin-arm64-0.0.1-unitop-test1.dmg 2>&1 | head -20
spctl -a -t install -v UniTop-desktop-darwin-arm64-0.0.1-unitop-test1.dmg
stapler validate UniTop-desktop-darwin-arm64-0.0.1-unitop-test1.dmg
```

Expected:
- `Authority=Developer ID Application: <user's name> (<TEAMID>)`
- `spctl`: `accepted`
- `stapler validate`: `The validate action worked!`

- [ ] **Step 5: Verify Windows signature**

Download `UniTop-desktop-windows-amd64-0.0.1-unitop-test1.exe` on Windows:

```powershell
signtool verify /pa UniTop-desktop-windows-amd64-0.0.1-unitop-test1.exe
Get-AuthenticodeSignature UniTop-desktop-windows-amd64-0.0.1-unitop-test1.exe
```

Expected:
- `signtool`: `Successfully verified`
- `Get-AuthenticodeSignature.Status`: `Valid`

- [ ] **Step 6: Install + run the macOS .dmg manually**

Open the downloaded `.dmg`, drag `UniTop.app` to Applications, launch it. Verify:
- Window title bar shows "UniTop"
- App icon is UniTop logo
- App launches without "unidentified developer" warning
- Web dashboard (if reachable) shows UniTop branding

- [ ] **Step 7: Confirm release assets include install scripts**

```bash
curl -I -L https://github.com/759012193/UniTop/releases/download/v0.0.1-unitop-test1/install.sh
curl -I -L https://github.com/759012193/UniTop/releases/download/v0.0.1-unitop-test1/install.ps1
curl -I -L https://github.com/759012193/UniTop/releases/download/v0.0.1-unitop-test1/install.bat
```

Expected: HTTP 200 for each.

If any returns 404: manually upload the missing files via `gh release upload v0.0.1-unitop-test1 scripts/install.sh scripts/install.ps1 scripts/install.bat`.

- [ ] **Step 8: Smoke-test the README install one-liner**

```bash
# From a clean macOS box (or VM):
curl -fsSL https://github.com/759012193/UniTop/releases/latest/download/install.sh | bash
```

Expected: uv installed (if not present), `pip install octop[browser]` runs successfully, `octop` command available in shell. (No UniTop-specific behavior change — the script is unchanged from upstream.)

- [ ] **Step 9: Roll back the test tag if anything fails**

```bash
cd /Volumes/MacMini/unitop
# Delete the GitHub Release (preserves the tag — see note)
gh release delete v0.0.1-unitop-test1 --yes
# Tag itself cannot be deleted if it has been "released"; use a new tag for retry
git tag -d v0.0.1-unitop-test1
git push origin :refs/tags/v0.0.1-unitop-test1
```

Then fix the failing step, recommit, retag.

---

### Task 16: Verify FnOS .fpk dispatch

**Files:** none (read-only verification)

- [ ] **Step 1: Confirm `fnos-build-fpk.yml` triggered**

After Task 15's release:

```bash
gh run list --workflow=fnos-build-fpk.yml --limit=5
```

Expected: at least one successful run, triggered by the v0.0.1-unitop-test1 release.

- [ ] **Step 2: Inspect the .fpk artifact**

```bash
gh run download --name fnos-build  # or appropriate artifact name; check `gh run view <run-id> --json artifacts`
ls -la *.fpk 2>/dev/null
```

Expected: a `.fpk` file downloadable from the run's artifacts.

- [ ] **Step 3: (Optional) Test install on a FnOS VM**

If a FnOS VM is available: upload the `.fpk`, install via the FnOS App Center, verify the app launches with UniTop branding.

Skip if no FnOS VM available; the build-pipeline success is the gate.

---

### Task 17: Cut first real release v1.0.0-unitop.1

**Files:** none (read-only verification)

- [ ] **Step 1: Confirm open questions resolved**

Verify the 6 placeholders from spec §6 are filled:
- Q1 (Windows signing path) — Azure Trusted Signing configured (Tasks 11-12 done)
- Q2 (Docker Hub) — confirmed disabled (Task 9 Step 3 done)
- Q3 (Apple Team ID) — secret provisioned
- Q4 (Discord link) — kept or removed per user decision
- Q5 (brand assets) — `unitop-assets/` populated
- Q6 (`.dlib`) — uploaded to repo at `AZURE_DLIB_PATH`

- [ ] **Step 2: Final review**

```bash
cd /Volumes/MacMini/unitop
git log --oneline upstream/main..unitop/branding
```

Expected: 10 commits corresponding to the spec §4.4 list (in order):
1. `docs: rebrand README/CHANGELOG/AGENTS to UniTop + swap brand assets`
2. `feat(dashboard): rebrand UI to UniTop + swap app icons`
3. `feat(desktop): rename build configs to UniTop + swap icons/dmg background`
4. `feat(fnos): rebrand NAS package to UniTop + swap icons`
5. `docs: remove Tencent WeCom customer QR + section from README`
6. `feat(dashboard): disable Tencent connector defaults`
7. `docs: redirect install URLs from Tencent COS to UniTop GitHub Releases`
8. `ci: retarget all workflows + Makefile to 759012193/UniTop`
9. `ci: disable PyPI publish job (octop package owned by TencentCloud)`
10. `ci(desktop): add macOS codesign + notarytool steps for darwin jobs`
11. `ci(desktop): add Windows Azure Trusted Signing steps`
12. `ci(desktop): rename release artifacts Octop-* → UniTop-*`

(Note: actual commit count is 12 due to splitting CI into 4 commits for clearer review. Adjust if you've combined.)

If any commit message is missing, **amend it** before proceeding:

```bash
cd /Volumes/MacMini/unitop
git commit --amend -m "<correct message>"
git push --force-with-lease origin unitop/branding
```

- [ ] **Step 3: Merge `unitop/branding` → `main` (or open PR)**

Per repo policy: typically `unitop/branding` is merged into `main` via PR. Either:

```bash
cd /Volumes/MacMini/unitop
gh pr create --base main --head unitop/branding \
  --title "UniTop rebrand fork" \
  --body "Fork of TencentCloud/Octop with cosmetic rebrand + multi-platform signing. See docs/superpowers/specs/2026-09-17-unitop-rebrand-design.md for the full design rationale."
```

Or directly merge if the user prefers (this is a personal fork):

```bash
cd /Volumes/MacMini/unitop
git checkout main
git merge --no-ff unitop/branding -m "Merge unitop/branding: UniTop rebrand"
git push origin main
```

- [ ] **Step 4: Tag the first real release**

```bash
cd /Volumes/MacMini/unitop
git tag v1.0.0-unitop.1
git push origin v1.0.0-unitop.1
```

- [ ] **Step 5: Confirm release pipeline**

```bash
gh release view v1.0.0-unitop.1
gh run watch --exit-status=1
```

Expected: full release pipeline runs to completion; signed artifacts uploaded to GitHub Release; install scripts attached.

- [ ] **Step 6: Announce**

Write a short announcement (issue, discussion, or external channel — user's choice) summarizing the fork, the runtime-identifier preservation policy, and how to install (`pip install octop` for server, GitHub Releases for desktop).

---

## Self-Review (writer's checklist)

### Spec coverage

| Spec section | Implemented by |
|---|---|
| §0 Goals (1-6) | All addressed across Tasks 2-13 |
| §1.3 in-place edits | Every task does in-place edits (no overlay dirs) |
| §2.A user-facing brand | Tasks 2, 3 |
| §2.B desktop brand | Task 4 |
| §2.C FnOS brand | Task 5 |
| §2.D install URLs | Task 8 (README only; scripts unchanged) |
| §2.E Tencent strip | Tasks 6, 7 |
| §2.F metadata | Task 2 Step 9 (pyproject.toml desc/authors) |
| §2.G CI target | Task 9 |
| §2.H preserve runtime | Verified in every task via grep checks |
| §3.1 macOS signing | Task 11 |
| §3.2 Windows signing | Task 12 |
| §3.3 Docker Hub disabled | Task 9 Step 3 |
| §3.4 FnOS | Task 5 + 16 |
| §3.5 artifact rename | Task 13 |
| §3.6 update endpoint | Implicit in Task 9 (workflow auto-uses `${{ github.repository }}`) |
| §4 fork + branch | Task 1 |
| §4.4 10-commit structure | Tasks 2-13 produce 12 commits (CI split finer) |
| §5 verification | Tasks 14-17 |
| §6 open questions | Tracked throughout; user fills before Task 15 |

**Gap to fix:** pyproject.toml `description` / `authors` change (§2.F) not assigned to any task. Either add as Task 4.5 or include in Task 9. Easiest: add a single step in Task 9.

### Placeholder scan

Searched for: "TBD", "TODO", "implement later", "fill in details", "appropriate", "handle edge cases". All `<TODO>` markers are intentional (cert CN, secrets awaiting user provisioning). No bare placeholders.

### Type / name consistency

- `APP_NAME`, `APP_NAME: UniTop` used consistently across Taskfile.yml edits.
- Artifact prefixes `UniTop-portable-*`, `UniTop-desktop-*` used consistently in Tasks 4, 13, 15.
- Secret names: `APPLE_TEAM_ID`, `APPLE_KEY_ID`, etc. — defined once in Task 11, referenced consistently.
- Azure secrets: `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_SUBSCRIPTION_ID`, `AZURE_DLIB_PATH` — defined in Task 12.
- GitHub org / repo: `759012193/UniTop` everywhere.

### Ambiguity

- Task 15 Step 7 explicitly handles the "release assets missing" case with `gh release upload` fallback.
- Task 17 Step 2 admits the actual commit count is 12 (not 10 as in spec §4.4) and explains why.
- Task 11 `<TODO: cert CN>` is explicitly called out as engineer-replaceable after user provisions the cert.