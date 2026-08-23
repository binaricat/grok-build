# binaricat/grok-build

Personal fork of [xai-org/grok-build](https://github.com/xai-org/grok-build) with:

1. **`terminal` theme** — full TUI with transparent / terminal-native colors (`Color::Reset` backgrounds)
2. **`GH_RELEASE_REPO = "binaricat/grok-build"`** — `installer = "gh-release"` updates from this fork

## Layout

| Path | Role |
|------|------|
| `patches/terminal-theme.patch` | Code overlay re-applied on every upstream sync |
| `.github/workflows/sync-upstream.yml` | Every 6h: hard-sync `upstream/main`, apply patch, push |
| `.github/workflows/release.yml` | Build macOS aarch64 + Linux x86_64, publish GitHub Release |
| `.upstream-sha` | Last synced upstream commit (skip if unchanged) |

## Local install (macOS Apple Silicon)

```bash
# One-shot from latest release
VERSION=$(gh release list --repo binaricat/grok-build --limit 1 --json tagName --jq '.[0].tagName' | sed 's/^v//')
gh release download "v${VERSION}" --repo binaricat/grok-build \
  --pattern "grok-${VERSION}-macos-aarch64" -D ~/.grok/downloads --clobber
chmod +x ~/.grok/downloads/grok-${VERSION}-macos-aarch64
ln -sfn "../downloads/grok-${VERSION}-macos-aarch64" ~/.grok/bin/grok
ln -sfn "../downloads/grok-${VERSION}-macos-aarch64" ~/.grok/bin/agent
```

`~/.grok/config.toml`:

```toml
[cli]
installer = "gh-release"
auto_update = true

[ui]
theme = "terminal"
```

Then `grok update` / background auto-update uses this fork’s Releases.

## When the patch fails

Sync upstream fails on purpose (no auto conflict resolve). Fix:

1. Clone this repo, add upstream if needed:
   `git remote add upstream https://github.com/xai-org/grok-build.git`
2. `git fetch upstream && git reset --hard upstream/main`
3. `git apply --reject patches/terminal-theme.patch` → fix every `*.rej`
4. Regenerate the overlay:
   `git add -A && git diff --cached > patches/terminal-theme.patch`
5. Commit only the refreshed patch (and any intentional overlay edits), push,
   then re-run **Sync upstream** / **Release**

Note: scheduled sync **force-pushes** `main` onto `upstream/main` + overlay.
Do not put long-lived work only on `main` without keeping it in `patches/`.

## Manual triggers

```bash
gh workflow run sync-upstream.yml --repo binaricat/grok-build
gh workflow run release.yml --repo binaricat/grok-build
```
