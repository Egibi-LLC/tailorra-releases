# tailorra

tailorra is a Windows desktop app for tailoring your resume to each job you apply to. You keep one master resume with everything you have ever done; for each job listing, the app helps you pick the relevant parts, phrase them to match the listing, style the result, and export it as PDF, DOCX, HTML, Markdown, or plain text. AI assists with extraction, selection, and wording, and never invents experience. It runs locally and stores data in a local SQLite database.

This repository hosts the release artifacts (installers and the auto-update manifest). The source repository is private.

## Download

- [Download tailorra for Windows (x64)](https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-x64-setup.exe)
- [All releases](https://github.com/Egibi-LLC/tailorra-releases/releases)

Windows will likely show a SmartScreen warning ("Windows protected your PC") because the installer is not Authenticode-signed. Click "More info", then "Run anyway". Release binaries are signed with the project's minisign key, and the app verifies that signature on every auto-update.

Once installed, tailorra checks for updates on launch and installs them in place; you do not need to come back here for new versions.

## Documentation

- [User guide and docs site](https://egibi-llc.github.io/tailorra-releases/)

## Requirements

- Windows 10 or later, 64-bit
- Microsoft Edge WebView2 runtime (preinstalled on current Windows 10/11; the installer handles it otherwise)
- Optional: an Anthropic API key or the Claude Code CLI for the AI features; everything else works without one
