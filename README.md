# proton-drive-cli-brew

Homebrew packaging for **[Proton Drive CLI](https://proton.me/blog/proton-drive-cli)**
— Proton's end-to-end encrypted cloud storage, from the terminal — for **macOS and
Linux** (arm64 + x86_64).

Proton ships pre-built binaries but no Homebrew formula. This repo generates one
and keeps it current in [`jakobhviid/homebrew-tap`](https://github.com/jakobhviid/homebrew-tap).

## Install

```sh
brew install jakobhviid/tap/proton-drive-cli
proton-drive version
proton-drive auth login
```

The command is `proton-drive` (matching Proton's own docs); the *formula* is
`proton-drive-cli` so it doesn't collide with the `proton-drive` cask (the macOS
desktop app).

## How it works

- **macOS (and non-x86_64 Linux) download straight from Proton** — the formula's
  `url` points at `https://proton.me/download/drive/cli/<version>/…`. Homebrew on
  macOS always has `clang` (via the required Command Line Tools), so it installs
  the Proton binary directly.
- **x86_64 Linux pours a bottle.** Homebrew-on-Linux requires a **C compiler** for
  any non-bottled formula — a problem on atomic/immutable distros (Bazzite, Silverblue)
  that don't ship one. A *bottle* is the only thing Homebrew pours without a
  compiler, so we repackage Proton's own Linux binary into an `x86_64_linux` bottle
  (no modification — verified against Proton's SHA-512) and host it on this repo's
  GitHub releases. Same pattern amdl/grove/pwtune use.
- **Checksums:** Homebrew needs a **SHA-256** but Proton publishes **SHA-512** only,
  so [`update.sh`](update.sh) fetches each binary once, **verifies Proton's SHA-512**
  for integrity, and computes the SHA-256 the formula (and bottle) need.
- A daily GitHub Action ([`.github/workflows/update.yml`](.github/workflows/update.yml))
  runs `update.sh`; when Proton ships a new version it releases the new bottle and
  pushes the formula to the tap. No-op when nothing changed. GitHub disables a repo's
  cron after 60 days of inactivity, so the job also pushes a throwaway `keep-alive`
  commit only when this repo is ≥50 days idle — enough to keep the schedule alive
  without daily noise.

## Setup (one time)

The updater pushes to the tap over SSH, so add the tap's deploy key as a secret
in **this** repo:

- `TAP_DEPLOY_KEY` — the same Ed25519 deploy key used by your other tool repos to
  write to `jakobhviid/homebrew-tap`.

Then run the workflow once (`Actions → update-formula → Run workflow`) to publish
the initial formula, or bump it locally: `./update.sh && cp proton-drive-cli.rb …/homebrew-tap/Formula/`.

## Notes

- Single self-contained binary (~114 MB, Bun-embedded); no runtime dependencies.
  Auth is browser-based, sessions stored in the OS credential store (Keychain /
  libsecret / Windows Credential Manager).
- Linux uses the glibc build; Windows and the musl variants are intentionally
  skipped (Homebrew targets glibc on Linux and doesn't do Windows).
- **Upstreaming:** Homebrew packages are community-maintained (the `proton-drive`
  *cask* is too, and is macOS-only). A homebrew-core source-build via `bun` is a
  possible future home; this tap is the immediate cross-platform one.

## License

The Proton Drive CLI is MIT-licensed (source in
[`ProtonDriveApps/sdk`](https://github.com/ProtonDriveApps/sdk)). This packaging is
MIT too.
