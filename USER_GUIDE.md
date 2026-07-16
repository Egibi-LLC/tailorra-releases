# tailorra user guide

tailorra is a desktop app for job seekers, on Windows with beta Linux builds. You build one master resume corpus (everything you have ever done), then generate a tailored resume and cover letter for each job listing you apply to. Your data stays on your machine in a local database. AI features are optional and run through a provider you configure yourself.

This guide covers the whole flow: install, set up, build your corpus, add listings, tailor, style, export, and back up.

## 1. Install

tailorra is distributed as a Windows x64 installer, with beta Linux builds (AppImage and deb). There is no macOS build yet. To check a download's integrity before installing, see [Verify your download](VERIFY.md).

1. Download the installer:
   <https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-x64-setup.exe>
   If that link does not work, open the releases list at <https://github.com/Egibi-LLC/tailorra-releases/releases> and download the `-setup.exe` file attached to the newest release.
2. Run the downloaded file.
3. Windows SmartScreen will likely show a blue "Windows protected your PC" dialog. This is expected: tailorra's installer is cryptographically signed for the app's own updater, but it does not yet carry a Microsoft Authenticode certificate, so SmartScreen does not recognize the publisher. To continue:
   1. Click **More info** in the dialog.
   2. Click **Run anyway**.
4. Follow the installer prompts. When it finishes, launch tailorra from the Start menu.

### Linux (beta)

The Linux builds come from the same source as the Windows app but have had much less real-world testing, PDF export in particular. Two packages, x86-64 only:

- **AppImage** (<https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-amd64.AppImage>): make it executable (`chmod +x tailorra-amd64.AppImage`) and run it. It auto-updates like the Windows app. Needs WebKitGTK 4.1, which current mainstream distributions include.
- **deb** (<https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-amd64.deb>): install with `sudo apt install ./tailorra-amd64.deb`. The deb does not auto-update; watch the releases page for new versions.

### Updates

You only deal with SmartScreen once. After that, tailorra updates itself:

- Every time the app starts, it quietly checks the release channel for a newer version.
- Nothing installs behind your back. When an update is available, you choose when to install it: open **Settings (gear icon) > About**, and under **Updates** click **Download and install**. The app saves your current work, downloads the update, and restarts into the new version.
- You can also check manually any time with the **Check for updates** button on the same page.
- Every update is signature-checked before it installs, so a tampered download is rejected.

## 2. First launch

### Workspaces

The first thing tailorra asks you to do is create a workspace. A workspace holds one complete resume project: your master corpus, job listings, tailored versions, and saved styles. Most people need exactly one. You might want more if you maintain genuinely separate professional identities (say, software engineering and technical writing), or if you manage applications for someone else. Workspaces are fully isolated from each other, and you can switch between them from the workspace menu in the header at any time.

### Create your first workspace

On the Welcome screen:

1. Under **Start fresh**, type a workspace name (your own name is a fine choice).
2. Click **Create workspace**.

tailorra drops you straight into guided setup: an 11-step walkthrough that takes you from empty workspace to a styled, tailored resume. Each step is a normal page with a footer at the bottom; **Save & Next** saves what you typed and moves on. The steps:

1. Connect an AI provider
2. Add your personal info
3. Add at least one experience
4. Add education
5. Add skills
6. Add projects
7. Add certifications
8. Add awards / publications / volunteer / language
9. Add a job listing
10. Create a tailored version
11. Pick or design a style

You can leave guided setup whenever you want (**Exit guided setup** on the Getting started page) and come back later; the sidebar shows a dot next to each step so you can see what is done. Nothing in the app requires the wizard; every page works standalone.

The Welcome screen also offers **Load a backup** (restore a workspace from a `.tailorra` file, see section 9) and, once you have workspaces, a **Resume an existing workspace** list.

The app defaults to a dark theme. Change it under **Settings > Appearance** (System, Light, or Dark).

## 3. Build your master corpus

The master corpus is the superset of everything that could ever appear on a resume: every job, every bullet point, every skill, degree, project, and certification. You build it once and maintain it over time. Tailored versions (section 6) are made by selecting a subset of it per job, so the more complete the corpus, the better the tailoring.

The corpus lives in the **Master** section of the sidebar, one page per category:

- **Personal info**: name, contact details, address, links, and a professional summary.
- **Experiences**: jobs, each with title, employer, dates, location, an optional summary, and any number of bullet points.
- **Education**: degrees and schools.
- **Skills**: individual skills, grouped by category, with optional proficiency.
- **Projects**: personal or professional projects.
- **Certifications**: with issuer and dates.
- **Other**: awards, publications, volunteer work, languages.

### Manual entry

Each page has an add form. Fill it in, save, repeat. Several text fields have a **Refine with AI** or **Generate with AI** helper next to them once a provider is configured (section 4); ignore them if you prefer to write everything yourself.

### Import an existing resume

If you already have a resume file, do not retype it. Open **Tools > Import resume**:

1. Drop a file onto the drop zone, or click **Pick a file**. Supported types: PDF, DOCX, DOC, TXT, MD. You can also click **Pick a folder** to scan a whole folder of documents (old resumes, offer letters, notes) and choose which ones to include.
2. The text is extracted locally and shown on the page. Nothing has been sent anywhere yet.
3. From here you have two paths:
   - **Extract with AI** (requires an AI provider): sends the extracted text to your provider, which returns structured items: experiences with bullets, education, skills, and so on. A confirmation panel titled **AI extraction ready** shows counts of what was found; click through to insert everything into your master corpus.
   - **Add manually**: quick-add buttons (Personal info, Experience, Education, Skill, Project, Certification, Other item) open small forms so you can copy and paste from the extracted text yourself.

### Review AI-extracted items

Anything the AI inserted arrives marked as unreviewed, so you always get the final say:

- The sidebar shows a count next to each Master page with items waiting for review.
- On the page, unreviewed items are highlighted and carry a check-mark **Accept** button. Click it to accept the item as-is.
- Editing an item also marks it reviewed, since you have clearly looked at it.
- If an extraction went badly, go back to **Import resume**. A notice shows how many unreviewed AI-extracted items you have, with a **Clear them** button that removes only the unreviewed ones. Items you accepted or edited stay.

## 4. Set up an AI provider

AI features are optional but they are the point of the app, so set one up if you can. Open **Settings > AI providers**. Two providers work today:

### Option A: Anthropic API key

Pay-per-use API access to Claude models.

1. Get an API key from <https://console.anthropic.com/account/keys> (requires an Anthropic account with billing set up).
2. In **Settings > AI providers**, expand **Anthropic API** and click **Add key**.
3. Paste the key and click **Save**. The key is stored in the Windows credential manager (the OS keychain), not in a plain file.
4. Click **Test connection**. You should see a success message within a few seconds.

### Option B: Claude CLI (Max / Pro plan)

If you subscribe to Claude Max or Pro and have the Claude Code CLI installed, tailorra can route AI calls through your existing login instead of a pay-per-token key.

1. Install Claude Code from <https://claude.ai/code>.
2. Open a terminal and run `claude login`, signing in with your plan.
3. In **Settings > AI providers**, expand **Claude CLI (Max / Pro plan)** and click **Re-check status**. It should flip to Configured.
4. Click **Test connection**.
5. Optionally pick which Claude model the CLI should use from the model dropdown, or leave it on default.

OpenAI, Google Gemini, and OpenAI-compatible endpoints appear in the list marked **Coming soon**; they are not usable yet.

### The AI log

Every AI call's full prompt and response is visible in the **AI log** panel (button in the bottom-right of any page), so you can see exactly what left your machine and what came back. The log is kept in memory only and clears when the app closes. On the AI providers page you can choose whether the panel opens automatically when an AI action starts.

### What works without a provider

Everything except the AI itself. Without a provider you can still: enter and edit the whole master corpus, add job listings (paste, URL fetch, and browser capture), create tailored versions and pick items by hand, write cover letters yourself, browse all built-in styles in the designer, edit CSS directly, save style templates, and export to every format including PDF (use **Render as-is** instead of **Polish & render**). What needs a provider: Extract with AI, Suggest with AI, cover letter generation and refinement, AI polish, and AI style generation.

## 5. Add job listings

Each listing is a job you are considering. Open **Job listings** in the sidebar. There are three ways to get a listing in.

### Paste it

1. Click **New listing**.
2. Fill in **Company** (required) and optionally Role, Location, and Source URL.
3. Paste the job posting into the **Listing text** box.
4. Click **Save**.

You can also click **Upload file** under the text box to extract the text from a saved PDF, DOCX, DOC, TXT, or MD file.

### Fetch from a URL

In the same form, paste the posting's address into **Source URL** and click **Fetch from URL**. tailorra downloads the page and converts it to text for you. Many job boards (Indeed, LinkedIn) block automated fetching; when that happens, click **Open in browser** instead: the posting opens in your normal, logged-in browser, and you copy the visible text and paste it into the box.

### Capture from your browser (recommended)

The companion browser extension sends any job posting to tailorra with one click, no copying. It talks to the app over a local-only connection on your machine; nothing goes over the network. Set it up once from **Settings > Browser capture**, which walks you through the same steps:

1. **Create a pairing token.** Click **Generate pairing token**. This is a secret that proves to tailorra the capture came from your browser. Treat it like a password.
2. **Get the extension files.** Click **Get the extension**. tailorra copies the extension to a folder on your computer and opens it.
3. **Load it into Chrome or Edge.** Browsers do not let apps install extensions directly, so this part is manual:
   1. Click **Open browser & copy address**. Your browser opens with the extensions-page address on your clipboard. Paste it into the address bar and press Enter (or type `chrome://extensions` yourself; on Edge, `edge://extensions`).
   2. Turn on **Developer mode** (toggle in the top-right corner).
   3. Click **Load unpacked** and choose the folder from step 2.
   4. Click the puzzle-piece icon in the toolbar and pin **Tailorra capture** so its icon stays next to the address bar.
4. **Connect it to tailorra.** Right-click the Tailorra capture icon, choose **Options**, paste the pairing token (use the **Copy token** button on the settings page), leave the endpoint on its default unless you changed the port, click **Test connection** (you should see "Connected. Token accepted."), then **Save**.
5. **Try it.** Open any job posting, click the Tailorra capture icon, then **Send to Tailorra**. The listing appears under **Job listings** in tailorra, with the details filled in. The settings page counts captures received this session so you can confirm it works.

The extension knows the page layouts of LinkedIn, Indeed, Greenhouse, and Lever, and falls back to capturing the page text on other sites.

If you capture while tailorra has no workspace open (for example, right after launch while you are still on the Welcome screen), the capture is not lost: tailorra holds it and shows a message like "Capture received. Open a workspace to save it." As soon as you open a workspace, the held captures are saved into it. They are held in memory only, though, so if you quit the app before opening a workspace, they are gone. Captures sent while tailorra is not running at all never reach the app.

## 6. Create a tailored version

A tailored version is one resume built for one job listing: a selection of items from your master corpus, plus an optional cover letter, style, and application status.

The free tier includes 3 tailored versions per workspace; a one-time license removes the cap (see **Settings > License**). Everything else in the app works the same either way.

1. On **Job listings**, find the listing and click **Tailor a version**. This creates the version and opens the version editor. (Existing versions live under **Tailored versions** in the sidebar.)
2. In the editor, every section of your master corpus is listed with checkboxes. Check the experiences, bullets, education, skills, projects, certifications, and other items that belong on this resume. Bullets only appear when their parent experience is also selected. Each section shows how many of its items are selected, with select-all and clear controls.

### Let the AI pick (Suggest with AI)

With a provider configured:

1. Click **Suggest with AI** at the top of the editor. The AI reads the job listing against your entire master corpus and picks the items that fit. This usually takes 5 to 15 seconds.
2. A review panel titled **AI suggestions** appears, showing how many items of each kind it chose and a short rationale explaining its reasoning.
3. Click **Apply (replaces current)** to swap your current selections for the suggested set, or **Dismiss** to keep what you had.
4. Changed your mind? The confirmation message includes an **Undo** button that restores your previous selections. It stays on screen for about 15 seconds.

After applying, treat the suggestion as a starting point: walk the checkboxes and adjust.

### Cover letter

Click **Cover letter** at the top of the editor to open the cover letter panel.

- Write it yourself in the text box, or click **Generate with AI** to draft one from your master corpus and the linked listing. The optional direction field steers the draft ("emphasize platform work", "shorter", "more formal").
- **Refine with AI** rewrites the current draft with instructions you give it.
- **Save as PDF** renders the letter to a PDF file.

The cover letter belongs to this version, so each application keeps its own.

### Track your application

Every version has a **Status** dropdown that follows the real pipeline: draft, applied, interviewing, offer, accepted, rejected, and archived (archived hides a version from the default list). The **Tailored versions** page filters by these states, so it doubles as a lightweight application tracker. There is also a per-version notes field.

## 7. Style it

The resume's content and its look are separate. Content comes from your selections; the look comes from CSS, which you can take from a built-in style, generate with AI, or write yourself.

### In the version editor

The **Style** section of the version editor manages the look of that one version:

- **Templates**: pick a saved template from the dropdown, preview it rendered against this resume as a real PDF, and click **Apply to this version**.
- **Edit CSS directly**: opens a CSS editor for this version's custom stylesheet, if you want hands-on control.
- **Reset to default** returns the version to the built-in classic look.
- **Save as template** stores the version's current CSS under a name so you can reuse it on other versions.

Style changes affect the preview and every export of that version.

### The Resume designer

**Tools > Resume designer** is the style workshop. It works on styles independent of any one version:

1. Choose what to preview against with the **Preview against** dropdown: built-in example content, your own master data, or one of your tailored versions.
2. Browse the built-in styles. tailorra includes a spread of hand-made designs (classic, modern, compact, sidebar, minimal, executive, an ATS-plain one, and more), each shown as a live thumbnail.
3. With an AI provider, type an optional **Direction** ("navy and warm gray, restrained, executive feel") and click **Generate variations** to have the AI restyle the whole spread to match.
4. Click a variation to focus it. In the focused view you can:
   - **Refine this style** with follow-up instructions ("tighter spacing", "smaller section titles"); a history lets you step back.
   - Edit the CSS directly.
   - **Apply to [version name]** if you arrived from a version.
   - **Save as template** for later reuse.
   - **Save as PDF** or **Print** the previewed resume directly.

No provider? The designer still works: browse the built-in styles, edit CSS, save templates, and export PDFs. Only Generate variations and Refine need AI.

## 8. Export

Exports live in the version editor's **Final output** section.

### PDF (the main path)

1. Optionally type AI polish instructions ("tighten the bullets, normalize date formats").
2. Click **Polish & render** to run the resume through your AI provider for a final cleanup pass and render the result to PDF, or **Render as-is** to skip AI entirely and render exactly what you selected.
3. The rendered PDF appears embedded in the page. It is byte-for-byte the file you get, so what you see is what you send.
4. Click **Download PDF** to save it, or **Open externally** to view it in your default PDF viewer.

PDF rendering happens inside the app using the same Windows web engine that draws the app itself; there is no external dependency to install.

### Other formats

Under **Other formats**, one click each:

- **DOCX**: a Word document built from the resume content. It uses Word's own native styling, so custom CSS does not apply to it.
- **HTML**: a self-contained web page.
- **Markdown**: plain structured text for pasting into forms or editors.
- **Plain text**: for ATS fields and email bodies.

tailorra does not export the legacy `.doc` format. If an employer insists on `.doc`, export DOCX and convert it in Microsoft Word (File > Save As > Word 97-2003 Document); that conversion requires Word to be installed.

### Apply in the browser (fill the employer's form)

If you have the browser extension set up (section 5), tailorra can fill an employer's online application form for you: your contact details go into the matching fields and the tailored resume is attached to the form's resume upload. The steps:

1. In the version editor, click **Prepare browser autofill** (under the export buttons). tailorra renders the resume to PDF and DOCX and stages it, with your contact info, on the same local-only connection the extension already uses. Nothing leaves your machine at this point.
2. Open the employer's application form in your browser, click the **Tailorra capture** icon, and press **Fill this application**. The popup names the role and company you prepared, so you can confirm you are applying with the right resume before filling.
3. The extension fills the fields it recognizes (name, email, phone, address, LinkedIn, GitHub, portfolio) and attaches the resume. If the form's upload only accepts Word documents, the DOCX goes in instead of the PDF. Fields you already typed something into are left alone.
4. Review the form, complete anything the extension could not match (employer-specific questions, dropdowns, yes/no checkboxes), and submit it yourself. **tailorra never submits the application**; the final click is always yours.

Notes:

- The staged package is held in memory only. Preparing another version replaces it, and closing tailorra clears it.
- Some application systems use heavily scripted upload widgets that swap their file inputs around at click time. If the resume does not attach, the popup tells you; export the PDF and attach it by hand.

## 9. Back up and restore

### Save a backup

A backup is a single `.tailorra` file containing everything in the active workspace: corpus, listings, versions, cover letters, and style templates. (It is JSON inside, if you ever want to look.)

- Quick way: the **Save** button in the app header.
- Same thing: **Settings > Workspaces > Active workspace tools > Save backup**.

Pick where to save it. Anywhere you like: a synced folder, a USB drive, email to yourself. Make one whenever you have done meaningful work.

### Restore a backup

Two ways in:

- **Into a new workspace** (the usual case, for example on a new machine): on the Welcome screen, click **Pick file** under **Load a backup**. tailorra creates a fresh workspace and fills it from the file.
- **Into an existing workspace**: the **Open** button in the header, or **Settings > Workspaces > Load backup**. This replaces everything in the current workspace with the file's contents, and asks you to confirm before doing so.

Both accept `.tailorra` files and older `.json` backups. Restoring is all-or-nothing: if anything goes wrong mid-restore, the workspace is left exactly as it was, not half-loaded. Loading one backup into several workspaces (say, to duplicate a workspace) is safe; each restore gets its own copies.

### Where the data lives

Day to day, everything sits in a local SQLite database under `%LOCALAPPDATA%\dev.hubbard.tailor`. Uninstalling the app does not necessarily preserve that folder for a future reinstall, and databases do not survive machine failures. `.tailorra` files are the supported backup; use them.

**Settings > Workspaces** also offers **Reset workspace** (empty it but keep it) and workspace rename/delete. Both destructive actions ask first, but a recent backup makes them painless.

## 10. License

tailorra is free to use with a cap of 3 tailored versions per workspace. A one-time license removes the cap; there is no subscription and no account.

- **Buy**: **Settings > License > Get a license** opens the purchase page in your browser. You receive a license key by email.
- **Activate**: paste the key (it starts with `TLRA-`) under **Settings > License** and click **Activate**. Activation happens entirely on your computer: the app checks the key's cryptographic signature and stores it in your OS keychain. No server is contacted, so activation works offline.
- **Moving or reinstalling**: keep the key from your purchase email and paste it again on the new machine. You can remove the license from a computer on the same page.

## 11. Troubleshooting

### "The capture server could not start" on the Browser capture page

Browser capture listens on port 7341 on your own machine (it binds 127.0.0.1 only, so nothing outside your computer can reach it). If another program already owns that port, captures cannot work and the settings page shows this banner. Fix:

1. Close the other program if you know what it is, then restart tailorra. Or:
2. Set the `TAILORRA_CAPTURE_PORT` environment variable to a free port number (Start menu, type "environment variables", add it under your user variables), then restart tailorra.
3. If you changed the port, update the extension: right-click the Tailorra capture icon, choose **Options**, set the endpoint to the address shown on the Browser capture settings page, and **Test connection**.

Running two copies of tailorra at once causes the same conflict; keep one instance open.

### The extension says sent, but no listing appears

- Make sure a workspace is open in tailorra. Captures received with no workspace open are held with an "Open a workspace to save it" message and land as soon as you open one, but they are lost if you quit the app first.
- Check the capture counter under **Settings > Browser capture** (step 5). If it does not increase when you send, re-run **Test connection** in the extension's Options; the token may have been regenerated since you paired.

### "Couldn't reach the update server"

The update check needs to reach github.com. If you are offline or a firewall or proxy blocks it, the About page shows this message. Try again on a normal connection, or update manually: download the latest installer from <https://github.com/Egibi-LLC/tailorra-releases/releases> and run it over your existing install. Your data is untouched by reinstalling.

### AI errors

- **"AI response was cut off at the provider's output-token limit"**: the model ran out of room mid-answer, usually on a very large resume. Try again, or reduce the size of what you are sending (fewer selected items, a shorter polish, a trimmed import text).
- **Anthropic errors mentioning 401 or authentication**: the API key is wrong or was revoked. Re-enter it under **Settings > AI providers** and use **Test connection**.
- **Claude CLI shows "Not detected"**: the `claude` binary is not on your PATH or you are not logged in. Run `claude login` in a terminal, then click **Re-check status**.
- **An AI action seems stuck**: use the **Cancel** button next to the running action. The **AI log** panel (bottom-right) shows the exact prompt and response of every call, which usually explains what went wrong.

### "Couldn't open the app's database" at startup

Usually a leftover lock from a previous instance, often right after an update. Your data is safe on disk. Click **Try again** on the Welcome screen; if it persists, make sure no other tailorra window is running, then restart the app. **Settings > About > Diagnostics** shows the database status and the last error, and confirms the data folder location.
