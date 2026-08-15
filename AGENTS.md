# AGENTS.md — mohammed-brueckner.com Website

Guidance for AI agents working on this repo. Read this before changing anything.

## What This Repo Is

Jekyll site (`jekyll-theme-architect`) on GitHub Pages, served at https://mohammed-brueckner.com. Push to `master` = automatic live deploy in ~1 minute. There is no staging; a broken build leaves the last good build serving (site can go stale, never dark).

## Access & Deploy Loop (established 2026-08-15)

- All agent git operations use a **fine-grained PAT** (`MOBRUEC_SITE_PAT`, stored as a Codespaces secret on `MoBRUEC/platformeconomies`, value write-only). Scope: this repo only — Contents RW, Pages RW, Pull requests RW. 90-day expiry.
- The broad codespace `GITHUB_TOKEN` must NOT be used for this repo.
- The remote URL is token-free; pass the PAT per-command: `git push "https://x-access-token:${MOBRUEC_SITE_PAT}@github.com/MoBRUEC/MoBRUEC.git" <branch>`.
- Deploy verification loop after every push: poll `gh api repos/MoBRUEC/MoBRUEC/pages/builds/latest` until `built` for the pushed commit SHA (check the SHA — "built" alone may be the previous build), then `curl` the changed pages and assert HTTP 200 + content markers. Rollback: `git revert` + push, never force-push.

## Hard Rules

1. **Never delete `medium-formatter/` or `mos-linkedin-formatter/`.** Personal tools of the owner. They are deliberately NOT in sitemap.xml (owner decision 2026-08-15). They were deleted once in the 2026-08-15 trim and had to be restored.
2. **Splash-page links are sacred.** Everything the homepage `README.md` links to must stay live: aicommunity, archimate, changemanagement, databricks, how-to-ai, jArchi-Build, plantuml-how-to, platform-economies, publications.
3. **No PDFs on the site.** All were removed 2026-08-15 (presentations, learning). Do not re-add; link out instead.
4. **Do not touch** `CNAME`, `google8ea635c601795bc4.html`, `robots.txt` without explicit instruction.
5. **sitemap.xml is hand-maintained.** Any page add/remove must update it in the same commit (lastmod = change date). Pages intentionally absent: medium-formatter, mos-linkedin-formatter.
6. **jekyll-seo-tag is active** (GitHub Pages default plugin) and `{% seo %}` is in `_layouts/default.html`. Every content page should carry `title:` + `description:` front matter — CTR-optimized, US audience in mind.

## Site Strategy (from GSC data, last-3-months report 2026-08-15)

- `plantuml-how-to/` is ~90% of all clicks (93/104). It is the hub; everything orbits it. Cluster: `archimate/`, `jArchi-Build/`, `plantuml-how-to/{templates,troubleshooting,plantuml-vs-mermaid}/` (added 2026-08-15).
- Winning queries: "plantuml archimate", "plantuml architecture diagram", long-tail "skinparam linetype ortho", "plantuml system architecture diagram", "enterprise architect plantuml import".
- Germany: high CTR, good position. USA: high impressions, low CTR — titles/descriptions optimized for US CTR.
- Secondary clusters: databricks (+cost-calculator), mlopswithdatabricks, changemanagement (+double-trinity model), how-to-ai (175 impressions, 0 clicks — CTR problem), integration.
- 2026-08-15 trim removed 16 dead sections (AMaaS, architect-career, cdmp-exam-simulator, dama-cdmp, excel, infographics, learning, matheapp, medium-formatter*, mos-linkedin-formatter*, openclaw, opensourcedalle2bearpictures, outsourcing, presentations, microsoft-copilot-and-copilot-studio, microsoft-fabric) — GSC confirmed zero clicks on all of them (* = restored by owner request).

## Book Promotion Pattern

The site promotes the owner's books, currently **Platform Economies** (launch September 1, 2026, Amazon ASIN B0GXN4PRB5, marketplaces US/UK/DE/JP/CA, book page `/platform-economies/`, external site platformeconomies.com).

Established pattern ("BY THE WAY:" block near the top of high-traffic pages, adapted per page — see `plantuml-how-to/readme.md` or `archimate/readme.md` for the template; compact variant on `databricks/`, `changemanagement/`, `how-to-ai/`, `mlopswithdatabricks/`). New content pages in traffic clusters should carry it. `developer-platforms/index.html` has an amber banner variant matching its own design system.

## Writing Style (keep it true)

Direct practitioner voice, first person. Bold key terms. Short punchy sentences mixed with longer ones. Dry parenthetical humor occasionally. `---` separators. Blockquotes for promos. Sparing emoji as section markers (📚 💡 🎧). No hype, no consultant-speak, no "In today's fast-paced world". Concrete numbers over adjectives. Match `plantuml-how-to/`, `archimate/`, `how-to-ai/` for reference tone.

## Conventions

- Folder-per-page: `<topic>/readme.md` (or `index.html` for standalone pages like developer-platforms, double-trinity-change-model). Sub-pages as nested folders (`plantuml-how-to/templates/readme.md`).
- Internal links between cluster pages are relative (`../troubleshooting/`); links from other sections are absolute (`https://mohammed-brueckner.com/plantuml-how-to/`).
- After ANY content change: run a repo-wide dead-link check (relative + absolute internal links) before pushing. A dead-link checker trap: `learning`, `excel` etc. also match plain words — check for actual link syntax only.
- **Front matter + folder-per-page trap (hit 2026-08-15):** adding YAML front matter to a `readme.md` makes Jekyll serve it at `/readme.html` instead of as the folder index — every folder page 404s. Any readme with front matter MUST also set `permalink: /<folder-path>/` in that front matter. Root `README.md` is exempt (no front matter there).
- **Live-render validation is mandatory for layout/nav/HTML changes:** after Pages reports `built` for the pushed SHA, run Playwright (bundled in the platformeconomies codespace: `NODE_PATH=/workspaces/platformeconomies/node_modules node test.js`, browsers at `~/.cache/ms-playwright`) against the live URLs with `pageerror`/`console` listeners, assert HTTP 200 + zero JS errors + content markers, click key nav links (assert the landed page is not a 404 — check a content marker, not just the URL), and view a screenshot. Note: Pages needs ~30–60s after "built" for new files to stop 404ing; also a URL-based nav check alone cannot distinguish a 404 page.
- **jekyll-seo-tag owns `<title>` and `<link rel="canonical">`.** The layout must NOT hardcode its own (removed 2026-08-15 after every page emitted two of each). Titles/descriptions live in page front matter; the site-wide fallback is `_config.yml` `title`/`description`.
- **og:image:** site-wide default via `defaults:` in `_config.yml` (book cover). Page-level `image:` front matter overrides, and MUST be an absolute path (`/platform-economies/...`) — relative paths resolve against the site root and 404 (bug hit 2026-08-15).
- **platformeconomics.com is a single-page site.** Do NOT link to sub-routes (gamebook25*, /podcast, /episodes/*) — they 404 (dead links removed 2026-08-15). Podcast links go to the Spotify show URL or `/podcast/`.
- PlantUML code blocks in new pages must **render** cleanly before shipping: download the jar (`https://github.com/plantuml/plantuml/releases/latest/download/plantuml.jar`), render each block with `java -jar plantuml.jar -tpng` into a scratch dir (add `!pragma layout smetana` if Graphviz is absent) and assert no "contains errors" on stderr. `java -jar plantuml.jar -syntax` is NOT sufficient — stdlib macros and relationship suffixes (e.g. `Strategy_Capability`, `Rel_Flow_Right`) only resolve at render time; wrong ones pass `-syntax` silently (trap hit and fixed 2026-08-15).

## History

- 2026-08-15: scoped-PAT access established; Phase A (book promotion across site); site trim (16 sections + all PDFs removed, sitemap rebuilt); formatters restored; Phase B (PlantUML cluster expansion + per-page SEO) via PR #2; permalink 404 incident (front matter bypasses readme-index) fixed same-day; content round 3 via PR #3 (scorecard tool, platform-economy explainer, C4 guide, APIOps guide, German Anleitung); full audit fixes (og:image 404, duplicate title/canonical, dead platformeconomies.com links, favicon/og on standalone pages, lang-per-page).
- 2026-08-15 additions to the page inventory: `scorecard/`, `platform-economy/`, `apiops/`, `plantuml-anleitung/`, `plantuml-how-to/{templates,troubleshooting,plantuml-vs-mermaid,c4-diagrams}/`, `llms.txt`.
