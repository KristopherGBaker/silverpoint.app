---
title: "Command plugins"
description: "A command plugin adds a code target, an exporter, an importer or an image provider to Silverpoint without Swift and without rebuilding the app: it is a folder holding a plugin.json manifest and a command. Silverpoint runs the command with one JSON request on stdin and reads one JSON answer from std…"
summary: "A command plugin adds a code target, an exporter, an importer or an image provider to Silverpoint without Swift and without rebuilding the app: it is a folder holding a plugin.json manifest and a command. Silverpoint runs the command with one JSON request on stdin and reads one JSON answer from std…"
layout: doc
toc: true
---

A command plugin adds a code target, an exporter, an importer or an image provider to Silverpoint without Swift and without
rebuilding the app: it is a folder holding a `plugin.json` manifest and a command. Silverpoint runs the command with one JSON
request on stdin and reads one JSON answer from stdout (protocol 1, below). A code target adds a platform such as Flutter or your
own component markup to `export_code`, the Export Code panel and the hand-off; an exporter adds an item to **File ▸ Export As**;
an importer adds an item to **File ▸ Import** and is what the `import_file` tool runs; an image provider makes the images of
`generate_image` and the ✨ button from a prompt (Settings ▸ General ▸ Images lists it).

Plugins run code on your machine, so **a plugin runs only after you approve it**, and again after any change to it (see
[Trust](#trust-and-approval)).

The logic is SilverpointKit's (`SilverpointCodegen/Plugins`): `PluginManifest`, `PluginFolder`, `PluginModelJSON`,
`PluginHost` (the one place a plugin runs), `PluginPermissions` and `PluginEnvironment` (what its permissions let it have),
`CommandCodeTarget`, `PluginExport`, `PluginImport`, `PluginImage`, `PluginMenu` and `PluginCheck`; `import_file` is in `DesignTools`,
an image provider's generator is SilverpointAgent's `CommandImageGenerator` (registered by `PluginSetup.registerImageProviders`),
and the CLI is `silverpoint plugins`.

## Where plugins live

One folder per plugin in the plugins folder: `$SILVERPOINT_PLUGINS_DIR` when set (read through `ProcessEnvironment`, with the
pre-rename `VELLUM_PLUGINS_DIR` as a fallback), else `~/Library/Application Support/Silverpoint/Plugins/<id>/`. Install one with
**Settings ▸ General ▸ Plugins ▸ Install…** (it asks for the plugin's folder), with `silverpoint plugins install <folder>`, or by
copying its folder in; an installed plugin is not approved until you approve it. Hidden folders are skipped; a folder whose manifest does
not validate, or two folders that declare the same id, are listed as invalid (`silverpoint plugins list`) and never run.

Library code has no default folder (`PluginSetup.none`): only the entry points (the app, `silverpoint mcp`, `call` and `tools`)
pass the standard one, so tests always use a temporary folder.

Code target plugins are registered once, when the app starts (and when `silverpoint mcp`, `call` or `tools` starts): install
or remove one and relaunch to see it in the target lists. Image provider plugins are registered the same way, into each entry
point's own image provider registry (the app's Settings ▸ General ▸ Images, and the one `silverpoint mcp`, `call` and `tools`
build for `generate_image`), never a shared one. Exporters and importers are read again each time a menu opens and on
each `import_file` call. Every run reads the plugin from its folder again, so a plugin edited after launch is checked (and run)
as it is now.

## The manifest

`plugin.json`:

```json
{
  "protocol": 1,
  "id": "flutter",
  "name": "Flutter (skeleton)",
  "version": "0.1.0",
  "kind": "codeTarget",
  "command": ["/usr/bin/python3", "generate.py"],
  "timeout": 30,
  "fileExtensions": ["dart"],
  "permissions": ["library:read"],
  "capabilities": {"dropShadow": "full", "imageFill": "approximated", "innerShadow": "unsupported"}
}
```

| Field | Kinds | Meaning |
| --- | --- | --- |
| `protocol` | all | Must be `1`. |
| `id` | all | Lowercase letters, digits and `-`, starting with a letter or digit. A code target's and an image provider's id in Silverpoint is `plugin:<id>`, so it never shadows a built-in. |
| `name`, `version` | all | Non-empty strings: the name is what menus, target lists and the image provider picker show. |
| `kind` | all | `codeTarget`, `exporter`, `importer` or `imageProvider`. |
| `command` | all | A non-empty argv. `command[0]` is resolved against the plugin folder unless it is absolute (`./run.sh`, `/usr/bin/python3`); the other arguments are passed as they are. The command runs with the plugin folder as its working directory, with the environment its [permissions](#permissions) allow. No shell is involved. |
| `timeout` | all | Seconds (more than 0; fractions allowed). A run that takes longer is stopped. |
| `capabilities` | codeTarget (required, may be `{}`) | `CodeFeature` name → `full`, `approximated` or `unsupported`: the same feature list the built-in targets declare (below). |
| `fileExtensions` | codeTarget (required), exporter | File extensions without the dot; a code target's first is its main one. |
| `fileTypes` | importer (required) | The extensions it reads (matched without case). |
| `renders` | exporter | `true` to be sent a PNG of each top-level frame. |
| `models` | imageProvider (optional) | The model names Settings ▸ General ▸ Images offers, the first being the default; leave it out for a free-form model field. |
| `permissions` | all (optional) | The scopes the plugin needs: `library:read`, `comments:read`, `library:write`, `environment` (see [Permissions](#permissions)). Leave it out and the plugin is legacy: it gets everything. |

Validation is strict, and an error names the field (`capabilities.sparkles: "sparkles" is not a code feature`): a missing or
malformed `id`, a `kind` outside the four, an empty `command`, a `timeout` of 0 or less, a `protocol` other than 1, an unknown
capability (or support level), a code target without `fileExtensions`, an importer without `fileTypes`, and `models` on a kind
other than an image provider (or an empty model name) are each refused. So
is a `permissions` that is not an array of strings, or that lists an unknown scope, one twice, or one its kind cannot declare
(`permissions[1]: "comments:read" is not valid for a code target; a code target can declare library:read, environment`).

The code features (`CodeFeature`): `linearGradient`, `radialGradient`, `angularGradient`, `imageFill`, `dropShadow`, `shadowSpread`,
`innerShadow`, `layerBlur`, `backgroundBlur`, `strokeWidthProfile`, `nonSolidStroke`, `textRuns`, `textPath`, `strokeDash`,
`strokeAlign`, `contentStroke`, `multipleFills`, `multipleStrokeFills`, `nonSolidShapeFill`, `gradientOpacity`, `imageOpacity`,
`iconEffects`, `vectorEffects`, `unmappedIcon`, `alphaMask`, `luminanceMask`, `vectorMask`, `layerBlendMode`, `paintBlendMode`,
`maskInLayout`, `wrapFill`, `rowAlignment`, `wrap`, `link`, `textCase`, `paragraphSpacing`, `paragraphIndent`, `textList`,
`leadingTrim`, `fontVariations`.

## Permissions

A plugin says what it needs in `permissions`, and Silverpoint sends it only that. The scopes:

| Scope | Kinds | Effect |
| --- | --- | --- |
| `library:read` | code target, exporter | A code target is sent the model's `tokens` (variables, themes, text styles); an exporter the document's `variables`, `themes`, `textStyles`, `layerStyles`, `imports` and `packages`. |
| `comments:read` | exporter | The exporter is sent the document's `comments`. |
| `library:write` | importer | The importer's fragment may add `variables`, `textStyles`, `layerStyles` and `themes`. Without it a fragment that carries any of them is refused: the run fails naming the plugin and `library:write`, and nothing is imported. |
| `environment` | all kinds | The command inherits Silverpoint's whole environment, API keys included. Without it, it gets only the allowlist below. |

- **No `permissions` key: legacy.** The plugin gets everything, as before scopes existed: the whole request, Silverpoint's full
  environment, and an importer's fragment merged with its library. Every surface says so:
  "No permissions declared: gets everything, including your environment".
- **`[]`** declares none of the four ("No extra permissions"): a code target still gets its screens and components, an exporter
  its nodes, an importer may still add nodes.
- **What an exporter with a declared list gets.** Always: `version`, `children` (the selected nodes, when nodes are selected),
  `kind`, `startScreen`, `guide` and `features`, and `renders` as the `renders` flag says. With `library:read`: `variables`,
  `themes`, `textStyles`, `layerStyles`, `imports`, `packages`. With `comments:read`: `comments`. Never, whatever it declares:
  `hooks`, `skills`, `lintRules`, `disabledLintRules` and `origins` (Silverpoint's own configuration, not design content), nor
  any top-level key a later Silverpoint adds.
- **A code target without `library:read`** gets `tokens` as `{"variables": [], "themes": {}, "textStyles": [], "unresolvedColors": []}`,
  so the model keeps its protocol-1 shape. Its nodes are unchanged: a color or value that came from a variable still names it
  (`{"token": "brand"}`), so the code can refer to a token it is not sent.
- **An importer's request** (`path`, `bytes`) is the same with or without scopes; `library:write` decides what its answer may add.
- **An image provider's request** (`prompt`, `aspect`, `model`) holds nothing of the document; the only scope it can declare is
  `environment`, for a plugin that calls a service with your API key (`"permissions": ["environment"]`).
- **The environment allowlist** (without `environment`): `PATH`, `HOME`, `USER`, `LOGNAME`, `SHELL`, `TMPDIR`, `LANG`, `TERM`
  and every `LC_*`, each copied with its value only when Silverpoint has it; nothing is added. So `ANTHROPIC_API_KEY`,
  `OPENAI_API_KEY`, `SILVERPOINT_*` and anything else Silverpoint inherited never reach the plugin.
- **Enforced in one place.** `PluginHost` filters the request (`PluginPermissions`) and the environment (`PluginEnvironment`) for
  every run, from the app, the agent, MCP, `call` and `plugins check`, using the manifest it reads again from the folder at run
  time. `plugin.json` is part of the approval hash, so **editing `permissions` makes the plugin `changed` and it asks again**.
- The approval alert, **Settings ▸ General ▸ Plugins ▸ Review…**, `silverpoint plugins list` and `check` show the scopes in
  words, one line each: "Read your variables, styles and themes", "Read comments", "Add variables, styles and themes",
  "Use your environment (including API keys)".
- Scopes change what is sent, not the protocol: it stays version 1, and a legacy plugin's request is byte for byte what it was.

## The protocol (version 1)

Every run is one process: Silverpoint writes one JSON object to stdin and closes it, and reads one JSON object from stdout until
the process exits. stderr is kept for error messages (its last lines). JSON is written with sorted keys and unescaped slashes.

### Code target

stdin:

```json
{"protocol": 1, "kind": "codeTarget", "model": <the UIModel JSON>}
```

stdout:

```json
{
  "files": [{"path": "lib/home.dart", "contents": "…"}],
  "diagnostics": [{"feature": "innerShadow", "nodeID": "card", "message": "left out", "severity": "warning"}]
}
```

- `files` is required (it may be empty). Each `path` is relative to wherever the caller writes (the `export_code` path's folder,
  the Export Code panel's folder, the hand-off's `code/`). A path that is empty, absolute (or starts with `~`), has a `..`
  component, or is given twice fails the whole run; nothing is written.
- `diagnostics` is optional. `feature` is a `CodeFeature` name, `nodeID` the design node (`UINode.sourceID`, the `id` in the
  model), `message` what the target did instead; `severity` is `info`, `warning` (the default) or `error`. They come back from
  `export_code` as `{nodeId, feature, action}`, with the message as the action.
- **Silverpoint adds its own diagnostics**: every node that uses a feature the manifest does not declare `full` or
  `approximated` gets one (`not declared by the … plugin` or `… does not support it`), unless the plugin reported that node and
  feature itself. So a plugin that declares nothing still has honest diagnostics.
- With `annotate: true` in the model (`export_code`'s `annotate`), a target should put the node ids in the code
  (`// silverpoint:id=<id>`); reading code back through a plugin is not supported.

### Exporter

stdin:

```json
{"protocol": 1, "kind": "exporter", "document": <document JSON>, "renders": [{"frameID": "home", "png": "<base64>"}]}
```

`document` is the document in the Silverpoint JSON schema (the `.silverpoint` package's `document.json`), or, when nodes are
selected, the same document with only those nodes as its `children`; with declared [permissions](#permissions) only the keys
they allow are kept. `renders` is there only when the manifest sets
`"renders": true`: one 2× PNG per top-level frame (of the selection, when there is one), in reading order.

stdout: `{"files": [{"path", "contents"}]}`, checked as above and written under the folder the user chose (File ▸ Export As asks
for one). Contents are text.

### Importer

stdin:

```json
{"protocol": 1, "kind": "importer", "path": "/abs/path/to/input.json", "bytes": "<the file, base64>"}
```

stdout:

```json
{"fragment": {"children": [{"type": "frame", "name": "Card", "width": 320, "height": 200, "children": []}]}, "report": {"frames": 1}}
```

- `fragment` is a document in the Silverpoint JSON schema (the `schema` skill): its `children` are new top-level nodes, and it may
  carry `variables`, `textStyles`, `layerStyles` and `themes` when the manifest declares `library:write` (or no `permissions`
  at all; with a list lacking it, such a fragment is refused). It is decoded with the same decoder documents use
  (`DesignDocument.decode`); a fragment it rejects fails the run, naming the plugin and what was wrong, and changes nothing. An
  empty fragment (no nodes, variables or text styles) is refused too.
- The fragment is added as **one undo step**: its top-level nodes go after the document's (a node id already in use gets a fresh
  one, and `ref`s to it follow), and its variables, styles and theme values are added where the document does not have them yet
  (existing ones are kept).
- `report` (optional, an object) is passed through to `import_file`'s result as it is.

### Image provider

stdin:

```json
{"protocol": 1, "kind": "imageProvider", "prompt": "a harbor at dawn, watercolor", "aspect": "landscape", "model": "indigo"}
```

stdout:

```json
{"image": "<the PNG or JPEG, base64>"}
```

- `aspect` is `square`, `landscape` or `portrait`: the plugin picks the size (the node's image fill is set to `cover`, so any
  size fits). `model` is the one chosen in Settings ▸ General ▸ Images (or `--image-model`), else the manifest's first `models`
  entry, else empty.
- `image` must decode as base64 to PNG or JPEG bytes (checked by signature); anything else fails the run naming the plugin
  (`plugin "swatch-image" returned an image that is not PNG or JPEG (5 bytes)`) and nothing is stored.
- The provider's id is `plugin:<id>`: choose it in **Settings ▸ General ▸ Images ▸ Provider** (it shows the manifest's
  `name`, its `models` as the model picker, or a free-form model field, and no key field), pass it as `generate_image`'s
  `provider`, or on the command line as `--image-provider plugin:<id>` (or `SILVERPOINT_IMAGE_PROVIDER`). Its images are not
  priced.
- Every image is one run through `PluginHost`, as every plugin run is: approval by content hash (the app asks the first time,
  MCP and the CLI never ask and fail naming the plugin), the request and environment cut to its permissions, its `timeout`,
  and the agent's Stop, the ✨ button's Cancel or a cancelled MCP call ending the process.

### The UIModel JSON

`model` is the neutral `UIModel` (`PluginModelJSON`): what every code target, built-in or not, generates from. It is stable within
protocol 1: keys are never renamed or removed, optional values are left out rather than written as `null`, numbers are rounded
to 6 decimal places, and the bytes are pinned by golden files (`Tests/SilverpointCodegenTests/Fixtures/Plugin/codeexport.uimodel.json`,
and beside it `duo.uimodel.json` for device frames; `PluginUIModelJSONTests`). Top-level keys:

| Key | Value |
| --- | --- |
| `protocol` | `1`. |
| `title` | The first requested node's name, or `Silverpoint export`. |
| `annotate` | Whether the code should carry `silverpoint:id` comments. |
| `screens` | The requested nodes (every top-level node when none were named), as nodes. |
| `components` | The components they use, dependencies first: `{id, name, variant, parameters: [{name, kind (text, boolean, slot), defaultValue, component?}], root: <node>}`. |
| `tokens` | `{variables: [{name, type, values: [{value, theme, alias?, expression?}]}], themes: {axis: [values, default first]}, textStyles: [{name, fontFamily?, fontSize?, fontWeight?, italic?, letterSpacing?, lineHeight?, fill?}], unresolvedColors: [names]}`. A value that is an alias of another variable (`"$blue500"`) carries the target's name in `alias` and, in `value`, what its chain resolves to under that value's theme (null when the chain is broken). A number value that is an expression of other variables (`"=$space * 2"`) carries its text as stored in `expression` and, in `value`, the number it computes to under that value's theme (null when it cannot be computed). |
| `assets` | `{fonts: [families], images: [the image URLs fills name], icons: [SF Symbol names]}`. |

A node:

| Key | Value |
| --- | --- |
| `id` | The design node's render id (`instanceID/childID` inside an instance). |
| `kind` | `frame`, `group`, `rectangle`, `ellipse`, `polygon`, `line`, `path`, `boolean`, `text`, `icon` or `instance` (one whose component is missing). |
| `name`, `codeName` | The layers-list name, and the resolved node name. |
| `visible`, `isComponent`, `absolute`, `clip` | Booleans. |
| `frame` | `{x, y, width, height}` laid out, in the parent's coordinates. |
| `opacity`, `rotation`, `innerRadius`, `subtreeSize` | Numbers. |
| `layout` | `{direction (free, horizontal, vertical, grid), gap, padding: {top, right, bottom, left}, justify, align, wrap, rowGap?, alignContent, gridColumns?, gridRows?, gridAreas?}`; a grid's tracks are `"1fr"`, `"auto"`, `"25%"`, a fixed value, or a function as the design writes it (`"minmax(120, 1fr)"`, `"repeat(2, 100, $colW)"`, `"repeat(auto-fill, minmax(160, 1fr))"`); `gridAreas` is the grid's named areas, CSS `grid-template-areas` rows such as `["head head", "nav main"]` (`.` an empty cell). |
| `gridColumn`, `gridRow` | On a child of a grid: `{start?, span?}` as the design stores it; absent is auto-placed. |
| `gridArea` | On a child of a grid: the name of the area of the grid's `gridAreas` it covers (an explicit `gridColumn`/`gridRow` wins on its axis); absent when it names none. |
| `width`, `height` | What the node asked for: `{"fixed": <value>}`, `"fit"` or `"fill"`; left out when it asked for nothing. |
| `minWidth`, `maxWidth`, `minHeight`, `maxHeight` | Values, when set. |
| `fills` | Paints: `{type: "solid", color}`, `{type: "gradient", gradient (linear, radial, angular), stops: [{color, position}], rotation, opacity}` or `{type: "image", url, fit, opacity}`, each with an optional `blendMode`. |
| `stroke` | `{fills, thickness, align (inside, center, outside), dash?, widthProfile?: [{t, width}]}`. |
| `corners` | `{topLeft, topRight, bottomRight, bottomLeft}` values. |
| `effects` | `{type: "shadow", inner, x, y, blur, spread, color}`, `{type: "blur", radius}`, `{type: "backgroundBlur", radius}`. |
| `mask` | `{type (alpha, luminance, vector), opacity, fill?}` on a node that masks the siblings above it (it is not drawn). |
| `blendMode` | The layer blend (CSS names), when set. |
| `text` | `{content, parameter?, fontFamily, fontSize, fontWeight, italic, underline, strikethrough, letterSpacing, lineHeight?, align, growth, style?, followsPath, hasRuns, runs: [{start, end, …, link?}], maxLines?, textCase?, paragraphSpacing?, paragraphIndent?, listItems?: [{paragraph, kind, level}], leadingTrim?, fontVariations?}`. `leadingTrim` (only when set) is `"capHeight"`: the box runs from the first line's cap height to the last line's baseline. `fontVariations` (only when set) is `{tag: number}`, variable-font axis values such as `{"wght": 550, "wdth": 87.5}`; `fontWeight` already carries a `wght` value. `listItems` (only when set) are the paragraphs shown as `bullet` or `number` list items, `level` 0–3. A run's `link` (only when set) is where its characters link, a `set_link` form; `hasRuns` is whether a run styles anything (a run that only links does not). `maxLines` is the line limit as designed (the canvas applies it only to fixed-width and fixed-size text not on a path); `textCase` is `upper`, `lower` or `title` and `content` stays as stored. |
| `icon` | `{name?}` (the SF Symbol). |
| `path` | Outline elements in the node's own coordinates: `{op: move/line, to}`, `{op: quad, control, to}`, `{op: curve, control1, control2, to}`, `{op: close}`. |
| `call` | An instance that only sets parameters: `{componentID, componentName, variant, fillsWidth, fillsHeight, arguments: [{name, value, kind, component?}]}`. |
| `interactions` | `{trigger, action, target?, url?, variant?, variable?, value?, toggle?, if?}` (the last four only when set: a `set_variable`'s variable and its value or `toggle`, and an interaction's condition). |
| `source` | A placeholder for code the importer could not read: `{file, line, text}`. |
| `visibleParameter`, `slotParameter` | The component parameter that shows or hides it, or that it shows. |
| `device`, `deviceSide` | On a device frame only (README, Device frames): `device` is the posture as stored (`iphone-duo-closed`, `iphone-duo-open`, `iphone-duo-half`, or an id this build does not know), `deviceSide` the long edge its 84-wide system strip runs down when the frame is landscape, `trailing` unless the design says `leading` (a portrait frame keeps its bar where the posture puts it either way). With them a target can do what the built-in SwiftUI and UIKit exports do: pair a closed `X` with its `X (Open)` (else `X (Half Open)`) by size class, and leave the system bar, the 34 home indicator and (half open) the 40-wide fold to the safe area. |
| `children` | Nodes. |

A value is `{"value": <number>, "token": "<variable>"}`, the token only when the value came from a variable. A color is
`{"hex": "#RRGGBB"}` (or `#RRGGBBAA`) or `{"token": "<variable>"}`. The pictures a built-in target renders on demand (an icon's
PNG, a mask's SVG) are not part of the model.

## Failures

A run fails, with an error that names the plugin (`plugin "flutter" …`, one line) and changes nothing, when:

- the plugin is not approved, or changed since it was (see below), or the person clicked Don't Allow;
- its folder no longer validates, or its id or kind changed (reinstall it);
- the command cannot be started (`command[0]` missing or not executable);
- it exits with a non-zero status (the message carries the last lines of its stderr) or is killed by a signal;
- it runs past its `timeout`: it is sent SIGTERM, then SIGKILL a second later, with everything it started (the plugin leads a
  process group of its own);
- stdout is not JSON, not an object, or not the shape above (a `files` entry without a string `path` and `contents`, an unknown
  diagnostic feature), or more than 256 MiB;
- a file path is empty, absolute or climbs out with `..`;
- an importer's fragment is not a valid Silverpoint document, or carries variables, styles or themes without `library:write`;
- an image provider's `image` is missing, not a base64 string, or not PNG or JPEG bytes.

A run whose work is cancelled (Cancel in Export Code's progress, an `export_code`, `import_file` or `generate_image` call whose
task is cancelled, the ✨ button's Cancel) is stopped the same way, its whole process group, and changes nothing.

A plugin that exits without reading stdin is fine (Silverpoint never gets SIGPIPE for it). Over the CLI a failing plugin is exit
status 1 with one line on stderr; over MCP it is a tool error; in the app it is an alert.

## Trust and approval

A plugin is a program, so it runs only once you have approved **exactly this version of it**:

- The approval is remembered in the same approvals file the [hooks](../hooks/) use (`$SILVERPOINT_HOOK_APPROVALS_FILE`, else
  `~/Library/Application Support/Silverpoint/hook-approvals.json`), one entry per plugin under the subject `plugin:<id>` beside the
  documents' hook entries (each entry is approved, replaced and removed on its own).
- What is approved is a content hash: SHA-256 over every regular file in the plugin folder (the manifest included; `.DS_Store`
  and symbolic links skipped), by relative path. Editing, adding, removing or renaming any file makes the plugin `changed`, and
  it asks again.
- **In the app**, the first run of a plugin (and the first run after it changed) shows an alert with its name, id, version, kind,
  folder, the **exact command it will run** and its [permissions](#permissions): **Allow** remembers the hash and runs it, **Don't Allow** runs nothing (it asks
  again next time). The in-app agent's runs ask the same way.
- **Over MCP and the CLI nothing asks**: an unapproved or changed plugin is an error naming the plugin and saying it is not
  approved. That holds for the app's own MCP server and its App Intents (Shortcuts, Siri) too: another program cannot get a
  command approved by putting a dialog in front of you. Approve a plugin in **Settings ▸ General ▸ Plugins** (its switch, or **Review…**, then **Approve**), in the
  alert on its first use in the app, or with `silverpoint plugins install <folder> --approve` in your own terminal (which prints
  the command).
- `silverpoint plugins check <folder>` runs the folder you name once without approval: checking it is your explicit act.
- `silverpoint plugins remove <id>` deletes the folder and forgets that plugin's approval only.

**Settings ▸ General ▸ Plugins** lists the installed plugins with their name, id, kind, version and state (**Approved**,
**Needs approval** or **Changed since approval**), and the folders that are not usable plugins with the reason. Each plugin's
switch is its approval: turning it on opens **Review…**, which shows the exact command, its permissions, the folder it runs in, the SHA-256 and
every file that hash covers, and only **Approve** there approves it (as it is on disk then: if it changed since Review… opened,
nothing is approved and it shows the plugin again). Turning the switch off revokes that plugin's approval and no other entry.
**Install…** copies a plugin folder in, never approved (a leftover approval for its id is removed); **Remove…** asks, then deletes
the folder and its approval; **Reveal Folder** shows the plugins folder in Finder, and **Reload** reads it again (the section
also reads it when the window opens or comes to the front). Installing or removing a code target or an image provider takes
effect after you relaunch Silverpoint; exporters and importers show in the menus at once. Every approval, there, in the alert and with `--approve`, goes
through `PluginHost.approve`; nothing the agent or an MCP client can reach approves a plugin.

Beyond approval, the only limits are its [permissions](#permissions): an approved plugin runs as you, and one that declares
`environment` (or no `permissions` at all) gets your whole environment, API keys included. Approve only plugins you trust.

## The command line

*Not in the beta yet: the `silverpoint` command-line tool doesn't ship in the download, so this section describes it for builds made from source.*

    silverpoint plugins list [--json]
    silverpoint plugins install <folder> [--approve]
    silverpoint plugins remove <id>
    silverpoint plugins check <folder> [--input <file>]

- `list` prints one line per plugin, `id<TAB>kind<TAB>version<TAB>status<TAB>name<TAB>permissions` with the status `approved`,
  `needs approval` or `changed` and the permissions as the scope ids comma-joined (`library:write,environment`), `none` for `[]`
  or `legacy` when the manifest declares none, then `invalid<TAB><folder><TAB><reason>` for each folder that does not validate.
  A missing plugins folder prints nothing and exits 0. `--json` gives `{plugins: [{id, name, kind, version, status, folder,
  command, permissions, permissionLines}], invalid: [{folder, reason}]}`, where `permissions` is the array of scope ids (`null`
  for a legacy plugin) and `permissionLines` the lines below.
- `install` validates the folder and copies it to `<plugins>/<id>`. It refuses (exit 1, nothing copied) an invalid manifest and
  an id that is already installed. `--approve` prints the command and records the approval.
- `remove` deletes the installed plugin and its approval; an unknown id exits 1.
- `check` validates the manifest and prints, after the `command:` line, `permissions: <ids, comma-separated | none | legacy>`
  and each scope's line indented two spaces (`  Read your variables, styles and themes`), then runs a code target on a built-in sample model (two screens, a component, tokens) or an
  exporter on the sample document (with its renders when the manifest asks), an importer on `--input <file>`, or an image
  provider on a sample prompt (square, its first model; it prints `image: png (<bytes> bytes)`), and checks the
  answer against the protocol. It prints what came back and exits 0 when it is clean; a protocol error is one line on stderr and
  exit 1. An importer without `--input` gets its manifest checked only (`manifest only: …`, exit 0). Nothing is written.
- `silverpoint mcp`, `call` and `tools` register the installed code targets first, so `tools --json` lists `plugin:<id>` in
  `export_code`'s `format`, and `call <file> export_code '{"format":"plugin:flutter"}'` runs it (once approved).
  `call <file> import_file '{"path": "/abs/file.json"}'` runs an importer. They register the installed image providers into
  their own image provider registry too: `call <file> generate_image '{"id": "hero", "prompt": "…"}' --image-provider
  plugin:<id>` (or `SILVERPOINT_IMAGE_PROVIDER=plugin:<id>`) runs one, and `generate_image`'s `provider` lists them.

## In the app

- Code targets join the built-ins in **File ▸ Export Code or Tokens…**, the hand-off's target picker and `export_code`. Export
  Code runs a target off the main actor, once per format: while the save panel is up (to count its files, which picks a file or
  a folder name) and again at Save only if the document or the selection changed meanwhile. The app stays responsive while a
  slow plugin runs; past a moment Save shows a progress sheet whose **Cancel** stops the plugin's process and writes nothing,
  and closing the panel stops a run it started. A plugin that needs approval still asks with the app's alert.
- **File ▸ Export As** has one item per exporter (its manifest `name`): it asks for a folder and writes the files there.
- **File ▸ Import** lists **From Figma…**, **From Mermaid…**, **From Excalidraw…**, **From draw.io…** (the last three built in, not plugins), then one item per importer,
  whose open panel offers the importer's `fileTypes`; the import is one undo step.
- `import_file` (`path`, optional `plugin`; otherwise the first importer, by id, whose `fileTypes` has the file's extension)
  returns `{plugin, created, report}`: `created` are the new top-level node ids.
- Image providers join the built-ins in **Settings ▸ General ▸ Images ▸ Provider** (with a note saying the provider is a
  plugin and runs once approved), so `generate_image` and the ✨ button use the one chosen there; `generate_image`'s `provider`
  takes `plugin:<id>` for one call.

## The samples

Three sample plugins are in the repository, under `Examples/plugins/` (nothing installs them; they are also the tests' fixtures).
All use only the system's `/usr/bin/python3` and its standard library.

- **`flutter-skeleton`** (id `flutter`, a code target, `.dart`): one file per screen and component under `lib/`, each importing
  `package:flutter/material.dart` and holding a `StatelessWidget` named after the root (frames become `Column`, `Row` or `Stack`,
  text `Text`, the rest sized `Container`s). It declares only a few capabilities (drop shadows full, image fills and multiple
  fills approximated, inner shadows unsupported, which it reports itself), so Silverpoint reports the rest. It declares
  `library:read`, since it looks a fill's color variable up in the model's `tokens` to write its value.

      silverpoint plugins check Examples/plugins/flutter-skeleton
      silverpoint plugins install Examples/plugins/flutter-skeleton --approve
      silverpoint call design.silverpoint export_code '{"format": "plugin:flutter"}'

- **`shapes-json`** (id `shapes-json`, an importer, `.json`): reads `{"frames": [{"name", "width", "height", "rects": [{"x", "y",
  "width", "height", "color"}]}]}` into frames holding rectangles, side by side, and reports the counts and anything it skipped.
  `sample.json` has two frames, `Sample Card` (three rectangles) and `Sample Banner` (two). It adds only nodes, so it declares
  `"permissions": []`.

      silverpoint plugins check Examples/plugins/shapes-json --input Examples/plugins/shapes-json/sample.json
      silverpoint plugins install Examples/plugins/shapes-json --approve
      silverpoint call design.silverpoint import_file "{\"path\": \"$PWD/Examples/plugins/shapes-json/sample.json\"}"

- **`swatch-image`** (id `swatch-image`, an image provider, models `indigo` and `coral`): answers a small solid-color PNG built
  with `zlib`, the same every time, with no network: 16×16 for `square`, 24×16 for `landscape`, 16×24 for `portrait`, in the
  model's color (the prompt is ignored). It needs nothing of yours, so it declares `"permissions": []`.

      silverpoint plugins check Examples/plugins/swatch-image
      silverpoint plugins install Examples/plugins/swatch-image --approve
      silverpoint call design.silverpoint generate_image '{"id": "hero", "prompt": "a swatch"}' \
        --image-provider plugin:swatch-image --image-model coral

## Not yet

Exporters return text files only (no binary `contents`); a plugin cannot read code back into the design (`parse` stays
unsupported); there is no sandbox beyond approval and permissions (no network or file limits), and no marketplace or updater.
