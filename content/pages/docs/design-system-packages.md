---
title: "Design systems as installable packages"
description: "A design system is a .silverpoint document of components, variables and a guide that other documents link. A design system package wraps one with a name, a semantic version and optional extras (skills, styles, font pairings, product types and palettes, lint rules), so a document can link it by id a…"
summary: "A design system is a .silverpoint document of components, variables and a guide that other documents link. A design system package wraps one with a name, a semantic version and optional extras (skills, styles, font pairings, product types and palettes, lint rules), so a document can link it by id a…"
layout: doc
toc: true
---

A design system is a `.silverpoint` document of components, variables and a guide that other documents link. A **design
system package** wraps one with a name, a semantic version and optional extras (skills, styles, font pairings, product types and palettes, lint rules), so a document can
link it **by id and version requirement** instead of by file path. Packages are installed into a per-user folder, several versions
side by side; a document that links `com.acme.ui` at `^1.2` uses the highest installed `1.x` at least 1.2 and picks up a newer
one the moment it is installed.

The path form (`imports`, a file linked by path) is unchanged and keeps working; packages are a second form beside it.

A path link is stored **relative to the document's folder** (`../ds/Brand.silverpoint`, `Brand.silverpoint`) when the design
system file is on the same volume as the document, and absolute otherwise, whichever way it was linked (the inspector's Link File…,
`link_design_system`, whose `path` may itself be relative to the document, or `silverpoint call`), so a design and its design system
can move together, live in one repository checkout or open on another Mac. Linking a file that is already linked, in either form,
keeps its alias. An unsaved document links absolute paths (it has no folder yet) and refuses a relative one; its first save makes
its links relative. Save As and a move (Move To…, the title bar, Finder) rewrite each relative link against the new folder so it
names the same file, or keep it when the design system moved along with the document; a link that no longer resolves is listed
with the other links that do not load, and absolute links are never rewritten. The inspector's Design Systems section shows each
path link as stored and, when relative, the file it resolves to. Documents with absolute links open and save unchanged.

## The package format: `.silverpointds`

A package is a **folder** whose name ends in `.silverpointds` (a package in Finder, so it double-clicks like a document):

```
com.acme.ui-1.2.0.silverpointds/
  manifest.json          required
  library.silverpoint/   required: the design system, a normal .silverpoint package
  skills/*.md            optional: skills, in the user-skills format (docs/skills.md)
  styles/*.md            optional: style presets, in the style-preset format (docs/templates-and-styles.md)
  pairings/*.md          optional: font pairings, in the font-pairing format (docs/templates-and-styles.md)
  products/*.md          optional: product type tables, in the bundled products table's format (docs/templates-and-styles.md)
  palettes/*.md          optional: product family palettes, one <family>.md each (docs/templates-and-styles.md)
  lint/*.json            optional: custom lint rules, in the lint-rules format (docs/lint-rules.md)
  README.md              optional: for people reading the package
```

A zip archive of that folder is **not** supported yet: the folder is the format (zip is listed under Left in the backlog).

### `manifest.json`

```json
{
  "id": "com.acme.ui",
  "name": "Acme UI",
  "version": "1.2.0",
  "description": "Acme's buttons, inputs and tokens",
  "author": "Acme Design",
  "homepage": "https://acme.example/ui",
  "minimumSilverpointVersion": "0.1.0"
}
```

| Field | Required | Rule |
| --- | --- | --- |
| `id` | yes | Reverse-DNS or a slug: lowercase letters and digits joined by dots or dashes (`[a-z0-9]+([.-][a-z0-9]+)*`): `com.acme.ui`, `acme-ui`. |
| `name` | yes | Text shown in the app. |
| `version` | yes | Strict semantic version, `MAJOR.MINOR.PATCH` with an optional `-prerelease` and `+build`: `1.2.0`, `2.0.0-beta.1`. `1.2` and `v1.2.0` are rejected. |
| `description` | yes | One line. |
| `author`, `homepage` | no | Text. |
| `minimumSilverpointVersion` | yes | The oldest Silverpoint that can use it (a semantic version). A package that needs a newer Silverpoint than the one running is rejected when it is installed. |

Every validation error names its field (`manifest.json: missing “version”`).

## Installing, listing and removing

Installed packages live in `~/Library/Application Support/Silverpoint/Design Systems/<id>/<version>/`. Set
`SILVERPOINT_DESIGN_SYSTEMS_DIR` to use another folder (the app, the CLI and the MCP server all read the same one).

- **In the app:** double-click a `.silverpointds` (or File ▸ Open…). Silverpoint shows its name, id and version and asks before it
  installs; Cancel writes nothing. **Settings ▸ General ▸ Packages** lists every installed version with its id, name and a
  Default badge, **Used by** (how many open designs link that version, their names in its help; closed documents are not
  checked) and **Newer version installed** when an open design uses an older version than one installed (informational: nothing
  is relinked). **Install…** copies in the `.silverpointds` you choose (choosing it is the consent; installing the same id and
  version again is refused), **Remove…** asks first and warns when an open design uses that version, **Reveal Folder** opens
  the packages folder, and **Reload** reads it again.
  Manage Design Systems… is still where each package's guide and **Use as default** are (Settings ▸ General ▸ Default design
  system ▸ **Manage…**); it shares the section's list, so a change in one shows in the other.
- **On the command line:**

```
silverpoint ds install <file.silverpointds>        installed <id> <version>
silverpoint ds list [--json]                       <id><TAB><version><TAB><name>, sorted by id, then version
silverpoint ds remove <id> [<version>]             removed <id> <version>, once per version removed
silverpoint ds outdated <file.silverpoint>         the document's package links with a newer installed version
silverpoint ds pack <library.silverpoint> --id <id> --version <semver>
                [--name <n>] [--description <d>] [--author <a>] [--homepage <url>] [--min-version <semver>]
                [--skills <dir>] [--styles <dir>] [--pairings <dir>] [--products <dir>] [--lint <dir>] [--readme <file>]
                [--out <path>]
```

Installing the same id and version twice fails (`already installed`) and changes nothing; different versions of one id sit side by
side. A failure exits 1 and a usage error exits 2.

## Packing

`silverpoint ds pack` builds the package folder from a library and the folders of extras you name (`*.md` skills, styles, font
pairings and product types, `*.json` lint rules) and prints where it wrote it (default `./<id>-<version>.silverpointds`). It never overwrites.
`--products <dir>` takes a folder shaped like the user products folder: its `*.md` tables go to `products/` and the `*.md` palettes of
its `palettes/` subfolder to `palettes/`, so `--products ~/Library/Application\ Support/Silverpoint/Products` packs yours as they are
(there is no separate `--palettes`). A folder named by a flag that does not exist is an error and nothing is written. In the app,
**File ▸ Export as Design System Package…** does the same for a document whose Type is Design System: it asks for the id, version
and name, then where to save. (File ▸ Export Design System Tokens & Code… is a different export: the tokens and component code as files.)

## Linking by package

A document stores package links in `packages`, beside `imports`:

```json
"packages": { "acme": { "id": "com.acme.ui", "version": "^1.2" } }
```

The key is the alias, as for a path link: components are `acme:btn`. Aliases are unique across `imports` and `packages`. A document
with path links only has no `packages` key and is saved byte for byte as before.

- **Link one:** the agent and MCP tool `link_design_system` takes `{"package": "com.acme.ui", "version": "^1.2", "alias": "acme"}`
  (exactly one of `path` and `package`; with no `version` it links `^` the highest installed version; a package that is not
  installed, or that no installed version satisfies, is an error and leaves the document unchanged). In the app the inspector's
  Design Systems **+** menu lists the installed packages. Settings can make a package the default for new documents
  (`com.acme.ui ^1.2`).
- **Read one:** `read_design_system` with the alias shows the package id and the version in use.

### Version requirements

| Requirement | Accepts |
| --- | --- |
| `1.2.3` or `=1.2.3` | exactly that version (a **pin**) |
| `^1.2` | `>=1.2.0 <2.0.0`; `^0.2` is `>=0.2.0 <0.3.0` (the leftmost non-zero part may not change) |
| `~1.2.3` | `>=1.2.3 <1.3.0`; `~1` is `>=1.0.0 <2.0.0` |
| `>=1.2 <2.0` | comparators `>=`, `>`, `<=`, `<`, `=` joined by spaces; all must hold |
| `1.2`, `1.x` | any patch (or minor) of that line |
| `*` | any release |

Versions in requirements may be partial. A prerelease (`1.3.0-beta.1`) satisfies a requirement only when the requirement names a
prerelease of the same `major.minor.patch`. A malformed requirement is a problem on that link, not an error that stops the document.

### Resolution and upgrades

A link resolves to the **highest installed version that satisfies its requirement**, when the document opens and again whenever
the links change or the installed packages do (the app checks every couple of seconds; the CLI resolves each time it opens a
document). Installing `1.3.0` makes a `^1.2` document use it; removing it falls back to `1.2.0`; an exact pin stays on its
version. `silverpoint ds outdated <doc>` prints `alias<TAB>id<TAB>requirement<TAB>resolved<TAB>latest` for each link whose latest
installed version is outside its requirement, and the inspector shows the same as `update available: 2.0.0` under the package id
and resolved version.

### When a package is missing

A package that is not installed, or whose requirement no installed version satisfies, is a **non-fatal problem**: the document
opens, and its instances of that library render exactly as they do when a path-linked library is missing. The problem names the
id, the requirement and why (`not installed`, or `no installed version satisfies the requirement (installed: …)`). It is shown in
the inspector, listed in the agent's design-system context, failed with by `read_design_system`, and printed by `silverpoint
render`, `call` and `mcp` as a `silverpoint: …` line on stderr (path links that do not load are reported the same way). The exit
status is otherwise unaffected.

## What the extras do

While a document links a package (and only then), its extras join the resolved lists, in their own `package` layer:

- **Skills:** bundled < **package** < user < document. A package skill with a bundled slug overrides it
  (`silverpoint skills <doc>` shows `package (overrides bundled)`); a user skill with the same slug beats the package's.
- **Styles:** bundled < **package** < user.
- **Font pairings:** bundled < **package** < user.
- **Product types and palettes:** bundled < **package** < user, products by slug and palettes by family; `get_style {product}`
  sees them.
- **Lint rules:** the package's `lint/*.json` count as the library's rules, with the source `library:<alias>`.

Unlink the package and they are gone. Installing alone adds nothing. A package's README is for people; Silverpoint does not read it.

## Not yet

A shared registry or remote install (a URL or a catalog), signing and verification of packages, and zip archives are left; see
`docs/BACKLOG.md`.
