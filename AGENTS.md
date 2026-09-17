# AGENTS.md - smtweb (Schooi's Multitool website)

Static site for Schooi's Multitool (SMT). No build step, no framework. Edit files directly and keep everything in sync by hand - this file explains how.

## Layout

- `index.html` - homepage (hero terminal with live tool search, install, about, contact, footer).
- `features/index.html` - the tools catalog. Renders cards from an inline `tools` JS array. Has search + category filter tabs.
- `tools-search.js` - search index for the homepage hero (`window.SMT_TOOLS`, `{ n, t }` entries). Loaded by `index.html`.
- `styles.css` - all shared styles, dark/light themes via CSS vars (`body.light-mode`).
- `privacy.html`, `terms.html`, `accessibility.html`, `license.html`, `404.html`, `error.html` - standalone pages with duplicated nav/footer (no templating; update each file).
- `get/index.html` - NOT a page. It is the PowerShell installer payload served as static content. Never "fix" its markup, and keep it out of search indexes (`robots.txt` disallows `/get/`).

## The two tool indexes (must stay in sync)

There are two sources of truth for the tool list. Every tool must exist in BOTH with the SAME name:

1. `features/index.html` -> `const tools = [...]`, objects shaped:
   `{ name, desc, icon, cat, tags, added }`
   - `cat` must be one of: `network`, `system`, `windows`, `security`, `apps`, `dev`, `school`, `advanced`.
     Each value needs a matching `<option>` in `#categorySelect` - if you add a new category, add an option too.
   - `icon` is a Font Awesome 6 Free class suffix (rendered as `fas ${icon}`). Only use icons that exist in FA 6.4 free.
   - `tags` is lowercase space-separated search keywords.
   - `added` is a version label like `v1.x` / `v2.x` / `v2.3`. Do NOT add `new` flags or badges - NEW badges were deliberately removed.
   - Optional enrichment for the global Details toggle: `info` (1-2 factual sentences), `menu` (launcher path like `Tools > Network > 6`), `file` (repo path like `Files/ednsc.bat`, linked to GitHub automatically). Only cards with `info` render a details panel.
   - Destructive/offensive entries also get `danger: true`. They stay hidden until the visitor accepts the `#dangerGate` disclaimer modal (wired to `#dangerToggle`), and render with red `.is-dangerous` styling plus a `Danger` tag. Never list a dangerous tool without the flag.
   - Keep entries grouped by category (network -> system -> windows -> security -> apps -> dev -> school -> advanced).
2. `tools-search.js` -> `window.SMT_TOOLS`, objects shaped `{ n, t }` (`n` = exact same name, `t` = same keywords as `tags`).

## Truth rule: verify against the real repo before adding anything

The sibling checkout `../smt` is the authority. Never invent a tool.

1. The tool must appear in `../smt/SchooiMultitool.bat` menus (`:apps`, `:danger`, `:Network`, `:fixes`, `:cracks`, `:sysadmin`, `:utils`, `:fun`, secret commands) OR exist as `../smt/Files/*.bat`, `../smt/Files/Apps/*.bat`, or root `Uninstaller.bat`.Menus that are fully commented out (`REM :Performance`, REM'd info-screen lines) do NOT count - the user can't reach them.
2. Read the first ~25 lines of the `.bat` and write the description from what it ACTUALLY does (installer scripts all follow the same winget-then-IRM pattern; check the `title`/`echo` lines and the winget id or download URL).
3. The launcher menu text is the naming authority (e.g. launcher says "Spotify (No Ads)" -> use that name, not the script filename `bts.bat`).
4. One script = one card. Duplicates happened before (two cards for zicrack, dflc, bts, creds, autorespo, rcmc) - check the whole array for the script before adding.
5. Copy voice: one short plain sentence, sentence case (e.g. `Install Steam.`, `Runs Tron to deep-clean the PC. Takes hours and restarts.`). No marketing adjectives, no "advanced"/"powerful"/"ultimate". Dual-use and offensive entries must carry own-PC/lab-only scoping in the desc (e.g. `on PCs you own`, `for authorized testing`, `Lab use only`) - never present them as unrestricted.

## Count discipline (mostly automatic)

Visible counts update themselves from the data - do NOT hardcode new numbers into page body copy:

- `index.html`: `applyToolCount(TOTAL)` fills `heroCount`, `statTools`, `bootCount`, `aboutCount`, `heroTagline`, `footerTag`, `browseAll`, and the search `aria-label` from `tools-search.js` length. Static numbers in the HTML are no-JS fallbacks only. If you add a hook, guard with `if (el)` / `if (!n) return` like the existing code.
- `features/index.html`: the IIFE after `renderTools(tools)` fills `toolsTitle`, `allTab`, and `footerTag` from `tools.length`.
- Still MANUAL (update by hand on releases): `<title>`, meta/og/twitter tags, JSON-LD blocks, and the legal-page footer brand lines (`privacy.html`, `terms.html`, `accessibility.html`, `license.html`). Crawlers read these without running JS.
- Both JS arrays must have exactly N entries. Verify with:
  `Get-Content tools-search.js | Select-String '\{ n:' | Measure-Object`
  `Get-Content features\index.html | Select-String '\{ name:' | Measure-Object`
  (Note: backtick-quoted names like `` `"Some Settings Managed" Fixer` `` won't match the second pattern - count those by hand.)

## Tech constraints

- No animation libraries. AOS was removed - do NOT add `data-aos` attributes, scroll libraries, or keyframe entrance animations. Hover transitions and smooth anchor scrolling are fine.
- CSP is strict (`default-src 'none'`). External hosts allowed: `fonts.googleapis.com` + `fonts.gstatic.com` (fonts), `cdnjs.cloudflare.com` (Font Awesome 6.4.0). Do not add other CDNs (unpkg was removed on purpose) - update the CSP `<meta>` if you ever must.
- Fonts in use: Bricolage Grotesque (display), Inter (body), JetBrains Mono (terminal/code). `features/index.html` loads its own font subset - extend that `<link>` if a new face is needed.
- Use clean directory URLs in hrefs (`features/`, `./`, `../`, `features/#tool-x`) - never link to `index.html` directly, it looks bad in the address bar.
- Every page duplicates its nav/footer/scripts. A nav or footer change means editing `index.html`, `features/index.html`, and all four legal pages.
- `node --check tools-search.js` passes; keep it that way (it's plain data, no modules).

## Release checklist (version bump / new installer)

- Displayed version lives in `version.js` (`window.SMT_VERSION`) - bump it once, `index.html` picks it up. `<title>`/meta/JSON-LD tags stay manual (SEO snapshots).
- Installer hash and changelog are RUNTIME data, never hardcoded: `index.html` fetches from GitHub main with mirror fallbacks (`raw.githubusercontent.com` -> `api.github.com` for the exe, `-> cdn.jsdelivr.net` for the changelog) and shows a manual-check fallback when all sources fail. Keep all three hosts in the CSP `connect-src`, NEVER quoted (`https://host`, not `'https://host'` - a stray quote silently kills that source). Wrap any new `localStorage` access in try/catch - private-mode browsers throw on access and one throw kills the whole inline script.
- Write release notes as one change per line in `../smt/updatelogs.txt` - the site renders them as-is.

## Catalog deep links

- Tool cards render as `#tool-<slug>` (slug = lowercase name, non-alphanumerics become `-`). These are shareable: `features/#tool-dns-changer` clears filters and scrolls to the card. Keep names unique so slugs stay unique.
