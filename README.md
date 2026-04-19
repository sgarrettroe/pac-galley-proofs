# PAC Galley Proof Merge Tool

A browser-based tool for assembling galley proof PDFs for the [POGIL Activity Clearinghouse](https://pac.pogil.org). It merges user-supplied activity PDFs with standard default documents (cover sheets, rubrics, donation pages) into a single downloadable galley proof.

**No server, no installation, no build step** — just open `pdf-merger.html` in Chrome or Safari.

---

## How It Works

1. Open `pdf-merger.html` in a browser
2. Sign in with your authorized Google account
3. Select the activity kind (Learning Cycle or Application) and level (Idea, Review, Testing, Approved)
4. Drag and drop the activity PDF files — they are automatically matched to their slots
5. Click **Merge & Download** to generate the galley proof PDF

Default files (cover sheets, rubrics, donation page) are fetched automatically from GitHub Pages at merge time. You only need to supply the activity-specific files.

---

## Security

The repository is public, which means the HTML source — including the OAuth Client ID and API Key — is publicly visible. This is acceptable because three layers of protection remain in place:

- **OAuth Client ID** — designed to be public. It only works with an active Google sign-in and cannot be used from an unauthorized origin.
- **API Key** — restricted to the Google Sheets API only, and additionally restricted by HTTP referrer to `https://sgarrettroe.github.io/*` in the Google Cloud Console. Someone who copies the key from source cannot use it from their own domain or machine.
- **Allowlist gate** — even with both credentials, an attacker still cannot access the tool without owning a Google account listed in the allowlist spreadsheet.

**To maintain the HTTP referrer restriction:**
Google Cloud Console → Credentials → your API Key → Application restrictions → Websites → `https://sgarrettroe.github.io/*`

---

## Access Control

Access is restricted to authorized users via Google OAuth. After signing in, the app checks your email address against an allowlist maintained in a Google Sheet. To add or remove a user, edit the sheet directly — no code changes needed.

To request access, contact the PAC administrator.

---

## Updating the Allowlist

1. Open the allowlist Google Sheet (link in the design document)
2. Add or remove email addresses in column A, one per row
3. Changes take effect immediately — no redeployment needed

---

## Updating Default PDF Files

Default files live in this repository under two directories:

```
templates/    <- cover sheets and component pages
rubrics/      <- activity and process skill rubric PDFs
```

To update a default file:
1. Replace the file in the appropriate directory
2. Commit and push to the `main` branch
3. Wait 1-2 minutes for GitHub Pages to redeploy (check the **Actions** tab)
4. Hard-reload the browser to bypass any cached version (hold Shift + click Reload in Safari; Ctrl+Shift+R in Chrome)

---

## Updating the Galley Formulas

The slot structure for each kind/level combination is defined in the `FORMULAS` object near the top of the `<script>` section in `pdf-merger.html`. Each formula is an array of slot objects:

```javascript
{ order: 1, desc: 'Cover Sheet', source: 'default', path: 'templates/Coversheet-Activity Idea.pdf', required: true }
{ order: 3, desc: 'Submission Form', source: 'upload', pattern: '$1-POGIL?Activity?Submission?Form*.pdf', required: true }
```

**Fields:**

| Field | Values | Description |
|-------|--------|-------------|
| `order` | integer | Display order in the queue |
| `desc` | string | Human-readable slot name shown in the UI |
| `source` | `'default'` or `'upload'` | Whether the file is auto-fetched or user-supplied |
| `path` | string | Path relative to the GitHub Pages base URL (default slots only) |
| `pattern` | string | Filename match pattern with wildcards (upload slots only) |
| `required` | `true` or `false` | Whether the slot must be filled before merging is allowed |

**Pattern wildcards:**

| Character | Meaning |
|-----------|---------|
| `$1` | The submission number (extracted from the uploaded filenames) |
| `?` | Any single character |
| `*` | Any sequence of characters |
| `+` | A literal `+` in the filename |

---

## File Naming Convention

All user-supplied files for a given activity must start with the submission number followed by a hyphen:

```
367-Activity-Introduction-to-NMR.pdf
367-POGIL+Activity+Submission+Form+.pdf
367-Facilitation+Guide-NMR.pdf
```

The app extracts the submission number from the first uploaded file and validates that all subsequent files share the same number. The output filename is automatically set to:

```
[submission#]-[Kind]-[level]-galley.pdf
```

For example: `367-LC-review-galley.pdf`

---

## Google Cloud Project

The app requires a Google Cloud project with the following configuration:

| Item | Notes |
|------|-------|
| **Google Sheets API** | Must be enabled |
| **OAuth 2.0 Client ID** | Web application type; authorized JavaScript origin must match the GitHub Pages URL |
| **API Key** | Restricted to Google Sheets API only; HTTP referrer restricted to `https://sgarrettroe.github.io/*` |
| **OAuth Consent Screen** | Published (not in Testing mode) |

The Client ID and API Key are baked into the HTML source and visible in the public repository. Security relies on the HTTP referrer restriction on the API Key and the OAuth allowlist gate — see the Security section above.

---

## Repository Structure

```
pdf-merger.html          <- the application (single file)
templates/               <- default cover sheet and component PDFs
rubrics/                 <- default rubric PDFs
README.md                <- this file
docs/
  design-document.docx   <- full design and architecture document
```

---

## Browser Compatibility

Tested in **Chrome** and **Safari**. Requires a modern browser with support for:
- ES2020 JavaScript
- File API / drag-and-drop
- fetch()
- Blob URLs

Internet Explorer is not supported.

---

## Known Limitations

- An internet connection is required even for local use (to fetch default PDFs from GitHub Pages and to authenticate via Google OAuth)
- Each upload slot accepts one file. If a slot pattern could match multiple files, only the first match is used
- The app does not stamp metadata (title, author, date) into the merged PDF output

---

## Design Document

For full architecture details, see `docs/PAC-Galley-Tool-Design-Document.docx` in this repository.
