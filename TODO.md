# SaintDesk — TODO

Ordered by what blocks what. `[!]` marks items only SK can do.

---

## Now

- [ ] **DNS: create `saintdesk.codesaint.co.in` → `92.4.92.212`.**
      Must be **grey-cloud / DNS-only** — SaintDesk is not HTTP and proxying kills it.
- [ ] **Repoint the relay**: `hbbs -r saintdesk.codesaint.co.in` in
      `/etc/systemd/system/rustdesk-hbbs.service`, then reload.
      Keep `rd.codesaint.in` resolving until no old client is left, since the
      rendezvous host is compiled into every binary already handed out.
- [ ] **TeamViewer-style home screen.** Two panels: "Your session" (ID +
      password, large) left, "Connect to partner" right. This is the change that
      makes it read as a product rather than a reskin.
- [ ] **Local Linux build to validate everything.** Nothing written since
      `89d4799` has been compiled — 51 regex-edited locale files, 4 Dart files,
      the vendored `hbb_common` and the pubspec font block are all unverified.
      Needs Rust pinned to 1.75 and Flutter to 3.24.5; the machine currently has
      1.98 / 3.47.2.

## Before handing a build to a client

- [ ] **Windows build.** Cannot be produced on Linux — RustDesk needs MSVC and
      vcpkg, with no cross-compile path. Comes from CI, or a Windows machine.
- [ ] **Client-facing SmartScreen guide.** Shipping unsigned is the accepted
      decision, so the friction moves to documentation: an illustrated
      "More info → Run anyway" walkthrough on codesaint.in.
- [ ] **Rehearse the support flow end-to-end** — client downloads the portable
      exe, reads out ID and password, SK connects. Never actually tested.
- [ ] **Three pages must exist on codesaint.in.** The first is an AGPL
      obligation, not cosmetic; the other two are linked from in-app help:
      - `/remote-support/source`
      - `/remote-support/macos-permissions`
      - `/remote-support/linux-x11`

## Going private

- [ ] `[!]` **`gh auth refresh -h github.com -s workflow`.** Pushing
      `.github/workflows/**` with a plain OAuth token is rejected even on a
      repo's first push — verified with a throwaway probe repo.
- [ ] Run `Company_Brain/products/codesaint-remote/go-private.sh`. GitHub refuses
      to make a public fork private and offers no detach, so the only route is a
      fresh non-fork repo with the history pushed into it.
- [ ] `[!]` Set `default_workflow_permissions=write` on the new repo, or
      `generate-sbom` fails the run at startup. The Claude Code auto-mode
      classifier blocks every `gh api .../actions/permissions` call, read
      included, so this one needs a human.
- [ ] Delete the archived forks once the private repo is confirmed good. They are
      renamed rather than deleted, so this is reversible until then.

## Deferred, with reasons

- [ ] **Code signing.** Decision was to ship unsigned. Revisit at Azure Trusted
      Signing, $9.99/month — Codesaint qualifies on the 3-year rule (registered
      2017). OV certs cost 15–25k/yr and still need reputation to accumulate, so
      they are worse value.
- [ ] **macOS notarization.** Unnotarized, Gatekeeper blocks harder than
      SmartScreen does. Only matters once a Mac client is actually in scope.
- [ ] **Relay monitoring.** If `saintdesk.codesaint.co.in` dies, the first signal
      is a client who cannot connect. The PBX outage already taught this lesson.
- [ ] **Android/iOS identifiers** still `com.carriez.*`. Left alone deliberately —
      changing an applicationId is a store-identity decision and mobile is not
      being shipped.
- [ ] Upstream README and docs still carry RustDesk text. Internal-only.

## Known non-issues

- Binary rename off `rustdesk` is **not needed** — `get_app_name()` already
  produces `SaintDesk.exe`. Renaming the Cargo package breaks every CI path and
  the `librustdesk` symbol the Flutter side loads, for nothing a client sees.
- The updater is inert: it only accepts `github.com/rustdesk/rustdesk/...` URLs
  and only fires when hbbs sends a software-update URL, which ours does not.
