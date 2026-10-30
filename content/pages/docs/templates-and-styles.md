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

Settings ▸ General ▸ **Templates and Styles** (in the Design systems group) lists both folders: each one's path with
Reveal in Finder (creating the folder if it is missing), your entries, a row for each file that does not load with the
reason (the same warning `silverpoint templates` and `silverpoint styles` print), then the built-in entries: the Start from
templates and every bundled style preset. A style of yours with a bundled preset's name says it replaces it. The section is
read-only (edit the folders in Finder); it reads them again when the Settings window shows or becomes key, and on Reload.

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
faces, tags and text; without one it returns one line per pairing, `slug — summary [moods; uses]`, filtered by `mood`
and `use` (each matched whole, ignoring case, both at once when both are given). No match returns a line starting
`No font pairing matches` with both vocabularies; an unknown name is a tool error that lists the pairings. It is in the
shared catalog, so the agent, MCP clients and `silverpoint call <doc> get_font_pairing '{"mood":"calm"}'` all get it.

Each is a Markdown file in `Resources/Pairings` with this front matter:

- `title:` and `summary:`, as for a style;
- `heading:` and `body:`, the two families, and an optional `mono:`;
- `moods:`, a comma list from editorial, calm, technical, playful, luxurious, friendly, formal, bold;
- `uses:`, a comma list from app UI, marketing, long reading, data, slides.

The text after it opens with `# <title>` and a line naming the faces, then `## Sizes and weights` (point sizes and
weights that work) and `## Avoid`. Fourteen pairings are bundled, among them `new-york-sf-pro` (editorial),
`sf-pro-sf-mono` (technical) and `sf-pro-rounded` (friendly). Every face in them is one of the fonts that ship with macOS: SF Pro, SF Pro
Rounded, SF Mono and New York are the system designs (SF Pro Rounded and New York draw as the system font's rounded and
serif designs when they are not installed by name), and the others are installed families such as Didot, Avenir Next,
Charter and Helvetica Neue. A test over the bundled data checks that each one resolves on the Mac running it.

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
filter matches whatever a pairing says. The faces are not checked: name families that are installed, or the design falls
back to the system font. `get_font_pairing` resolves the layers when it runs, so a file added to the folder between turns is there
on the next call; its description and `name` choices list the pairings as they were when the tool list was built. The
agent, MCP (`silverpoint mcp`) and `silverpoint call` all read the same folder.

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
an error that names both. `silverpoint styles --product <slug>` prints the same text, and `silverpoint styles --products`
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
naming the file and the roles, and does not claim its family, so the bundled or package palette still applies. Only the
structure is checked: the contrast checks the bundled palettes pass are not run on yours. The warnings show on
`silverpoint styles --product` and `--products` (stderr); the agent and MCP skip what does not load.

A product may name a family that has no palette in any layer: its direction then ends with `Palette (<family>): no palette
for this family.` A preset or pairing it names that is not in the resolved styles or pairings is left out of its list,
which ends with one `(not found, left out: <slug>, <slug>)` line. `get_style` resolves the layers when it runs, so a row
added between turns is there on the next call; its `product` choices list the product types as they were when the tool
list was built, and the unknown-product error lists the resolved families and their products. The agent, MCP
(`silverpoint mcp`), `silverpoint call` and `silverpoint styles` all read the same folder.

## Command line

    silverpoint styles [<file.silverpoint>] [--json] [--show <name>]
    silverpoint styles --product <slug> [--json]
    silverpoint styles --products [--json]
    silverpoint templates [--json]

`styles` lists one line per resolved style (with a document, those of the packages it links too),
`slug<TAB>source[ (overrides bundled)]<TAB>title`, or with `--json` an array
of `{slug, title, summary, source, overrides}`; `--show <name>` prints the text `get_style` would return (exit 1 for an
unknown name). `--product <slug>` prints what `get_style` with `product` returns (with `--json`, `{slug, title, family,
body}`; an unknown product exits 1 with the tool's error), and `--products` lists the product types, `slug<TAB>family<TAB>title`
by family and then slug (with `--json`, an array of `{slug, title, family}`); beside either, the only flag is `--json`.
`templates` lists the user templates, `name<TAB>absolute path` per line, or with `--json` an array of
`{name, file}`; a missing folder lists nothing and exits 0. Warnings go to stderr. Both read only, honor
`SILVERPOINT_STYLES_DIR` and `SILVERPOINT_TEMPLATES_DIR` (and `--product` and `--products` the products folder,
`SILVERPOINT_PRODUCTS_DIR`), and exit 2 for a flag outside their set.

The old `VELLUM_` spelling of these variables still works, like every `SILVERPOINT_*` variable.
