---
title: "User templates and style presets"
description: "Silverpoint ships starter templates (the Launch window's Start from cards, File ▸ New from Template) and style presets (what the agent's getstyle serves). Both can be extended with your own, in folders you control. Skills work the same way (see skills.md)."
summary: "Silverpoint ships starter templates (the Launch window's Start from cards, File ▸ New from Template) and style presets (what the agent's getstyle serves). Both can be extended with your own, in folders you control. Skills work the same way (see skills.md)."
layout: doc
toc: true
---

Silverpoint ships starter **templates** (the Launch window's Start from cards, File ▸ New from Template) and **style
presets** (what the agent's `get_style` serves). Both can be extended with your own, in folders you control. Skills work
the same way (see [skills.md](../skills/)).

## Templates

A template is a design you start new documents from.

The bundled starters are Blank, iPhone app, Web page and Slides (the Home window's Start from row), and Design system,
Diagram and Sample card (the Templates section and File ▸ New from Template). **Diagram** (`Resources/Templates/diagram.json`)
is set up the way the `diagrams` skill draws: the Diagram Minimal palette as variables with light and dark values
(`background`, `surface`, `ink`, `muted`, `border`, `accent`, `chart-1` … `chart-6`), the `Title`, `Node`, `Label` and
`Legend` text styles, components for each node kind (Process, Decision, Terminal, Data, Actor, Note, Group) and a Legend,
and a Checkout flow example: seven node instances joined by attached connectors and arranged by `layout_diagram`, with a
callout and the legend under it. Node outlines are `$muted` rather than the presets' `$border` hairline, which is under the
contrast lint's 3:1 for a box's edge. The file is generated: `DiagramTemplateBuilder` (SilverpointAgentTests) builds it
through the real operations, `DiagramTemplateTests` checks the committed file equals its output, and
`SILVERPOINT_BLESS=1` rewrites it.

### Where they live

| What | Where | Moved with |
| --- | --- | --- |
| Templates | `~/Library/Application Support/Silverpoint/Templates` | `SILVERPOINT_TEMPLATES_DIR` |

The folder holds `*.silverpoint` packages and `*.json` documents (a `document.json` on its own: no images). The app and
`silverpoint templates` read the same folder. Hidden files, subfolders and other extensions are ignored. A file that
cannot be read as a Silverpoint document is skipped and reported as a warning (on stderr in `silverpoint templates`),
never a crash.

**The name** is the file name without its extension (`Phone App.json` is "Phone App"). A document has no title field of
its own, so the file name it is. Templates are listed sorted by name, ignoring case. If `Hero.silverpoint` and
`Hero.json` are both there, they are one template: the package wins and the `.json` is a warning.

### Where they show up

- **Home window:** the Start from row shows the bundled cards, then your templates under a "Your templates" label (only
  when there are any; the row scrolls sideways when there are many). Each card has a thumbnail, drawn by the same renderer
  and cache as the Recent cards. The Templates section of the sidebar shows them too.
- **File ▸ New from Template:** a "Your Templates" section after the bundled ones. The menu is read each time it opens.

Clicking a card, or choosing the menu item, opens a **new untitled document** with the template's content. The document
is a copy: it has no file, so saving asks where to put it, and nothing you do to it can change the template. A
`.silverpoint` template brings its images along.

### Save as Template…

File ▸ **Save as Template…** asks for a name and saves the current document into the templates folder as
`<name>.silverpoint`. When the selection holds top-level frames there is an **Only the selected frames** checkbox: the
template then has just those frames, and keeps what they lean on (the document's themes, variables, text and layer
styles, the top-level components, and the images the kept layers use). It drops comments and import origins, which belong
to the whole canvas.

If a template with that name is already there (either extension, ignoring case) Silverpoint **asks before replacing it**.
Names are made safe for a file name: slashes, backslashes and colons become dashes, and leading or trailing dots, dashes
and spaces are trimmed, so a template is always a direct child of the templates folder (`../Evil` saves as `Evil`). A
name with no letters or digits is refused.

### Managing

File ▸ **Reveal Templates in Finder** opens the folder (creating it first if it is missing). **Remove…** in a card's
context menu asks first, then moves the file to the **Trash** (it is not deleted). The card's menu also has Open and
Reveal in Finder.

Settings ▸ General ▸ **Templates and Styles** (in the Design systems group) lists your templates and styles folders, and
your font pairings and products folders (the section keeps its name: pairings and palettes are part of a style): each
one's path with Reveal in Finder (creating the folder, and the products folder's `palettes/`, if missing), your entries,
and a row for each file that does not load with the reason (the same warning `silverpoint templates` and `silverpoint
styles` print). Templates and styles then list the built-in entries: the Start from templates and every bundled style
preset. **Font pairings** counts yours and the built-in ones (`2 yours · 35 built-in`, and `· 1 not loaded`) and lists
yours by slug. **Palettes and product types** does the same for the palettes in `palettes/` and the product types in the
folder's tables (a bad table row is its own row, named by file and line), and under each of your palettes lists the
[contrast checks](#contrast-checks-on-your-palettes) it fails: the pair, the mode and the ratio against the minimum
(`Low contrast: onAccent on accent, dark, 2.31:1 (needs 4.5:1)`). An entry of yours with a bundled one's name says it
replaces it. The section is read-only (edit the folders in Finder); it reads them again, off the main actor, when the
Settings window shows or becomes key, and on Reload. Its model (`TemplatesStylesSettingsModel`) reads only the folders it
is given: the app passes the standard ones (`TemplatesStylesSettingsModel.standard()`), and tests pass temporary ones.

## Style presets

A style preset is a Markdown file with the same format as a skill (`title:` and `summary:` front matter, then the
guide): fonts, colors, type scale, shapes. `get_style` loads one.

Eleven presets are bundled. Eight are for pages and screens (`bold-brutalist`, `corporate-clean`, `dark-tech-glow`,
`editorial-serif`, `luxury-noir`, `soft-pastel`, `swiss-minimal`, `warm-organic`) and three are for diagrams, the ones the
`diagrams` skill names:

| Preset | Look | Fonts |
| --- | --- | --- |
| `diagram-minimal-light` | white nodes on warm paper, one rust accent | SF Pro for display and body, SF Mono for technical text |
| `diagram-minimal-dark` | lighter surfaces on near-black, one amber accent | SF Pro for display and body, SF Mono for technical text |
| `diagram-editorial` | a serif title over the diagram on cream, one ink-blue accent | Charter for display, SF Pro body, SF Mono |

**Page preset colors: full roles, light and dark.** Each of the eight page presets defines the same color roles, each with a
light and a dark value designed for that preset (not an inversion):

| Role | Used for |
| --- | --- |
| `background` | the page |
| `surface` | cards, panels, fields |
| `ink` | the surface foreground: headings and body text on background and surface |
| `muted` | the muted foreground: captions, labels, placeholders, and control and input outlines |
| `border` | hairlines and dividers |
| `accent` | primary buttons, links, highlights |
| `onAccent` | text and icons on `accent` |
| `destructive` | delete buttons and error text |
| `onDestructive` | text and icons on `destructive` |
| `focus` | the focus ring |

Extra roles a preset already had (`accent2`, `accent3`, `success`, soft-pastel's four `pastels`) stay, with dark values too.
A color line reads `- accent: #RRGGBB / #RRGGBB`, the two values in the order the heading names: `## Colors (mode: light /
dark; define these with set_variables)`, or `mode: dark / light` for the two presets that are dark by nature
(`dark-tech-glow`, `luxury-noir`), whose first value stays their own look. Apply them as variables with a `mode` theme axis;
the first value is plain (the default mode) and the second is tagged with the other mode. Each preset's Colors section
carries one example line, which `get_style` returns with the rest of the preset:

```json
{"op":"set_variables","variables":{"accent":{"type":"color","value":[{"value":"#7A1E2C"},{"value":"#E07A8A","theme":{"mode":"dark"}}]}},"themes":{"mode":["light","dark"]}}
```

Every pair passes contrast in both modes (`BundledStyleDataTests`, and the `contrast` lint, which checks each theme mode):
`ink` on background and surface, `muted`, `destructive` (it is error text) on background, `onAccent` on accent and
`onDestructive` on destructive at 4.5:1; `accent` and `focus` on background, and `focus` on surface, at 3:1. The
border is decorative and has no check: hairlines between areas that already differ carry no meaning, so a control's
outline uses `muted`, which passes 3:1 and more. save_style accepts the same light / dark form: it writes the body as it is given.

Each diagram preset defines the six palette variables (`background`, `surface`, `ink`, `muted`, `border`, `accent`) in one mode (the light and dark looks are separate presets), a small type scale,
a radius of 10 or less and a 4 pt spacing unit, and its notes ask for 1 pt hairlines, no shadows, one accent and a
horizontal legend. The fonts are installed on macOS and offered by `list_fonts`: SF Pro and SF Mono stand in for Geist and Geist Mono, and Charter for Instrument Serif, the faces diagram-design uses (neither ships with macOS).

| Layer | Where | Edited by |
| --- | --- | --- |
| Bundled | inside the app (`Resources/Styles`) | not editable; override it by saving a style with the same name |
| Package | `styles/*.md` of each design system package the document links (`docs/design-system-packages.md`) | the package's author |
| User | `~/Library/Application Support/Silverpoint/Styles/*.md` (`SILVERPOINT_STYLES_DIR`) | you; the agent's `save_style` |

**Precedence:** a user style beats a package style, which beats a bundled one with the same name (the name is the file name
without `.md`, lowercased, with runs of anything but letters and digits turned into one dash, as for skills). A resolved style
records its source and, when it replaces a lower layer's, says so: `silverpoint styles` prints `user (overrides bundled)` or
`package (overrides bundled)`. Package styles are in the list only while the document links the package (`silverpoint styles
<doc>`, `get_style` and the agent's prompt for that document). Two files in the folder with one name: the first file name in
byte order wins, the other is a warning.

**There is no document layer for styles**, and no disabled list. A document carries its look in its variables, themes
and text styles, and the agent already reads those; a style is a starting point for a design that has none. (Both are
possible later steps; see the backlog.)

`get_style`, its description (an index of every resolved style, also the `name` choices), the agent's system prompt
("Styles available through get_style"), MCP clients and the CLI all see the resolved list. It is read when a tool runs,
so a style saved between turns is found at once.

### Saving a style: `save_style`

"Save this look as a style called Editorial": the agent calls `save_style` with `name`, `title`, `summary`, `body` and
`confirm`. It is one tool in the shared catalog, with the same confirm flow as `save_skill`:

- **In the app** it never writes. A card shows the text (or a diff against the style it replaces) with Save and Discard;
  Save writes `<name>.md` into your styles folder.
- **Over MCP and `silverpoint call`** without `confirm: true` it returns a preview (`{written: false, slug, title,
  replaces, text, diff}`) and writes nothing; with it, it writes and returns `written: true`. `replaces` is `user` or
  `bundled` when the style replaces one.
- An empty body or a name with no letters or digits is a tool error and writes nothing. So is a setup with no styles
  folder (a library caller that did not pass one).

Saving over a style that came from a file with another name (`Neon Look.md`) rewrites that file, so the saved version is
the one that is read.

## Font pairings

A font pairing is a heading face and a body face (sometimes a mono face too), kept apart from the style presets so a
style's palette can take a different type voice. `get_font_pairing` serves them: with `name` it returns the pairing's
faces, tags, where its faces come from (`fonts: installed with macOS`, or `fonts: library (…)` with its fallback) and
text; without one it returns one line per pairing, `slug — summary [moods; uses]`, a library pairing's line ending
` (library: <families>; fetched when added)`, filtered by `mood`, `use` and `fonts` (`installed` or `library`; each
matched whole, ignoring case, all at once when several are given). The installed pairings always come first, then the
library ones, so a design depends on the network only when a library pairing is chosen. No match returns a line starting
`No font pairing matches` with the vocabularies; an unknown name is a tool error that lists the pairings. It is in the
shared catalog, so the agent, MCP clients and `silverpoint call <doc> get_font_pairing '{"mood":"calm"}'` all get it, and
`silverpoint styles --pairings [--fonts installed|library] [--json]` lists them (see Command line).

Each is a Markdown file in `Resources/Pairings` with this front matter:

- `title:` and `summary:`, as for a style;
- `heading:` and `body:`, the two families, and an optional `mono:`;
- `moods:`, a comma list from editorial, calm, technical, playful, luxurious, friendly, formal, bold;
- `uses:`, a comma list from app UI, marketing, long reading, data, slides;
- `fonts:`, `library` when the faces come from the font library (anything else, or no line, is `installed`);
- for a library pairing, `fallback:`, the installed pairing to use when its faces cannot be fetched, and `products:`, the
  product types (slugs from `products.md`) it is offered for.

The text after it opens with `# <title>` and a line naming the faces, then `## Sizes and weights` (point sizes and
weights that work) and `## Avoid`. Fourteen installed pairings are bundled, among them `new-york-sf-pro` (editorial),
`sf-pro-sf-mono` (technical) and `sf-pro-rounded` (friendly). Every face in them is one of the fonts that ship with macOS: SF Pro, SF Pro
Rounded, SF Mono and New York are the system designs (SF Pro Rounded and New York draw as the system font's rounded and
serif designs when they are not installed by name), and the others are installed families such as Didot, Avenir Next,
Charter and Helvetica Neue. A test over the bundled data checks that each one resolves on the Mac running it.

### Library pairings

Twenty-one more bundled pairings (`fonts: library`) are made of font library families (see The font library below), our
own picks, each with a line on why the faces work together and the weights to use:

| Pairing | Faces | For |
| --- | --- | --- |
| `source-serif-4-source-sans-3` | Source Serif 4, Source Sans 3, Source Code Pro | docs and help: one system, shared proportions |
| `ibm-plex-serif-ibm-plex-sans` | IBM Plex Serif, IBM Plex Sans, IBM Plex Mono | reports and finance: one institution, three registers |
| `ibm-plex-sans-ibm-plex-mono` | IBM Plex Sans, IBM Plex Mono | dense dashboards whose figures line up |
| `geist-geist-mono` | Geist, Geist Mono | developer tools: one grid for text and commands |
| `space-grotesk-inter` | Space Grotesk, Inter, JetBrains Mono | tech launches: a quirky headline, a neutral text |
| `fraunces-figtree` | Fraunces, Figtree | warm, appetizing food and everyday apps |
| `playfair-display-source-sans-3` | Playfair Display, Source Sans 3 | glossy magazine and fashion headlines |
| `newsreader-public-sans` | Newsreader, Public Sans | news: a serif made for reporting, a civic sans around it |
| `instrument-sans-literata` | Instrument Sans, Literata | reading apps and editors: modern chrome, a book face |
| `instrument-serif-instrument-sans` | Instrument Serif, Instrument Sans | contemporary editorial: a condensed display serif and its sans |
| `cormorant-garamond-manrope` | Cormorant Garamond, Manrope | hushed luxury and hospitality |
| `dm-serif-display-dm-sans` | DM Serif Display, DM Sans | boutique commerce: one project's serif and sans |
| `bricolage-grotesque-dm-sans` | Bricolage Grotesque, DM Sans | loud, contemporary marketing |
| `syne-manrope` | Syne, Manrope | studios, galleries and culture brands |
| `bebas-neue-work-sans` | Bebas Neue, Work Sans | training, sport and event posters |
| `unbounded-onest` | Unbounded, Onest | festivals, music and streaming |
| `abril-fatface-lora` | Abril Fatface, Lora | menus, recipes and food writing |
| `nunito-lexend` | Nunito, Lexend | kids' learning: rounded titles, a sans built for early readers |
| `caveat-figtree` | Caveat, Figtree | journals and habit tools: handwritten accents |
| `noto-serif-jp-noto-sans-jp` | Noto Serif JP, Noto Sans JP | Japanese editorial: Mincho headings over a Gothic body |
| `m-plus-1-noto-sans-jp` | M PLUS 1, Noto Sans JP | friendly Japanese app UI |

**The rule.** A library pairing's faces must be in the document before any text uses them. `add_library_font` with
`{"pairing": "<slug>"}` (instead of `family`) fetches every library face the pairing names that the document does not
already carry, and only when all of them arrived carries them, with each family's OFL license, as one undo step (Add Font
Pairing); then text styles may name them. When one cannot be fetched (offline, no library in this context, a file that does
not match), nothing is added and the answer starts `Fallback:`: it gives the reason, names the installed pairing to use
instead (the pairing's `fallback`, else an installed pairing that shares a mood) with that pairing's details, and tells the
agent not to name the library families, so the design never carries a `missing-font` warning. An installed pairing has
nothing to add. `design-principles` and the schema skill say so, and `get_font_pairing`'s details for a library pairing
repeat the call to make. Tests over the bundled data (`LibraryPairingTests`) check that every family each library pairing
names is in `library.json`, spelled as the catalog spells it, at weights (and italics) its files have, that each has an
installed fallback and real products, and that every product type is offered at least one.

### Your own pairings

Pairings are layered like styles, by the same resolver, and matched by slug (the file name, as for a style):

| Layer | Where | Wins over |
| --- | --- | --- |
| User | `~/Library/Application Support/Silverpoint/Pairings/*.md` (`SILVERPOINT_PAIRINGS_DIR`) | package, bundled |
| Package | a linked design-system package's `pairings/*.md` (see `design-system-packages.md`) | bundled |
| Bundled | `Resources/Pairings/*.md` | |

A package's pairings are there only while the document links the package; when two linked packages define one slug, the
first wins. A file in the user folder or a package's `pairings/` has the bundled front matter and must name `title`,
`summary`, `heading` and `body`; a file that lacks one, has no front matter or is not UTF-8 text is skipped with a warning
that names it (two files with one slug: the first filename in byte order wins and the other is a warning), and the rest
still load. Its moods and uses are free words: the bundled vocabularies above bind only the bundled pairings, and a
filter matches whatever a pairing says. The faces of a file you write yourself are not checked (`save_font_pairing`
checks them): name families that are installed, or the design falls
back to the system font, or font library families with `fonts: library` (and a `fallback:`), which `add_library_font`
with `pairing` carries like a bundled library pairing's. `get_font_pairing` resolves the layers when it runs, so a file added to the folder between turns is there
on the next call; its description and `name` choices list the pairings as they were when the tool list was built. The
agent, MCP (`silverpoint mcp`) and `silverpoint call` all read the same folder.

### Saving a pairing: `save_font_pairing`

"Save these fonts as a pairing": the agent calls `save_font_pairing` with `name`, `title`, `summary`, `heading`, `body`,
an optional `mono`, `moods` and `uses` (lists), `notes` (the Markdown after the front matter: sizes and weights per role,
what to avoid), and for a library pairing `fonts: "library"`, a `fallback` and the `products` it suits. It writes
`<slug>.md` into your pairings folder in the bundled form (front matter, then `# <title>` and the notes), which
`get_font_pairing` serves at once. It is one tool in the shared catalog (the agent, MCP, `silverpoint call`), with
`save_style`'s confirm flow and its rule for an existing name:

- **In the app** it never writes. The same card as a style's shows the file (or a diff against the pairing it replaces)
  with Save and Discard; Save writes it.
- **Over MCP and `silverpoint call`** without `confirm: true` it returns a preview (`{written: false, slug, title,
  replaces, text, diff}`) and writes nothing; with it, it writes and returns `written: true`.
- **A bundled name** is allowed, like a style's: your file then replaces the bundled pairing for every document (the
  bundled file is never touched). It is never silent: the preview and the card say `replaces: bundled` (or `package`,
  `user`) with a diff, before anything is written. A pairing of yours that came from a file with another name
  (`My Pair.md`) is rewritten in place.
- **Refused, with the reason, writing nothing:** a name with no letters or digits; an empty title, summary, heading,
  body, moods, uses or notes; `fonts` other than `installed` or `library`; an installed pairing naming a family that is not
  installed on this Mac (one in the font library says to use `fonts: "library"`); a library pairing naming a family that
  is neither in the font library nor installed; a `fallback` that is not an installed pairing; a product that is not a
  product type; and a setup with no pairings folder.

### Fonts in the document

A document can carry its own font files, so a family that is not installed on this Mac still draws, here and on the next
Mac that opens the file. In the text inspector, the Font family menu lists the document's fonts in their own section, In
This Document, above the installed families; **Add Font File…** opens a panel for TrueType (`.ttf`), OpenType (`.otf`) and
collection (`.ttc`) files and carries the chosen one (one undo step, Add Font), with the license file beside it when there
is one (`OFL.txt`, `LICENSE.txt`, `<family>-OFL.txt` and similar); **Remove Font** stops carrying one (Remove Font, undone
like any edit). Only a file CoreText parses as a font is taken, up to 50 MB; anything else is refused with a message, and
nothing in the file is executed. Adding a font is the person's action: no agent or MCP tool reads a path to add one.

Each font is an entry in `document.json`'s `fonts` (an id, the family, style, weight and italic read from the file, its
PostScript name, the file's name in the package's `fonts/` folder, an optional license file and the source: a file you added,
or a library font by id and version), and its file sits in `fonts/` beside `assets/`. A save writes only the files the
entries name, so a removed font's file goes with the next save; the version history keeps the files its versions used, so
restoring a version brings its fonts back. A document with fonts declares the `embeddedFonts` feature, so an older
Silverpoint opens it read-only instead of dropping them; a document without fonts is unchanged.

While a document is open its fonts are registered for Silverpoint's process only (CoreText's process scope, from a copy in a
temporary folder), so the canvas, text metrics, rendering, screenshots and PNG, PDF and SVG export draw them; they are
never installed into your font folders, and closing the last document that uses one unregisters it. `silverpoint render`,
`score`, `call` and `mcp` register the fonts of the document they open the same way. **An installed font always wins**: a
file whose family or PostScript name an installed font already has is not registered, the installed font draws, and the
menu and `list_fonts` say so. `list_fonts` lists the document's fonts first, each with its file, source, license and whether
it is registered, then the installed families. The built-in lint rule `missing-font` names each text and text style whose
family is neither installed nor carried by the document (rendering still falls back to the system font); `fix_lint` leaves
it to you, since which font to use is a choice.

### The font library

Font family ▸ **Google Fonts…** in the text inspector opens the font library: 56 curated families from
[Google Fonts](https://fonts.google.com) a designer reaches for, by category (sans serif, serif, display, monospace,
handwriting) with a few mood and use tags each (`ui`, `editorial`, `poster`, `code`, `japanese`), among them Inter, Roboto
Flex, DM Sans, Manrope, Space Grotesk, IBM Plex Sans, Fraunces, Source Serif 4, Playfair Display, Newsreader, EB Garamond,
Bricolage Grotesque, Syne, Bebas Neue, JetBrains Mono, IBM Plex Mono, Fira Code, Caveat, and for Japanese Noto Sans JP
(9.6 MB), Noto Serif JP (13.6 MB) and M PLUS 1 (4.2 MB). Each row previews the family's name in its own face once its
files are at hand (in the document, or cached), says whether it is not downloaded, downloading, downloaded (cached) or in
this document, and has **Add**: the family's files and its license are fetched, checked, and carried in the document
through the machinery above (one undo step, Add Font Family), and the selected text takes the family. A failure (offline,
a file that does not match) says so and changes nothing.

The catalog is bundled and pinned: `Packages/SilverpointKit/Sources/SilverpointAgent/Resources/Fonts/library.json` lists
each family's name, category, tags, its `ofl/<family>/OFL.txt` license, and per file its path in the
[google/fonts](https://github.com/google/fonts) repository at commit `6c1dbe27dcf7d817d4b62cec033ba77fa8e2a5ec`, its style
(roman or italic), weight range and axes, byte size and SHA-256. Variable files are preferred (one or two files cover every
weight); a family with only static files lists each. Only families under `ofl/` (the SIL Open Font License) are in it,
nothing from `apache/` or `ufl/`. `Scripts/fonts/build-library-catalog.py` generates it from the curated list in the
script (re-run it after changing the list or the commit, with network access, then `FILTER=FontLibraryToolTests make test`
checks the catalog's shape).

**Fetched only on demand.** Nothing is downloaded at launch, in the background, or to list or search the library: a request
is made only when a family is added (the browser's Add, `add_library_font`, `silverpoint fonts add`), straight from
`https://raw.githubusercontent.com/google/fonts/<commit>/<path>`, with no API key, no fonts.googleapis.com and no other
request. Each file's size and SHA-256 must match the catalog before it is used (a mismatch is an error and the bytes are
discarded), and a font file must parse with CoreText. Verified files are cached in `~/Library/Caches/Silverpoint/Fonts`
(`SILVERPOINT_FONT_CACHE_DIR` overrides it), under the commit and the repository path, so the next document that adds the
family reads it from disk; a cached file is checked again before it is used. `FontLibrary` (SilverpointCore) does this
through an injected client and cache folder, both none by default; only the app's documents and the CLI's `call`, `mcp` and
`fonts` pass the standard ones, and `SILVERPOINT_FONT_LIBRARY_OFFLINE=1` leaves the client out (only cached families can be
added; the CLI tests set it).

A library family's entries in `fonts` have the source `library:<family>@<commit>` (`list_fonts` shows it), its files are
stored as `<File>-Variable.ttf` (the repository's axes list made plain) and its license as `<Family>-OFL.txt` beside them.
A family already in the document is not added twice. A variable family draws each `fontWeight` at that weight on its `wght`
axis (clamped to the family's range), so 300 through 900 each render as themselves.

For the agent, MCP and the CLI: `list_fonts` with `library: true` lists the catalog by category with tags, weights and each
family's state (`category` and `query` filter it); `add_library_font` takes a family name from it (never a URL or path) and
carries the family as one undo step. `silverpoint fonts [<query>] [--category c] [--json]` lists the catalog and whether each
family is cached (it never downloads), and `silverpoint fonts add <file.silverpoint> <family>` adds one, as does `silverpoint
call <file> add_library_font '{"family":"Inter"}'`. The schema skill and the system prompt tell the agent it may use a
library family once it is added, and never a family that is neither installed, carried nor in the library; `missing-font`
points to `add_library_font` when the library has the missing family.

**Licensing.** Every family in the library is under the SIL Open Font License 1.1, which allows using, embedding and
redistributing the fonts with documents (not selling them on their own), provided the license travels with them: so the
family's `OFL.txt` is always fetched and saved in the document with its files. The fonts are by their authors, named in
each license; Google Fonts and the google/fonts repository are where they are fetched from.

### Fonts in exports

An export that uses a font the document carries carries it too, so the result looks like the canvas without anyone
installing the font. One shared function (`FontExport.used`, SilverpointCore) finds the files an export's scope actually
uses: the exported frames' and components' text, the text styles they apply, their styled runs and instance overrides,
each matched to the carried fonts of its family (compared without case). It prefers the same slant, then a file that draws
the weight (a static file of that weight, a variable file whose `wght` range holds it), then the nearest weight. Installed
and system fonts are never matched, so never copied, and a carried font the scope does not use is not copied either; an
export whose scope uses no carried font is byte for byte what it was before. The exports read the store's fonts through
`DesignStore.carriedFonts`; library code is given none by default.

Every copied file goes with its license file: the one the document carries (`<Family>-OFL.txt` for a library family, the
license file added with a font file), or, when none was given, a `<font>-LICENSE-NOTE.txt` saying so. What each export
writes:

- **HTML** (`export_code`, Export Code): `fonts/` beside the page, and an `@font-face` per file at the top of the page's
  `<style>` (the family, the `wght` range of a variable file or its weight, the style, `font-display: swap`, and
  `src: url("fonts/<file>") format("truetype" | "opentype" | "collection")`). Every `font-family` keeps its fallback stack.
- **React**: the same `fonts/` beside `tokens.css`, with the `@font-face` rules at the top of `tokens.css`.
- **The website** (File ▸ Publish Website…, `silverpoint publish inkwell`, `export_site`): the theme's
  `assets/fonts/`, with the rules at the top of `assets/tokens.css`.
- **SwiftUI and UIKit**: `Fonts/` with the files, their licenses and a `README.md` that lists them, gives the `UIAppFonts`
  entries for an iOS app's Info.plist and the `ATSApplicationFontsPath` entry for a Mac app's, and names each face's
  PostScript name. The code loads a face by the PostScript name CoreText reports for it (`Font.custom` / `UIFont(name:)`):
  a named face of the weight (`Fraunces-Bold`), or, for a weight no named face has, the file's default face with its `wght`
  axis set (SwiftUI's generated `Font.carried(_:size:weight:)`, UIKit's `variedFont`).
- **Compose**: `res/font/` with the files under Android resource names (lowercase letters, digits and underscores:
  `fraunces_variable.ttf`), the licenses in `licenses/` (a text file is not a font resource), and `Fonts.kt`, a
  `FontFamily` per carried family with a `Font(R.font.<name>, FontWeight.…, FontStyle.…)` for each weight and style the
  texts use (a variable file's with `variationSettings` setting `wght`), which the texts name in `fontFamily`.
- **hand_off**: the target's files above, in the bundle's `code/`; `HANDOFF.md` names them.
- **SVG**: an SVG is one self-contained file whose images are embedded as `data:` URLs and whose text is live text, so a
  used carried font is embedded the same way: an `@font-face` with a `data:` URL in the SVG's `<defs>`. **PNG, JPEG and
  PDF** draw with the registered fonts, as before.

When a carried family has no face of the weight or slant a text asks for (Bebas Neue at 700, or italic), the export uses
the nearest file and reports an `embeddedFont` diagnostic naming what was approximated (the browser, the system or Compose
synthesizes the rest). Fonts from a linked design system are not exported yet (BACKLOG 66).

## Product types and palettes

A product type says what fits a kind of product before a style is picked: which page presets and font pairings suit it,
its color mood and type mood, and what to avoid ("banking: no playful rounded display type, no neon or glow accents").
The directions are our own, written from what each product is for: a habit tracker is a calm daily tool, so it gets
soft-pastel or warm-organic with friendly type, and "no landing-page layout" is one of its anti-patterns.

They are one table in `Resources/Products/products.md`, one row per product (38 of them), with these columns:

| Column | Holds |
| --- | --- |
| `slug` | the product's name for `get_style` and the CLI (`banking`, `habit-tracker`, `developer-tool`) |
| `title` | its title |
| `family` | its product family, which names its palette |
| `presets` | 1 to 3 bundled page presets that fit, a comma list |
| `pairings` | 1 to 3 font pairings that fit, a comma list |
| `color mood` | one phrase for the colors |
| `type mood` | one of the pairing moods (editorial, calm, technical, playful, luxurious, friendly, formal, bold) |
| `avoid` | 1 to 3 anti-patterns separated by `; `, with the reason where it is not obvious |

The families are `commerce`, `developer-tools`, `editorial-media`, `enterprise`, `finance`, `health-wellness`,
`kids-education`, `productivity`, `social-community` and `travel-hospitality`. Each has its own palette in
`Resources/Palettes/<family>.md`, in the page presets' form: the same ten roles as `#LIGHT / #DARK` lines under `## Colors
(mode: light / dark; define these with set_variables)`, with an `Apply as one batch_design operation:` example. The
palettes are designed for the family (none of their colors is a stock Tailwind shade, and no two share an accent) and pass
the presets' checks in both modes: the contrast pairs above, a dark mode that is not an inversion and is darker, the
example applying through the operation runner, and the `contrast` lint clean in light and dark.

`get_style` with `product` and no `name` returns the product's direction: a `Product:` line with its family, the color and
type mood, the index lines of the presets and pairings that fit (as the resolved styles and pairings give them, so a user
override's summary shows), the anti-patterns, and the family palette's Colors section as written. The product is matched
loosely (`Habit Tracker` and `HABIT_TRACKER` are `habit-tracker`). An unknown product is a tool error that lists every
family and its products; with `name`, `get_style` returns that style as before and ignores `product`; with neither it is
an error that names both. After the product's own pairings, which are all installed and stay the default, the direction
offers the library pairings whose `products` name it under `Library pairings that also fit (Google Fonts, fetched when
added; the installed pairings above stay the default):`, each index line marked; a product no library pairing names has no
such section. The style presets name no pairings, so they are unchanged. `silverpoint styles --product <slug>` prints the same text, and `silverpoint styles --products`
lists the product types by family (see Command line below). `design-principles` asks the agent to read
`get_style {product}` before choosing a style.

### Your own product types and palettes

Product types and family palettes are layered like pairings, by the same resolver: products are matched by slug and
palettes by family.

| Layer | Where | Wins over |
| --- | --- | --- |
| User | `~/Library/Application Support/Silverpoint/Products/*.md` and its `palettes/<family>.md` (`SILVERPOINT_PRODUCTS_DIR`) | package, bundled |
| Package | a linked design-system package's `products/*.md` and `palettes/*.md` (see `design-system-packages.md`) | bundled |
| Bundled | `Resources/Products/products.md` and `Resources/Palettes/<family>.md` | |

A file in the user folder (or a package's `products/`) holds one or more product tables in the bundled form, the same
header row `| slug | title | family | presets | pairings | color mood | type mood | avoid |` and its `---` line; any
number of files, read in file-name byte order and their rows in order. Slugs and families are slugified (`Pet Care` is
`pet-care`), and a family's palette is the file `palettes/<family>.md` beside the tables, in the bundled form: front matter
with `title` and `summary`, and a `## Colors` section with all ten roles as `- role: #RRGGBB / #RRGGBB` (light / dark). A
package's palettes and those of the user folder replace a lower layer's palette of the same family; a package's are there
only while the document links it, and when two linked packages define one slug or family the first wins.

Nothing in these folders is an error. A row with the wrong number of cells or an empty slug, family, presets, pairings
or avoid (title and the moods may be empty), a second row with a slug the layer already has, a row after a blank line
(which ends a table) and a file with no product table are skipped with a warning that names the file and the row's line (`skipped mine.md line 5: has 4 cells, not 8`). A
palette with no `## Colors` section, a role missing or a role not written as `#RRGGBB / #RRGGBB` is skipped with a warning
naming the file and the roles, and does not claim its family, so the bundled or package palette still applies. The
warnings show on `silverpoint styles --product` and `--products` (stderr) and in Settings; the agent and MCP skip what
does not load.

#### Contrast checks on your palettes

A palette that loads is checked against WCAG AA in light and in dark (`PaletteContrast`, SilverpointAgent), with the
ratio the `contrast` lint rule and the `lowContrast` layout problem use (`RGBA.contrast(with:)`). The pairs, each
foreground on the background it is drawn on:

| Foreground | Background | Minimum |
| --- | --- | --- |
| `ink` (surface foreground) | `background`, `surface` | 4.5:1 |
| `muted` (muted foreground) | `background`, `surface` | 4.5:1 |
| `onAccent` | `accent` | 4.5:1 |
| `onDestructive` | `destructive` | 4.5:1 |
| `destructive` (error text) | `background` | 4.5:1 |
| `accent` (a control's fill) | `background` | 3:1 |
| `focus` (the focus ring) | `background`, `surface` | 3:1 |

`border` is decorative and has no check. A failure never keeps a palette from loading: it is a warning, shown as the
pair, the mode and the ratio cut to two places (never rounded up to the minimum), `onAccent on accent, dark: 2.31:1
(needs 4.5:1)`, in three places: under the palette's row in Settings, after the palette in what `get_style` with
`product` returns (`Contrast warnings (this user palette fails WCAG AA; …)`, so the agent and MCP see it), and on stderr
from `silverpoint styles --product` and `--products` (`palette pets (user): …`). Your palettes and a linked package's are
checked; every bundled palette passes every check (`PaletteContrastTests`).

A product may name a family that has no palette in any layer: its direction then ends with `Palette (<family>): no palette
for this family.` A preset or pairing it names that is not in the resolved styles or pairings is left out of its list,
which ends with one `(not found, left out: <slug>, <slug>)` line. `get_style` resolves the layers when it runs, so a row
added between turns is there on the next call; its `product` choices list the product types as they were when the tool
list was built, and the unknown-product error lists the resolved families and their products. The agent, MCP
(`silverpoint mcp`), `silverpoint call` and `silverpoint styles` all read the same folder.

## Command line

*Not in the beta yet: the `silverpoint` command-line tool doesn't ship in the download, so this section describes it for builds made from source.*

    silverpoint styles [<file.silverpoint>] [--json] [--show <name>]
    silverpoint styles --product <slug> [--json]
    silverpoint styles --products [--json]
    silverpoint styles --pairings [--fonts installed|library] [--json]
    silverpoint templates [--json]

`styles` lists one line per resolved style (with a document, those of the packages it links too),
`slug<TAB>source[ (overrides bundled)]<TAB>title`, or with `--json` an array
of `{slug, title, summary, source, overrides}`; `--show <name>` prints the text `get_style` would return (exit 1 for an
unknown name). `--product <slug>` prints what `get_style` with `product` returns (with `--json`, `{slug, title, family,
body}`; an unknown product exits 1 with the tool's error), and `--products` lists the product types, `slug<TAB>family<TAB>title`
by family and then slug (with `--json`, an array of `{slug, title, family}`); beside either, the only flag is `--json`.
Both print, on stderr, the products folder's warnings and then each failed contrast check of your palettes and linked
packages' ([Contrast checks on your palettes](#contrast-checks-on-your-palettes)).
`--pairings` lists the font pairings `get_font_pairing` serves, `slug<TAB>layer<TAB>fonts<TAB>title` (fonts is `installed`
or `library`), installed first and then by slug, or with `--json` an array of `{slug, title, summary, source, fonts,
heading, body, mono, fallback}`; `--fonts` keeps one kind. It reads `SILVERPOINT_PAIRINGS_DIR` and never downloads.
`templates` lists the user templates, `name<TAB>absolute path` per line, or with `--json` an array of
`{name, file}`; a missing folder lists nothing and exits 0. Warnings go to stderr. Both read only, honor
`SILVERPOINT_STYLES_DIR` and `SILVERPOINT_TEMPLATES_DIR` (and `--product` and `--products` the products folder,
`SILVERPOINT_PRODUCTS_DIR`), and exit 2 for a flag outside their set.

The old `VELLUM_` spelling of these variables still works, like every `SILVERPOINT_*` variable.
