# Security

PwMgr is a local-only password manager. This document is the short,
user-facing version of the threat model — what's defended, what isn't,
and how to verify a release.

The full technical security document lives in the source repository.

## What's defended

| Concern | Defense |
|---|---|
| **Lost / stolen laptop** | Master password gated by Argon2id (auto-calibrated to ~500 ms per attempt). A strong master password is infeasible to brute-force at any practical scale. |
| **Casual malware reading the vault file** | The vault file is fully AEAD-encrypted with XChaCha20-Poly1305. Without the master password, the file is indistinguishable from random bytes. |
| **Clipboard scrapers / Cloud Clipboard exfil** | Copy operations set `CanIncludeInClipboardHistory=false` and `CanUploadToCloudClipboard=false`, plus a 30-second auto-clear. |
| **Power loss / crash mid-save** | Atomic writes via `File.Replace`, rolling backups (`vault.pwmgr.bak0`…`bak9`), and a 30-day daily snapshot pool. |
| **Silent disk corruption** | AEAD authentication tag + HMAC-SHA256 on the header. A corrupted vault is detected on open and the most recent valid backup is offered; the corrupted file is never silently overwritten. |
| **Format / KDF-parameter downgrade attacks** | HMAC-SHA256 over the header bytes, keyed by the KEK, detects any modification of KDF parameters, algorithm choice, or format version. |
| **Compromised release server** | The in-app updater pins the publisher thumbprint `9305B727B55F12315408F7BFD29A635A9D679D3C`. A substituted or unsigned installer is rejected before launch. |
| **Browser extension impersonation** | Native-messaging registrations declare the exact extension ID Windows is allowed to launch the host for; the desktop app's IPC server cross-checks every connection against the per-user authorized list, which the user explicitly approves on first connection. |

## What's NOT defended against

- **Keyloggers on an already-compromised host.** A keylogger sees your
  master password the moment you type it. No software-only defense
  helps. Hardware authenticators (FIDO2 / YubiKey) are deferred to a
  future version; the vault format reserves slots for them.

- **Kernel-level attackers / memory dumps.** PwMgr runs on a managed
  runtime (.NET), which cannot guarantee that key material is never
  swapped to disk or copied by the garbage collector before pinning.
  This is true of every managed-language password manager including
  Bitwarden's desktop client. We minimize plaintext key lifetime and
  pin sensitive buffers, but an attacker with kernel-level access to
  your machine wins.

- **Forgotten master password.** There is no cloud backup, no recovery
  email, no reset link. **Use the in-app Recovery Export the first time
  you create a vault** — it produces a separately-encrypted file you
  can stash on a USB stick or print as a QR code.

- **Nation-state adversaries with physical access.** A personal-use tool
  built by one developer cannot match the threat model of an
  HSM-backed enterprise product. What you do get: best-in-class
  primitives (Argon2id RFC 9106, XChaCha20-Poly1305), zero network
  attack surface beyond the daily update check, a tamper-evident open
  format, and an independent reference decryptor.

## Reporting vulnerabilities

If you find a security issue, **don't include exploit details in a
public issue**.

- For low-severity findings: open an issue and tag it `security`.
- For anything that meaningfully reduces an attacker's effort: reach
  out to the maintainer privately (open an empty issue titled "security
  contact request" and the author will follow up via your GitHub
  notification email).

This is a personal-use project; treat coordinated disclosure as a
courtesy, not a process.

## Verifying a release

Every PwMgr-Setup.exe published here is signed by the same publisher
cert (`CN=Catualus`, thumbprint
`9305B727B55F12315408F7BFD29A635A9D679D3C`).

To verify a downloaded installer before running it:

```powershell
$sig = Get-AuthenticodeSignature .\PwMgr-Setup.exe
$sig.Status                          # must equal "Valid"
$sig.SignerCertificate.Thumbprint    # must equal 9305B727B55F12315408F7BFD29A635A9D679D3C
$sig.SignerCertificate.Subject       # must equal "CN=Catualus"
```

Or via Windows Explorer: right-click `PwMgr-Setup.exe` → **Properties**
→ **Digital Signatures** tab → select the signature → **Details**.

If any of those don't match, **do not run the installer**. Open an
issue on this repo with `security` so the author can investigate.
