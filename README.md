# tailorra

tailorra is a desktop app for tailoring your resume to each job you apply to, on Windows with beta Linux builds. You keep one master resume with everything you have ever done; for each job listing, the app helps you pick the relevant parts, phrase them to match the listing, style the result, and export it as PDF, DOCX, HTML, Markdown, or plain text. AI assists with extraction, selection, and wording, and never invents experience. It runs locally and stores data in a local SQLite database.

This repository hosts the release artifacts (installers and the auto-update manifest). The source repository is private.

## Download

- [Download tailorra for Windows (x64)](https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-x64-setup.exe)
- Linux, beta (x86-64): [AppImage](https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-amd64.AppImage) or [deb](https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-amd64.deb)
- [All releases](https://github.com/Egibi-LLC/tailorra-releases/releases)

Windows will likely show a SmartScreen warning ("Windows protected your PC") because the installer is not Authenticode-signed. Click "More info", then "Run anyway". Release binaries are signed with the project's minisign key, and the app verifies that signature on every auto-update.

Once installed, tailorra checks for updates on launch and installs them in place; you do not need to come back here for new versions. The one exception is the Linux deb, which does not auto-update (the AppImage does).

## Verify your download

Every release includes a `SHA256SUMS` manifest and a PGP signature over it (`SHA256SUMS.asc`), signed with the key in [tailorra-pgp-key.asc](tailorra-pgp-key.asc), fingerprint `F031 92B6 D53C E830 7257 67CE 8455 383C 29F2 D64E`. Step-by-step instructions: [Verify your download](https://egibi-llc.github.io/tailorra-releases/VERIFY.md).

## Documentation

- [User guide and docs site](https://egibi-llc.github.io/tailorra-releases/)

## Requirements

- Windows: Windows 10 or later, 64-bit. Microsoft Edge WebView2 runtime (preinstalled on current Windows 10/11; the installer handles it otherwise).
- Linux (beta): x86-64 with WebKitGTK 4.1, which current mainstream distributions include.
- Optional: an Anthropic API key or the Claude Code CLI for the AI features; everything else works without one.
