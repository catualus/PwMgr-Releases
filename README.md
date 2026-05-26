# PwMgr — Releases

A local-only password manager for Windows. No cloud, no sync, no telemetry.

This repository hosts the signed installer for [PwMgr](#) — the source repository is currently private (personal-use project), but the installer is public, signed, and reproducibly built by GitHub Actions from each tagged release.

---

## Download

→ **[Get the latest PwMgr-Setup.exe](https://github.com/Catualus/PwMgr-Releases/releases/latest)** (~130 MB)

The installer is signed with `CN=Catualus`, certificate thumbprint
`9305B727B55F12315408F7BFD29A635A9D679D3C`. PwMgr's in-app updater pins
that thumbprint and refuses to launch any downloaded installer that
doesn't match — so once you trust the initial install, future updates
can only be served by the same signer.

---

## What PwMgr does

- **One encrypted vault file per person**, stored wherever you choose. No
  cloud account, no sync server, no shared infrastructure.
- **Master password unlocks via Argon2id** — auto-calibrated to ~500 ms
  per attempt on your hardware. Strong KDF, GPU/ASIC attacks unhelpful.
- **XChaCha20-Poly1305 AEAD** body encryption (192-bit nonces, libsodium
  primitives via NSec).
- **Browser autofill** for Chrome / Edge / Brave / Firefox via a companion
  extension that talks to PwMgr over a local-only native-messaging
  channel. Nothing leaves your machine.
- **Save-credential prompts** when you log into a site that isn't in your
  vault yet — same flow as the major password managers, but every byte
  stays local.
- **TOTP codes** displayed inline with their countdown; copy with
  auto-clear.
- **Auto-lock** on idle, on Windows lock (Win+L), and on suspend.
- **Rolling backups + daily snapshots** — accidentally deleting an entry
  or hitting a power cut mid-write is recoverable.
- **Daily update check** against this repository, with publisher-signature
  verification on the downloaded installer before anything launches.

## What PwMgr deliberately doesn't do

- **No cloud, no sync, no telemetry, no analytics, no crash reporting.**
  Verifiable from the source (which is private for now — see "Why isn't
  the source public" below): the desktop app makes exactly one outbound
  network request per day, to GitHub's Releases API on this repo, to
  read the latest version tag. That's it.
- **No "forgot password" reset.** There is no recovery email, no
  cloud-backed key escrow. Use the in-app Recovery Export feature the
  first time you create a vault — it produces a separately-encrypted
  backup file you can stash on a USB stick or print as a QR code.
- **No mobile client yet.** Windows-only for now. The crypto and vault
  format are platform-agnostic, so a future Avalonia/MAUI client could
  consume the same vault file.

---

## Installing

1. Download `PwMgr-Setup.exe` from the latest release above.
2. Double-click it → click **Yes** on the Windows UAC prompt.
3. The wizard walks you through:
   - Installing the publisher cert (so Windows accepts the signed MSIX)
   - Installing the Windows App Runtime (the .NET 10 / WinUI 3 runtime)
   - Installing PwMgr itself
   - Optionally registering the browser extension for each installed
     browser
4. PwMgr appears in your Start menu. Launch it, create a vault, set a
   strong master password, and — important — run **Settings → Export
   recovery file** before adding real entries.

Average install time: under a minute on warm storage.

## Updating

PwMgr checks for updates once per day in the background. When a newer
release lands here, the app shows an in-app banner with three options:
**Download & Install** / **Skip this version** / **Dismiss**. You can
also manually check at **Settings → Updates → Check now**, or disable
the daily check entirely from the same place.

The auto-updater verifies the downloaded installer's signature against
the pinned publisher thumbprint *before* launching it. If GitHub's
release page were ever compromised and served a substituted binary, the
update would be refused.

---

## Security model

| Threat | Defense |
|---|---|
| Lost / stolen laptop | Master password → Argon2id KDF; no offline brute force at any practical scale for a strong password |
| Vault file readable on disk | Fully AEAD-encrypted; without the master password the file looks like random bytes |
| Clipboard scraper / Cloud Clipboard | Copied secrets auto-clear in 30 s and are hidden from Win+V history + Cloud Clipboard |
| Power loss mid-write | Atomic `File.Replace` + rolling backups + daily snapshots |
| Compromised release server | Auto-updater pins the publisher thumbprint — a mis-signed or unsigned installer is refused |
| Browser extension talking to a different process | Native-messaging host registers the extension's ID with the OS; the desktop app's IPC server cross-checks every connection |

Full threat model and honest limitations: [SECURITY.md](./SECURITY.md).

## License

[MIT](./LICENSE) — do what you like with it, no warranty.

---

## Why isn't the source public?

PwMgr is a personal-use project built for the author and a small circle
of friends. Keeping the source private removes the maintenance burden
of supporting drive-by contributors / random installs while the project
is still finding its shape.

The installer is reproducible from a tagged release via the GitHub
Actions workflow in the private source repo, the vault format is
fully documented (and ships with an independent ~150-line Python
reference decryptor in the source tree), and the in-app
signature-verification policy means even a private source repo can't
secretly substitute a backdoored update — the signed installer's
publisher thumbprint is the trust anchor, not the build system.

Source may open later. In the meantime, issue reports + feature
suggestions are welcome here.

## Issues / questions

Open an [issue](https://github.com/Catualus/PwMgr-Releases/issues). The
author triages these on a personal-time cadence — expect a few-day
response, not the next morning.

To report a security vulnerability, see [SECURITY.md](./SECURITY.md) —
please don't include exploit details in a public issue.
