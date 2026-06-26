# CM360 Bulk Builder

A single-file, browser-based tool that generates Campaign Manager 360 (CM360, formerly DCM) bulk import files. It replaces the old Excel "DCM bulk sheet creator" workflow for trackers and tags, adds a bulk URL editor for live exports, and lets you configure your own placement naming convention. Built for the Initiative MENA trafficking workflow.

Everything runs in the browser. No server, no build step, no data leaves the page.

---

## What it does

Trafficking in CM360 means importing a "Campaign spreadsheet": a fixed 36-column sheet that describes placements, ads, and creatives. Building those by hand in Excel is slow and error-prone. This tool generates that exact file from a few inputs, validates every row before export, and produces a `.xlsx` ready to import.

It has four tabs:

| Tab | Purpose |
| --- | --- |
| **Naming** | Configure and save placement naming conventions (delimiter plus ordered tokens). |
| **Tracker** | Generate 1x1 click trackers (Type = Tracking), optionally one placement per creative dimension. |
| **Tag** | Generate served ad tags (Type = Standard) from a pasted CM360 creative export. |
| **Bulk edit** | Load a live CM360 export, find-and-replace on a column (usually the URL), re-export only the changed rows. |

---

## Tabs in detail

### Naming

Build a convention by choosing a delimiter and ordering tokens. Search the catalog (campaign, client, platform, product, dimension, language, medium, bid type, objective, communication type, audience, channel) or add a custom token. A live preview shows the result as both a Tracker and a Tag placement name.

Tokens are one of two kinds:

- **Variant** (platform, language, target, dimension, product): things you select several of, which multiply your placements.
- **Constant** (campaign, client, bid type, objective, and so on): typed once per batch.

Conventions are saved as **named presets**. The Tracker and Tag tabs each pick which preset they use from a dropdown at the top of their setup panel, so you can keep a separate tracker convention and tag convention while sharing one governed pool. When a preset includes a token a tab does not natively produce, that tab automatically shows a small input for it and slots the value into the name in the right position. Tokens that do not apply to a tab are dropped from that tab's name with no doubled delimiters.

Presets persist in the browser (see Persistence below) and can be exported or imported as JSON for sharing across a team.

### Tracker

1. Choose a naming preset, client, and campaign, set dates, tracker name, tracker ID, and time zone.
2. Pick platforms, languages, and formats/targets. The equation shows how many placements that produces.
3. Optionally tick "Create a separate placement per creative dimension" to make one placement per size (same tracker ID), otherwise a single 1x1.
4. Set click URLs. Auto-build a UTM from a base landing page, with optional per target/language overrides for Arabic vs English LPs or per-product URLs. Every row URL stays editable so you can paste shortened links.
5. **Generate** appends rows below (de-duplicated), each removable. Fix any flagged rows, then export.

### Tag

1. Choose a naming preset, campaign, and medium (file type), and pick platforms.
2. Paste the raw CM360 creative export (creative ID plus name, tab or comma separated, either column order). The tool parses dimension, language, and product from each creative name and shows them in an editable table; your edits lock so they survive a re-parse.
3. Set a default click URL (with an optional per product/language override matrix and a UTM toggle).
4. **Generate** builds one served tag per platform per included creative. Ads are Type = Standard with the real creative ID and name attached.

Tags carry no ad start/end/timezone; they inherit timing from the placement in CM360, matching the original workflow.

### Bulk edit

1. Load a CM360 export (`.xlsx`, `.xls`, or `.csv`). All columns and IDs are preserved so re-import updates the existing entities instead of creating new ones.
2. Filter to the rows you want (by site, by placement/ad name, or by what the target column currently contains).
3. Find-and-replace on the chosen column (defaults to the creative-assignment click-through URL), or set the whole cell to a fixed value, scoped to the filtered rows. Match-case optional.
4. Preview shows current value next to new value, changes highlighted. **Apply** commits; you can stack multiple passes. "Revert these rows" undoes edits within the current filter.
5. Export gives you only the changed rows, every original column intact.

No file handy? Each tab has a **Load sample** button.

---

## Output format

The export is a workbook with two sheets:

- **Campaign spreadsheet**: the 36-column CM360 import sheet, headers byte-matched to a real CM360 export (including the embedded newlines in header cells).
- **Help**: the standard CM360 help links sheet.

Export file names:

- Tracker: `DCM Tracker - {campaign} - Upload on CM360.xlsx`
- Tag: `DCM Tags - {campaign} - Upload on CM360.xlsx`
- Bulk edit: `{original name} - edited - Upload on CM360.xlsx`

The structure was verified column-by-column against the original Initiative DCM sheets. Always run one real import on your account to confirm before trusting a large batch.

---

## Running and hosting

It is one HTML file. To run locally, just open `cm360-bulk-builder.html` in a browser.

To host (recommended for the team so presets persist and everyone uses the same link):

1. Push the file to a GitHub repo.
2. Deploy with Vercel (no framework, no build command, output is the static file) or enable GitHub Pages on the repo.
3. Point a domain through GoDaddy if you want a branded URL.

There is no build step and no dependencies to install. SheetJS loads from a CDN at runtime.

---

## Configuration and maintenance

All lookup data lives in one `CONFIG` object near the top of the `<script>` block. Edit it directly to maintain the tool:

- **`platforms`**: each entry is `{name, siteId, siteName}`. Add a platform by adding an object. The `name` becomes the UTM source slug.
- **`clients`**: a list of client names (16 shipped). Users can also add a client on the fly from the Tracker tab, but those additions last only for the session, so add recurring clients here.
- **`languages`**: default language chips (AR, EN). More can be added in-session.
- **`dimensions`**: the creative size catalog, each `{s, l}` (size and label). Custom sizes can be added in-session.

### Known blank site names

Two platforms ship with an empty `siteName` because it was blank in the source workbook:

- Chinese Cars (`10174526`)
- Almuraba (`1664221`)

When selected, the tool shows a notice with an inline field to type the correct CM360 site name for that session. To fix it permanently, fill in the `siteName` for those two entries in `CONFIG.platforms`.

---

## Naming presets and persistence

- Default presets `Tracker (default)` and `Tag (default)` reproduce the original Initiative names exactly, so nothing changes unless you choose a different preset.
- Presets are stored in browser `localStorage` under the key `cm360_naming`. This works once the tool is hosted (Vercel, Pages, or any real URL). It does not persist inside a sandboxed preview, so use **Export presets** to keep a portable `cm360-naming-presets.json` and **Import** to load it elsewhere or share it.

### A note on duplicates

If you select several values of a variant (for example AR and EN) but drop that token from the active convention, the generated placements collide on an identical name. The built-in duplicate-name validator flags this on generate, before export.

---

## Technical notes

- **Single file**: all HTML, CSS, and JavaScript in one document. The Initiative logo is embedded as a base64 data URI.
- **Dependency**: SheetJS (`xlsx`) `0.18.5`, loaded from `cdnjs.cloudflare.com`. Needs internet access on first load to fetch it.
- **Fonts**: League Gothic (headings) and Poppins (body) from Google Fonts. The brand's Aldine 721 BT body font is a paid Bitstream font and is substituted with Poppins.
- **Brand**: primary `#5EC7EB`, secondary `#FFFF99`, accent `#2196F3`, background `#F5F2EE`.
- **Privacy**: fully client-side. Files you load are read in the browser and never uploaded anywhere.
- **Browser support**: any current Chrome, Edge, Firefox, or Safari.

---

## Roadmap

Possible additions, not yet built:

- A naming-convention validator that checks an imported sheet's names against the active preset.
- Dedupe a new batch against a live tag export so nothing is double-trafficked.

---

## Credits

Built for Initiative MENA. Maintained by the Measurement and Analytics team.
