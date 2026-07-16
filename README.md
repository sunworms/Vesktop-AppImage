# vesktop-appimage-builder

A tiny GitHub Actions workflow that builds a Linux **AppImage** from the
**latest commit** of [Vencord/Vesktop](https://github.com/Vencord/Vesktop)
and publishes it as a release in this repo, so you always have a fresh
build to download without waiting for an upstream tagged release.

## Setup

1. Create a new (empty) GitHub repository.
2. Copy `.github/workflows/build-vesktop-appimage.yml` from this folder into
   the same path in your new repo.
3. In your repo go to **Settings → Actions → General → Workflow permissions**
   and select **"Read and write permissions"** (required so the workflow can
   create a release).
4. Go to the **Actions** tab and manually run the "Build Vesktop AppImage"
   workflow once (`Run workflow` button) to trigger the first build.
   After that it also runs automatically every day at 06:00 UTC, so the
   AppImage stays up to date with upstream.

## What it does

- Checks out `Vencord/Vesktop` at its current `main` branch (i.e. whatever
  the latest commit is at run time).
- Installs Node.js 20 + pnpm (via Corepack, using the version pinned in
  Vesktop's `package.json`).
- Runs `pnpm install` and `pnpm build`, then packages a Linux target with
  `electron-builder --linux AppImage`.
- Uploads the `.AppImage` both as a workflow artifact (for the current run)
  and as the asset of a rolling GitHub Release tagged `latest`, which is
  overwritten on every run — so `https://github.com/<you>/<repo>/releases/latest`
  always points at a build of the newest Vesktop commit.

## Keeping the repo "active"

GitHub automatically disables a workflow's `schedule` trigger if the repo
goes 60 days without a commit. To prevent that, the workflow also commits
a small `.status/last-build.md` file (timestamp + the Vesktop commit it
built) back to this repo on every run, so the schedule never goes stale.

## Notes

- Vesktop is Electron-based; the build only runs on Linux runners here, so
  you get a Linux AppImage. If you also want Windows/macOS builds, extend
  the `electron-builder` command with `--windows` / `--mac` and change
  `runs-on` accordingly (macOS builds need a `macos-latest` runner; Windows
  builds can cross-compile from Linux but native modules may need extra
  care).
- Since `main` may not build cleanly every single day (it's a dev branch),
  a run can occasionally fail upstream — check the Actions log if that
  happens. You can always re-run the workflow once upstream is fixed.
- Building from source is unofficial; the safest way to run Discord
  long-term is still the official Vesktop releases. Use this for testing
  the latest changes.
