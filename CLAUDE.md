# CLAUDE.md — mohammed-brueckner.com

Claude Code settings for this repository. When a Claude session runs from
`/workspaces/platformeconomies`, the root `CLAUDE.md` there also applies; this file is more
specific and wins inside this repo.

**Kimi Code's settings for the same site are `AGENTS.md` in this directory. It holds the facts
— access model, hard rules, conventions, page inventory, history — and it is canonical.** This
file carries behaviour and points at `AGENTS.md` rather than copying it, so the two agents'
settings cannot drift. If this file and `AGENTS.md` disagree about a fact, `AGENTS.md` is right
and this file is the bug. Read `AGENTS.md` before any change. Do not write it unless the user
has explicitly granted an exception for that change.

Neither file is published: `_config.yml` `exclude:` keeps both off the site.

## Defaults

- **Deploy by default.** Push to `master` is the live deploy. Do not ask first.
- **Access:** use `$MOBRUEC_SITE_PAT`, never the codespace `GITHUB_TOKEN`, never print the token:
  `git push "https://x-access-token:${MOBRUEC_SITE_PAT}@github.com/MoBRUEC/MoBRUEC.git" master`.
- **Verify the build for the pushed SHA**, not "built" alone:
  `gh api repos/MoBRUEC/MoBRUEC/pages/builds/latest` until `status` is `built` and `commit`
  matches. New files can 404 for 30–60 seconds after that. A failed build leaves the previous
  build serving, so a green push is not proof of anything.
- **Validate live.** Playwright with bundled Chromium against the live URLs:
  `NODE_PATH=/workspaces/platformeconomies/node_modules node <script>`. `pageerror` and
  `console` listeners, zero errors, content markers rather than URLs, and for any page with a
  diagram, rendered SVG count equal to diagram block count. Delete the script after.
- **Rollback** is `git revert` + push. Never force-push.

## Rules specific to this site

1. **`AGENTS.md` → Hard Rules** first. The ones most often broken: never delete
   `medium-formatter/` or `mos-linkedin-formatter/`; splash-page links are sacred; no PDFs;
   `sitemap.xml` is hand-maintained; a `readme.md` with front matter needs `permalink:`; never
   restore `/openclaw/` from git history.
2. **mermaid loads on demand.** Never add a static mermaid `<script>` to `_layouts/default.html`.
   A diagram on a page needs no layout change.
3. **Plugins:** `jekyll-seo-tag` and `jekyll-redirect-from` only. Never `jekyll-sitemap`.
4. **Redirects** go in the *destination* page's `redirect_from:` front matter, and only where the
   topic genuinely continues — otherwise leave the 404.
5. **hreflang** goes in `alternates:` front matter: the full language set, including the page
   itself and `x-default`, on every page in the set.
6. **Author name** is Mohammed Brückner, with the umlaut, in anything visible or structured. The
   domain and file paths stay ASCII.
7. **Book promotion:** `AGENTS.md` → Book Promotion Policy. On the PlantUML hub and any new page:
   one plain sentence above the fold at most, the argued callout after the reader has what they
   came for, an author block at the end. No "BY THE WAY:" blocks, no flag strips, no locale lists.
   Buyers go to platformeconomies.com.
8. **PlantUML on a page must render.** Download the jar, install Graphviz (`linetype` needs it;
   Smetana ignores it silently), render every block to PNG and SVG, fail on errors, **and look at
   the images.** `-syntax` is not enough, and rendering without errors is not enough either.
9. **A new root-level `.md` file** must be added to `_config.yml` `exclude:` in the same commit,
   or GitHub Pages publishes it.
10. **After a content change**, submit the changed URLs to IndexNow (no account needed). Key file
    `/94401e4d4f0f3f5076d3290bb508b2e4.txt`, same key as platformeconomies.com; command in
    `/workspaces/platformeconomies/landing-page/VERCEL_ARCHITECTURE.md` §10 with this host.
11. **Exposure review before shipping** a new standalone HTML page, a new redirect set, or any
    change to what the build publishes: `.claude/agents/security-reviewer.md` in the
    platformeconomies repo, diff only.
12. **Legal or brand findings** get an adversarial pass before they reach the user. Verified facts
    are not a verified conclusion.
13. **Never invent metadata** — no dates, guests, ratings or attributions that cannot be checked.

## Design and voice

The site runs on `jekyll-theme-architect` with its own established styling; keep to it. The
Matisse default in the platformeconomies root `CLAUDE.md` does not restyle this site. Standalone
HTML pages inherit nothing from the layout and carry their own canonical, og tags, favicon, home
link and book-funnel link. Voice: `AGENTS.md` → Writing Style.

## Measurement

Search Console baseline, 12 months to 2026-09-10: 418 clicks, 27,389 impressions, 1.53% CTR.
`/plantuml-how-to/` is 73% of impressions and 79% of clicks at position 7.42; USA 0.37% CTR
against Germany 4.00%. The first honest read of the 2026-09-12 changes is a fresh export around
2026-10-10. Never make a deletion decision on a 3-month window.
