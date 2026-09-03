# SaintDesk — Changelog

Remote support client for Codesaint Technologies Pvt. Ltd., forked from
[RustDesk](https://github.com/rustdesk/rustdesk) (AGPL-3.0).

Upstream baseline: `e4539fc`.

---

## [Unreleased]

### Product identity
- Named the product **SaintDesk**, joining SaintVoice and SaintBill.
  `APP_NAME` must stay a single token — `src/common.rs` derives the URI scheme
  from `get_app_name().to_lowercase()`, so a space would emit an invalid
  `saint desk://`.
- Relay moves from `rd.codesaint.in` to **`saintdesk.codesaint.co.in`**, so the
  hostname carries the product rather than an abbreviation.
- macOS bundle identifier `in.codesaint.saintdesk`.
- Company name stays *Codesaint Technologies Pvt. Ltd.* in copyright, vendor and
  `CompanyName` fields. The product is SaintDesk; the company is Codesaint.

### Branding
- `APP_NAME` turned out to drive most of it: `get_app_name()` derives the Windows
  install directory, Start-menu folder, service name, registry subkey, tray mutex
  **and the installed executable name**. Installed layout is therefore
  `C:\Program Files\SaintDesk\SaintDesk.exe` with no binary rename.
- `src/lang.rs` also substitutes `RustDesk` → `APP_NAME` at runtime whenever
  `is_rustdesk()` is false, *including on a lookup miss*, which covers strings
  that have no translation entry at all.
- Fixed the surfaces `APP_NAME` does **not** reach:
  - `tabbar_widget.dart` hardcoded the literal `"RustDesk"` in the desktop title
    bar; now reads `bind.mainGetAppNameSync()`.
  - `[package.metadata.winres]` in both `Cargo.toml` and `libs/portable/Cargo.toml`.
    Windows shows `FileDescription` on the **UAC elevation prompt**, which was the
    one string a client was guaranteed to read.
  - The sciter UI keeps a separate palette in `src/ui/common.css`.
- Locale sweep across 51 files. Only the **value** half of each pair is touched —
  keys are the identifiers `translate()` takes, and renaming one makes the lookup
  miss and render the raw key.
- Icons regenerated from the Company_Brain mark for every platform target.

### Design
- Accent `#f37649` through `MyTheme` (accent / button / peer ID), `ColorScheme`
  primary, the msgbox default, the connection-manager gradient the *controlled*
  machine displays, and the sciter palette.
- **Inter** bundled at four weights as the base `fontFamily` on both themes.
  Poppins is the brand display face but reads heavy at UI sizes.

### Infrastructure
- Self-hosted relay: `hbbs` + `hbbr` as native systemd units on an OCI
  Always-Free micro. Native rather than Docker — 8 MB + 2.5 MB RSS on a 954 MB box.
- **Relay signing key backed up** to `Company_Brain/secrets/keys/rustdesk_relay_signing_key`.
  It previously existed only on the instance; losing that box would have meant
  reinstalling on every client machine. Not to be confused with `rustdesk_oci`,
  which is only the SSH login key.
- Vendored `hbb_common`, dropping the submodule. A private repo's `GITHUB_TOKEN`
  cannot clone a private sibling, and 19 checkout steps use `submodules: recursive`.
  Cost: upstream updates become a merge rather than a pointer bump.

### CI
- `flutter-build.yml` gained a `platforms` input gating 13 jobs, defaulting to
  `windows,macos,linux`. Matching is comma-delimited via `format()` so `linux`
  does not also select `linux-drm`.
- Removed every automatic trigger: the nightly `0 0 * * *` cron, and the
  `push`/`pull_request` builds in `flutter-ci` and `ci`. Tag builds stay.
  Rationale: measured on run `33716192726`, a full matrix is ~2500–3500 billable
  minutes against a 2000/month allowance, because the three macOS jobs carry a
  **10× multiplier**. A single Windows target is ~66 billable minutes, so
  `-f platforms=windows` is roughly 28 builds a month inside the free tier.
- Fixed `startup_failure`: forks default `GITHUB_TOKEN` to read-only, which fails
  `generate-sbom`'s `contents: write`. The API exposes nothing about this —
  `/jobs` empty, `/logs` 404 — the message exists only on the run's web page.
- Gating measured, not assumed: `-f platforms=windows,macos` produces **8 jobs
  against 22** for the full matrix. Android, iOS, sciter, web, linux-drm,
  appimage and flatpak skip without occupying a runner.

### Packaging

The product name turns out to be load-bearing in packaging paths, and upstream
never had to notice: `RustDesk.exe` matches the built `rustdesk.exe` on a
case-insensitive filesystem, and `RustDesk.app` matches its own bundle. Renaming
broke both, in places whose step names gave no hint packaging was involved.

- **MSI** — `preprocess.py` locates the app as `<app-name>.exe` inside the dist.
  The rename now happens on a *copy* of the dist, because `./rustdesk` is still
  consumed by the portable self-extractor and the MSI template step, and only the
  exe moves: `librustdesk.dll` and `drivers\RustDeskPrinterDriver` are payload
  that must keep their names.
- **macOS** — `build.py` copied the service binary into a hardcoded
  `RustDesk.app/Contents/MacOS/`. It now reads `PRODUCT_NAME` from
  `AppInfo.xcconfig` so the two cannot diverge again. Seven `RustDesk.app`
  references in CI, the dmg filename, the notarisation target and a rename glob
  now follow `env.APP_NAME`.
- The two artifacts a client actually receives — the MSI and the portable
  self-extractor — are named after the product.
- The Linux binary stays `/usr/bin/rustdesk`; only its `.desktop` display name is
  SaintDesk. Client-facing platforms are Windows and macOS, and renaming it would
  reach into the Cargo package and the `librustdesk` symbol Flutter loads. The
  Linux package *filenames* are likewise still `rustdesk-*`.

**Verified** in run `33720955099`: `x86_64-pc-windows-msvc` and
`aarch64-apple-darwin` both pass — the exact two jobs that failed in
`33716192726`. Deliberately run while the repository was still public, where CI
minutes are unmetered, because these were the last unproven changes.

### Legal
- AGPL-3.0 obligations: rebranding is permitted, but conveying builds to clients
  requires offering them the Corresponding Source. Private repositories are fine;
  publishing to the world is not required. The About panel carries the Codesaint
  copyright plus a source-offer link.
