---
permalink: /plantuml-how-to/troubleshooting/
title: "PlantUML Troubleshooting: Every Error I Have Hit (and Fixed)"
description: "The PlantUML error catalog: include failures, size limits, linetype ortho and skinparam syntax, blurry exports, theme conflicts, Graphviz issues, VS Code and Kroki fixes."
---

# PlantUML Troubleshooting: Every Error I Have Hit (and Fixed)

Years of PlantUML in real projects, condensed into a catalog. Organized by symptom, because that is how you actually search for these things. If your error is not here, the answer is almost always: update PlantUML first, then come back.

BY THE WAY:
**Platform Economies** — out now.  
[**Buy on Amazon**](https://www.amazon.com/dp/B0GXN4PRB5) — Kindle and paperback. Also available in [🇬🇧 UK](https://www.amazon.co.uk/dp/B0GXN4PRB5), [🇩🇪 DE](https://www.amazon.de/dp/B0GXN4PRB5), [🇯🇵 JP](https://www.amazon.co.jp/dp/B0GXN4PRB5), and [🇨🇦 CA](https://www.amazon.ca/dp/B0GXN4PRB5).  
Book page: **[mohammed-brueckner.com/platform-economies](https://mohammed-brueckner.com/platform-economies/)**.

> 📚 Companion pages: [PlantUML with ArchiMate Guide](https://mohammed-brueckner.com/plantuml-how-to/) · [Templates](../templates/) · [PlantUML vs. Mermaid](../plantuml-vs-mermaid/)

---

## Table of Contents

- [Include and Library Errors](#include-and-library-errors)
- [Size and Rendering Limits](#size-and-rendering-limits)
- [Layout Problems (linetype, skinparam, direction)](#layout-problems-linetype-skinparam-direction)
- [Export Quality](#export-quality)
- [Theme and Styling Conflicts](#theme-and-styling-conflicts)
- [Graphviz Problems](#graphviz-problems)
- [Editor and Tooling Issues](#editor-and-tooling-issues)
- [ArchiMate-Specific Traps](#archimate-specific-traps)

---

## Include and Library Errors

**Symptom:** `!include <archimate/Archimate>` fails with "cannot include" or renders an error diagram.

- **Cause 1: ancient PlantUML.** The ArchiMate library ships inside the PlantUML standard library, which ships inside the jar. A jar from the early 2020s predates parts of it. Fix: download the current jar from [plantuml.com/download](https://plantuml.com/download). No separate library install exists — if a tutorial tells you to download ArchiMate sprites manually, it is outdated.
- **Cause 2: someone set `plantuml.include.path` and shadowed the stdlib.** Remove the custom include path or add the stdlib location to it.
- **Cause 3: online editor with an old backend.** Some web editors pin an old PlantUML. Try [Kroki.io](https://kroki.io/) or the official server to isolate whether the problem is your code or their backend.

**Symptom:** `!include` of your own local file fails.

- Relative includes resolve against the *current working directory of the PlantUML process*, not necessarily the diagram file. In CI and VS Code these differ. Fix: use absolute paths in CI, or keep includes next to the entry diagram and invoke PlantUML from that directory.

---

## Size and Rendering Limits

**Symptom:** "Your diagram is too large" or a truncated image on the public online server.

- The hosted PlantUML server caps image dimensions. This is the limit people hit first with real ArchiMate models.
- **Fix A (best): render locally.** The jar has no size limit worth mentioning. `java -jar plantuml.jar diagram.puml`
- **Fix B: Kroki.** Higher limits, same text in, image out.
- **Fix C: split the diagram.** If you hit the cap, the diagram was answering too many questions anyway. One view per stakeholder question.

**Symptom:** rendering takes forever or the Java process runs out of memory.

- Large diagrams need heap: `java -Xmx2048m -jar plantuml.jar diagram.puml`. If 2 GB is not enough, see Fix C above. Seriously.

---

## Layout Problems (linetype, skinparam, direction)

These are the most searched-for PlantUML fixes, and the syntax is less obvious than it should be.

**Symptom:** arrows cross everything; the diagram is a plate of spaghetti.

The baseline trio, in this order, directly after your includes:

```plantuml
left to right direction
skinparam linetype ortho
```

- `left to right direction` switches the main flow from top-down to left-right. Wide diagrams (landscapes, migration views) almost always read better this way.
- `skinparam linetype ortho` forces right-angle connectors. The diagram instantly looks like architecture instead of pasta.
- `skinparam linetype polyline` is the middle ground: angled but simplified lines. Try it when `ortho` produces awkward detours. Note: `ortho` and `polyline` are mutually exclusive — the last one wins.

**Symptom:** `linetype ortho` does nothing / throws an error.

- Ortho requires Graphviz for most diagram types. If Graphviz is missing or too old, PlantUML silently falls back or fails. Verify with `java -jar plantuml.jar -testdot` — it prints the detected Graphviz version or the problem.
- Some diagram types ignore linetype entirely. For ArchiMate/component-style diagrams it works; for pure sequence diagrams it does not apply.

**Symptom:** two elements insist on sitting next to each other (or refuse to).

- Steering arrows: every ArchiMate relationship macro accepts direction suffixes `_Up`, `_Down`, `_Left`, `_Right`: `Rel_Flow_Right(a, b, "label")` hints a left-to-right flow. For plain arrows: `a -down-> b`, `a -right-> b`.
- `[hidden]` links create ranking constraints without visible arrows: `a -[hidden]-> b` pulls b near a.
- **The honest rule:** if you need more than three or four manual hints, the layout engine is telling you the diagram wants to be two diagrams. Listen to it.

**Symptom:** `skinparam` seems to have no effect.

- Order matters. `skinparam` after `!theme` overrides the theme; before it, the theme wins. Put your skinparams *after* includes and themes.
- Typos fail silently. `skinparam` with an unknown property renders fine and changes nothing. If nothing happens, suspect the spelling before anything else.

---

## Export Quality

**Symptom:** blurry diagrams in PowerPoint or print.

- `skinparam dpi 300` before rendering PNG. Screen DPI (96) is why it looks fine on your monitor and terrible on paper.
- Better: export **SVG** (`plantuml -tsvg`) and insert the vector file. Scales forever, zero DPI discussions.

**Symptom:** SVG looks wrong in some viewers (fonts, missing glyphs).

- Embed fonts or convert text to paths depending on your toolchain. Simplest robust path for decks: render PNG at 300 DPI instead of fighting SVG font embedding.

---

## Theme and Styling Conflicts

**Symptom:** ArchiMate shapes disappear or look wrong after adding a theme.

- Some themes redefine sprites and colors in ways that break ArchiMate element rendering. The stable combination:

```plantuml
!theme plain
!global $ARCH_SPECIAL_SHAPES = %true()
```

- `$ARCH_SPECIAL_SHAPES = %true()` enables the proper ArchiMate glyphs (the notched corners, the service circles). Without it you get generic boxes and wonder why your diagram does not look like the examples.
- Apply custom `skinparam` styling **after** the includes and theme, so yours win.

---

## Graphviz Problems

**Symptom:** weird layout, "dot: command not found" in logs, or `-testdot` fails.

- PlantUML delegates most layout to Graphviz `dot`. Smetana (the pure-Java layout engine) is an alternative: `!pragma layout smetana` at the top of the diagram removes the Graphviz dependency entirely — handy in minimal CI containers.
- If you install Graphviz, restart your editor/preview; the detection happens at startup.
- Graphviz too new or too old can both misbehave. `-testdot` output tells you what PlantUML actually found — trust it over what you think is installed.

---

## Editor and Tooling Issues

**Symptom:** VS Code extension shows nothing / stale preview.

- The extension uses its own bundled PlantUML or a configured server. If your diagram uses current stdlib features and the preview fails, the extension's PlantUML is old. Point it at a current local jar in the extension settings, or at a Kroki/PlantUML server URL you control.
- Preview not updating: save the file (preview renders on save in most configurations), then reload the window before suspecting your syntax.

**Symptom:** works locally, fails in CI.

- Almost always a working-directory problem with includes (see above) or a missing Graphviz in the CI image. `-testdot` in a CI step answers this in seconds.
- Headless servers sometimes need font packages installed (`fonts-dejavu` or similar) or text renders as boxes.

---

## ArchiMate-Specific Traps

**Symptom:** relationship macros render but the model is semantically wrong.

- PlantUML checks syntax, not ArchiMate semantics. `Rel_Serving(database, user)` renders beautifully and means the database serves the user — probably not your intent. Serving points from provider to consumer. See the [Relationship Cheat-Sheet](../#archimate-relationship-cheat-sheet) in the main guide.

**Symptom:** `Grouping` nesting breaks the layout.

- Deep nesting (3+ levels) reliably confuses the layout engine. Flatten: prefer two levels and good names over a full containment hierarchy in one view.

**Symptom:** multiline labels (`\n`) shift elements into odd positions.

- Long labels are layout ballast. Shorten the label, move detail into a `note`, and let the structure breathe.

---

## Still Stuck?

1. `java -jar plantuml.jar -version` and `-testdot` — know your actual versions.
2. Reduce to the smallest failing diagram (usually 5 lines).
3. Check [plantuml.com](https://plantuml.com) docs and the [PlantUML Q&A forum](https://forum.plantuml.net/) — the error message in quotes finds most answers.

---

**Last Updated:** August 2026
