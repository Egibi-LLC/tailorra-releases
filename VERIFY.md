# Verify your download

Every tailorra release on [Egibi-LLC/tailorra-releases](https://github.com/Egibi-LLC/tailorra-releases/releases) ships with the artifacts needed to check that what you downloaded is what was built by the release workflow:

| File | What it is |
| --- | --- |
| `tailorra_<version>_x64-setup.exe` | Windows installer (NSIS, 64-bit) |
| `tailorra_<version>_amd64.AppImage` | Linux AppImage (beta) |
| `tailorra_<version>_amd64.deb` | Linux deb package (beta) |
| `*.sig` | minisign signature for the file it sits next to (used by the auto-updater) |
| `SHA256SUMS` | SHA-256 checksums of every installer |
| `SHA256SUMS.asc` | PGP signature over `SHA256SUMS` |
| `latest.json` | Update manifest polled by installed apps |

The stable-named copies (`tailorra-x64-setup.exe`, `tailorra-amd64.AppImage`, `tailorra-amd64.deb`) are byte-identical to the versioned files and are listed in `SHA256SUMS` too.

## 1. Check the SHA-256 checksum

Download `SHA256SUMS` from the same release as your installer.

Windows (PowerShell):

```powershell
Get-FileHash .\tailorra-x64-setup.exe -Algorithm SHA256
```

Compare the printed hash with the matching line in `SHA256SUMS`. `certutil -hashfile tailorra-x64-setup.exe SHA256` works too.

Linux, with the installer and `SHA256SUMS` in the same directory:

```sh
sha256sum -c SHA256SUMS --ignore-missing
```

## 2. Verify the checksum file with PGP

The checksum file itself is signed with the tailorra release-signing PGP key:

- Fingerprint: `F031 92B6 D53C E830 7257 67CE 8455 383C 29F2 D64E`
- Key: [tailorra-pgp-key.asc](tailorra-pgp-key.asc) (also [in the releases repo](https://github.com/Egibi-LLC/tailorra-releases/raw/main/tailorra-pgp-key.asc))

```sh
curl -LO https://egibi-llc.github.io/tailorra-releases/tailorra-pgp-key.asc
gpg --import tailorra-pgp-key.asc
gpg --verify SHA256SUMS.asc SHA256SUMS
```

A good signature prints `Good signature from "Tailorra Release Signing <ahubbard0597@gmail.com>"`. gpg will warn that the key is not certified by a trusted signature; that is expected for a key you just imported. What matters is that the fingerprint gpg prints matches the one above.

## 3. Verify the installer with minisign (optional)

The `.sig` next to each installer is a [minisign](https://jedisct1.github.io/minisign/) signature. It is what the built-in updater checks before installing an update, so you normally never need to check it by hand. To do it anyway:

```sh
minisign -Vm tailorra_<version>_x64-setup.exe -P RWRV77HDi8+0VR2s0jqetvkMfSqtYXVGBWo80UpZl4a6Z6SwuBtrbg8g
```

That public key is also baked into the app itself (`plugins.updater.pubkey` in its configuration), which is how installed apps authenticate every auto-update.

## About the Windows SmartScreen warning

The Windows installer is not Authenticode-signed (no code-signing certificate yet), so Windows shows "Windows protected your PC" the first time you run it. Click "More info", then "Run anyway". The checks above are the honest substitute: they prove the installer came from this project's release pipeline and was not modified in transit.

## About the Linux builds

The Linux AppImage and deb are beta: they build from the same source as the Windows app, but have had far less real-world testing (PDF export in particular). The AppImage auto-updates like the Windows app does; the deb does not auto-update, so check the releases page for new versions if you use it.
