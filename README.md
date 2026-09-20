<h1 align="center">smtweb</h1>
<p align="center">The official website for Schooi's Multitool</p>
<p align="center"><img src="https://smt.xubi.org/images/SMT.png" width="120" alt="Schooi's Multitool logo"></p>
<p align="center">
	<img src="https://img.shields.io/github/languages/top/SchooiCodes/smtweb" alt="GitHub top language">
	<img src="https://img.shields.io/github/commit-activity/w/SchooiCodes/smtweb" alt="GitHub commit activity">
	<img src="https://img.shields.io/github/languages/code-size/SchooiCodes/smtweb" alt="GitHub code size in bytes">
	<img src="https://img.shields.io/github/license/SchooiCodes/smtweb" alt="GitHub License">
	<a href=https://smt.xubi.org><img src="https://img.shields.io/badge/website-here-purple" alt="smt.xubi.org"></a>
</p>

About
-
Static website for [Schooi's Multitool (SMT)](https://github.com/SchooiCodes/smt), live at [smt.xubi.org](https://smt.xubi.org).

No build step, no framework - just HTML, CSS, and vanilla JS served via GitHub Pages (`CNAME` -> `smt.xubi.org`). Covers the homepage with live tool search, the **145**-tool catalog, the field guide, and the legal pages. Version display is currently **v2.3**.

Features
-
**Browse everything at [smt.xubi.org](https://smt.xubi.org)**, but here's what's in the repo:

- Homepage with hero terminal, live tool search, install instructions, and auto-fetched installer hash + changelog
- Tools catalog (`features/`) - searchable cards with category filters, shareable `#tool-<slug>` deep links, and gated Danger / ToS Bypass toggles
- Field guide (`guide/`) - launch flags, secret commands, and tips, verified against `SchooiMultitool.bat`
- Dark / light themes, responsive layout, SEO meta + JSON-LD, sitemap and robots
- Standalone legal pages: privacy, terms, accessibility, license, 404 / error

Compatibility
-
| Requirement | Details |
| ----------- | ------- |
| Hosting | GitHub Pages + custom domain (`smt.xubi.org`) |
| Browser | Any modern browser, no JS required for core content (counts enhance with JS) |
| Build | None - edit files directly |
| External hosts | Google Fonts + Font Awesome 6.4 CDN only (strict CSP) |

Installation
-
### View Live
Just open [https://smt.xubi.org](https://smt.xubi.org).

### Run Locally
If you have git installed, you can clone the repo:
```
git clone https://github.com/SchooiCodes/smtweb
```
Then open `index.html` directly, or serve the folder:
```
npx serve .
```

Usage
-
### Pages
| Path | Purpose |
| ---- | ------- |
| `/` (`index.html`) | Homepage - hero search, install, about, contact |
| `features/` | Full tool catalog, rendered from the inline `tools` array |
| `guide/` | Field guide, rendered from the inline `GUIDE` array |
| `privacy.html` / `terms.html` / `accessibility.html` / `license.html` | Legal pages (duplicated nav/footer, update each) |
| `404.html` / `error.html` | Error pages |

Shared assets: `styles.css` (themes via `body.light-mode`), `tools-search.js` (`window.SMT_TOOLS`), `version.js` (`window.SMT_VERSION`).

### The Two Tool Indexes
Every tool must exist in **both** with the **same name**:

1. `features/index.html` -> `const tools = [...]` (`{ name, desc, icon, cat, tags, added }`, plus `info` / `menu` / `file` / `admin` / `danger` / `tos` where applicable)
2. `tools-search.js` -> `window.SMT_TOOLS` (`{ n, t }`)

Verify counts match (currently **145** each):
```
Get-Content tools-search.js | Select-String '\{ n:' | Measure-Object
Get-Content features\index.html | Select-String '\{ name:' | Measure-Object
```

### Rules Before Adding a Tool
The sibling checkout `../smt` is the authority - never invent a tool. It must appear in `SchooiMultitool.bat` menus or as a `.bat` under `Files/`, with the description written from what the script actually does. See `AGENTS.md` for the full truth rule, copy voice, and category list.

### Checks
```
node --check tools-search.js
```
Counts on `index.html` / `features/index.html` update automatically from the data - only `<title>`, meta/JSON-LD tags, and legal-page footers need manual updates on release. Version lives in `version.js`.

Related
-
- Main tool: [SchooiCodes/smt](https://github.com/SchooiCodes/smt) - over **130** command line tools, installers, and utilities
- One-line installer: [SchooiCodes/getsmt](https://github.com/SchooiCodes/getsmt) - the bootstrapper served at `smt.gleeze.com` (`irm "https://smt.gleeze.com/" | iex`)
- Full feature list: [smt.xubi.org/features/](https://smt.xubi.org/features/)

Contributing
-
Contributions are welcome! If you have ideas for pages, catalog fixes, or guide corrections, please open an issue or submit a pull request. Keep changes dependency-free, CSP-clean, and verified against the `smt` repo.

License
-
smtweb uses the MIT license, find more [here](https://github.com/SchooiCodes/smtweb/blob/main/license.html). Same license as [SMT](https://github.com/SchooiCodes/smt/blob/main/LICENSE).
