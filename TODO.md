# SaintDesk — TODO

Ordered by what blocks what. `[!]` marks items only SK can do.

---

## In flight

- [ ] **Run `33720955099`** — Windows + macOS. `x86_64-pc-windows-msvc` and
      `aarch64-apple-darwin` have already passed, which is the proof the MSI and
      bundle-name fixes work; the remaining two are their siblings on the same
      code paths. Run deliberately while the repo is still public, where CI
      minutes are unmetered.
- [ ] **Migration is held until that goes green.** No reason to flip to private
      and then discover a packaging break on metered minutes.

## Going private

- [x] `gh auth refresh -h github.com -s workflow` — granted; all commits pushed.
- [ ] Run `Company_Brain/products/codesaint-remote/go-private.sh`. Creates
      `saurav61091/saintdesk`, archives the old forks by rename (reversible).
      GitHub refuses to make a public fork private and offers no detach, so a
      fresh non-fork repo with the history pushed in is the only route.
- [ ] `[!]` Set `default_workflow_permissions=write` on the new repo, or
      `generate-sbom` fails the run at startup. The Claude Code auto-mode
      classifier blocks every `gh api .../actions/permissions` call, read
      included, so this one needs a human.
- [ ] Delete the archived forks once the private repo is confirmed good.
- [ ] Delete the old workspace `/mnt/data320/Projects/Rust/codesaint-remote`.

## Before handing a build to a client

- [ ] **Client-facing SmartScreen guide.** Shipping unsigned is the accepted
      decision, so the friction moves to documentation: an illustrated
      "More info → Run anyway" walkthrough.
- [ ] **Rehearse the support flow end-to-end** — client downloads the portable
      exe, reads out ID and password, SK connects. Never actually tested.
- [ ] **Three pages must exist on codesaint.in.** The first is an AGPL
      obligation, not cosmetic — it is what discharges the source offer now
      baked into the About panel:
      - `/remote-support/source`
      - `/remote-support/macos-permissions`
      - `/remote-support/linux-x11`

## Infrastructure

- [ ] **No monitoring on the relay.** If `saintdesk.codesaint.co.in` dies, the
      first signal is a client who cannot connect. The PBX outage already taught
      this lesson once.
- [ ] **No access control on the relay.** Anyone who extracts the key from a
      distributed binary can use it as a free relay. Low stakes at current
      volume; worth knowing before wider distribution.
- [ ] **Retire `rd.codesaint.in`.** Still resolving on purpose. Safe to drop only
      once no binary built against it is left in the field — the rendezvous host
      is compiled in, not configured.
- [ ] **WorldPhone recharge ≥ ₹2,000** (telephony, unrelated but oldest open
      item). Outbound is dead on `402`, so the IVR reaches nobody.

## Dev environment

- [ ] **Pin Rust 1.75 and Flutter 3.24.5 locally.** The machine has 1.98 / 3.47.2.
      That mismatch is the entire cause of the 7 `DialogTheme` /`TabBarTheme`
      errors `dart analyze` reports, and until it is fixed a local build proves
      nothing. Everything else analyses clean.

## Deferred, with reasons

- [ ] **Code signing.** Decision was to ship unsigned. Revisit at Azure Trusted
      Signing, $9.99/month — Codesaint qualifies on the 3-year rule (registered
      2017). OV certs cost 15–25k/yr and still need reputation to accumulate, so
      they are worse value.
- [ ] **macOS notarization.** Unnotarized, Gatekeeper blocks harder than
      SmartScreen does. Only matters once a Mac client is real.
- [ ] **macOS signing path is inert** — CI references `MACOS_CODESIGN_IDENTITY`
      and `MACOS_P12_BASE64`, which we do not hold. An unsigned dmg is what comes
      out, which is consistent with the unsigned decision.
- [ ] **Linux package filenames** are still `rustdesk-1.5.0-*.deb/.rpm`. Only the
      Windows MSI and portable exe were renamed, because those are the two files
      a client receives. Linux is SK's own machine.
- [ ] `aarch64-unknown-linux-gnu` fails at "Install vcpkg dependencies".
      Upstream infrastructure, not branding, and no longer in the default
      `platforms`. Only worth chasing if an ARM Linux client ever matters.
- [ ] **Android/iOS identifiers** still `com.carriez.*`. Changing an
      applicationId is a store-identity decision and mobile is not shipping.
- [ ] Upstream README and docs still carry RustDesk text. Internal-only.

## Known non-issues

- Renaming the Cargo package is **not** needed. `get_app_name()` already produces
  `SaintDesk.exe` in `C:\Program Files\SaintDesk`. Renaming it breaks every CI
  path and the `librustdesk` symbol the Flutter side loads. Note this is
  narrower than it first looked: the product name *is* load-bearing in
  **packaging** paths, which is what broke the MSI and macOS jobs.
- The updater is inert: it only accepts `github.com/rustdesk/rustdesk/...` URLs
  and only fires when hbbs sends a software-update URL, which ours does not.
- `src/lang.rs` substitutes `RustDesk` → `APP_NAME` at runtime whenever
  `is_rustdesk()` is false, including on a lookup miss, so a missing translation
  entry cannot leak the upstream name.
