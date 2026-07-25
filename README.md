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

- **`brew install` downloads the binary straight from Proton** — the formula's
  `url` points at `https://proton.me/download/drive/cli/<version>/…`. Nothing is
  rehosted; no build server sits in your install path.
- Homebrew requires a **SHA-256**, but Proton's manifest publishes **SHA-512**
  only (and the formula DSL has no SHA-512 field). So [`update.sh`](update.sh)
  fetches each binary once, **verifies Proton's SHA-512** for integrity, and
  computes the SHA-256 the formula needs.
- A daily GitHub Action ([`.github/workflows/update.yml`](.github/workflows/update.yml))
  runs `update.sh`; when Proton ships a new version it regenerates the formula and
  pushes it to the tap. It's a no-op when nothing changed.

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
