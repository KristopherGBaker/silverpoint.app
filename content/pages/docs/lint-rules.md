---
title: "Custom lint rules"
description: "Silverpoint's design check (lint, the inspector's Design Check, silverpoint score, the agent's self-check) runs ten built-in rules: raw-color, off-grid, text-style, tiny-text, contrast, clipped, empty-mask, brand-drift, variable-alias and variable-expression. Custom lint rules add a team's own conv…"
summary: "Silverpoint's design check (lint, the inspector's Design Check, silverpoint score, the agent's self-check) runs ten built-in rules: raw-color, off-grid, text-style, tiny-text, contrast, clipped, empty-mask, brand-drift, variable-alias and variable-expression. Custom lint rules add a team's own conv…"
layout: doc
toc: true
---

Silverpoint's design check (`lint`, the inspector's **Design Check**, `silverpoint score`, the agent's self-check) runs ten
built-in rules: `raw-color`, `off-grid`, `text-style`, `tiny-text`, `contrast`, `clipped`, `empty-mask`, `brand-drift`,
`variable-alias` and `variable-expression`. **Custom lint rules**
add a team's own conventions to the same check: "buttons use the 8 or 12 radius", "every screen is 390 wide", "icon names are
kebab-case".

`contrast` (`error`, no fix) reports what is hard to see against what is behind it, the last opaque solid fill of an enclosing
frame or rectangle: 4.5:1 for text, 3:1 for text of 24 pt or more (18.5 pt bold), and (WCAG 1.4.11, non-text contrast) 3:1 for an
icon, a shape up to 48 by 48 pt and a control's border. An icon is checked in its color, a shape in its fill (its stroke when it
has no fill; a boolean is one shape), and a border is a frame's or rectangle's stroke of 1 pt or more: a border is checked only
when the control's fill does not reach 3:1 against what is behind it, so an unfilled input or a card the color of the page is
checked and a filled button is not. Dividers are decoration, so lines are not checked, and neither are connectors and masks. A
layer with an opacity or alpha of 0.1 or less, or named `decor` or anything starting with it (`decor dot`, `Decorative`, in any
case), is decoration with everything inside it: its icons, shapes and borders are not checked, its text still is. Silverpoint
has no disabled state, so there is no disabled exception, and no focus state on the canvas: a focus ring is a stroke, so a
`focused` variant's ring is checked as a border. With nothing solid behind a node, nothing is checked: gradient or image
backgrounds are not checked. The rule checks every theme mode, one axis at a time (no cross product): besides the canvas, each
other mode of each axis that a color variable has a value for is rendered once, with the other axes as the canvas shows them,
and a problem found only there is reported once, its message ending `(theme <axis>: <mode>)`, e.g.
`contrast 1.4:1 of #334155 on #1E293B (theme mode: dark)`. The detail names the part: `contrast …` for text,
`icon contrast …`, `shape contrast …` and `border contrast 1.2:1 of #E5E7EB on #FFFFFF`.

`brand-drift` is advisory (`info`, no fix): it flags a top-level frame whose background is one of the cream neutrals of Claude's
own brand (#FAF9F5, #F0EEE6, #E8E6DC) and that holds a clay or terracotta fill (#D97757, #C6613F, #C15F3C), each within a few
values per channel, once per frame. It stays quiet in a document that links a design system (`imports` or `packages`). When the
user asked for or supplied that palette, the agent turns it off with `set_builtin_lint_rule {rule: "brand-drift", enabled: false}`.

`variable-alias` (`warning`, no fix) reports, once per variable on the document root, a variable alias (`"$other"`) that does not
resolve under the default theme or some combination of the theme axes: a cycle (`alias cycle a → b → a`), a missing target
(`alias x → later (no such variable)`) or a hop of another type. `set_variables` already refuses a cycle or a type mismatch, so
in practice it finds aliases of variables not defined (or linked) yet, and documents saved before the check.

`variable-expression` (`warning`, no fix) does the same, once per variable, for a number variable with an expression among its
values (`"=$space * 2"`; see the schema's Variables and themes): an expression that does not parse
(`expression error at column 8: …`), a cycle through aliases and expressions (`alias cycle a → b → a`), a missing target
(`alias m → later (no such variable)`), a reference to a variable that is not a number, a division by zero (`division by zero`)
or a result too large to be a number (`result is not a finite number`), under the default theme or any theme combination. A
plain alias whose chain breaks inside an expression further on stays `variable-alias`'s, so each broken variable is one issue.
`set_variables` refuses an expression that does not parse, a cycle or a type mismatch; the rest are accepted and reported here.

A rule is **declarative JSON**. Nothing a rule says is ever executed as code: Silverpoint reads it, validates it and runs its
own checks. You add, change or remove rules without rebuilding Silverpoint.

A rule picks nodes (`match`), says what must be true of each one (`expect`), and optionally how to fix a node that breaks it
(`fix`). Every issue a rule raises carries the rule's id, its severity and where the rule comes from, so you can tell your rules
from the built-in ones.

## Where rules live

Rules come from three layers, lowest first. A higher layer replaces a lower one with the same rule id.

| Layer | Where | Source shown on an issue |
| --- | --- | --- |
| **user** | `*.json` files in `~/Library/Application Support/Silverpoint/Lint` (or the folder in `$SILVERPOINT_LINT_DIR`) | `user` |
| **library** | the `lintRules` of every design system the document links | `library:<alias>` |
| **document** | the `lintRules` stored in the document itself | `document` |

So **document > library > user**. The team's rules travel with the file (document) and with a design system (library: a document
that links the library gets the library's rules), and personal rules sit in the folder and apply to every document.

- Each `*.json` file in the user folder is a rule file, `{"rules": [rule, ...]}`. Files are read in byte order of their names.
- A document keeps its rules as a `lintRules` list of rule objects, and its switched-off built-ins as `disabledLintRules`
  (a list of rule names). Both are stored only when there is something in them, so a document without custom rules is
  byte-for-byte the document it was before.
- Library code, tests and the agent's tools resolve the document's and its libraries' rules only. The app, the CLI and the MCP
  server also read the user folder.
- A design system **package** (`docs/design-system-packages.md`) can ship rule files in its `lint/*.json`, in this same format.
  While a document links the package they are in the **library** layer, with the source `library:<alias>` (the alias the
  document links the package under), after the `lintRules` of the library inside it. Unlink the package and they are gone; a
  rule file that does not validate is reported with the file `library:<alias>/<name>.json`.
- When two libraries define one id, the first linked library wins. Nothing is reported.

### Same id twice

- In one layer's document or file, a repeated id is a problem and the later rule is skipped.
- In the user folder, if two files define one id, the file that sorts first by name (byte order) wins. The later file's rule is
  skipped and reported as a problem that names the winning file: `b.json: dup-id: skipped: a.json already defines the rule
  "dup-id" (the first file in byte order wins)`.
- Across layers it is not a problem, it is the precedence above. `silverpoint lint-rules` shows `(overrides user)` or
  `(overrides library)` beside the winner.

### Turning a built-in rule off

A document can turn a built-in rule off: ask the agent ("this document uses its own spacing scale, turn off off-grid"), or use
the `set_builtin_lint_rule` tool. The rule name is stored in the document's `disabledLintRules`, so it travels with the file. A
turned-off built-in does not run, and `rules: ["off-grid"]` does not bring it back. Custom rules are not switched off this way:
replace or delete them instead.

## The rule format

```text
{
  "id": "button-radius",
  "severity": "warning",
  "message": "{name} radius {value} should be one of {expected}",
  "match":  { "type": "frame", "name": "Button*" },
  "expect": [ { "check": "in", "property": "cornerRadius", "values": [8, 12] } ],
  "fix":    { "set": { "cornerRadius": 8 } }
}
```

| Key | |
| --- | --- |
| `id` | Required. Kebab-case (`button-radius`). It must not be the name of a built-in rule. |
| `severity` | `error`, `warning` or `info`. Default `warning`. |
| `message` | A template for the issue's text, see [Messages](#messages). Optional. |
| `match` | Which nodes the rule looks at. Optional: without it every node. |
| `expect` | Required, a non-empty list of checks. A node that fails a check raises an issue. |
| `fix` | Optional. How to fix a node that fails. |
| `description` | Optional note for people; ignored. |

Any other key is a problem (a typo like `"mesage"` would otherwise do nothing, silently).

### `match`

Every condition that is present must hold (they are ANDed). All are optional.

| Key | Selects |
| --- | --- |
| `type` | A node type (`"frame"`) or a list of them (`["text", "icon"]`): `frame group rectangle ellipse polygon line path text icon ref boolean`. |
| `name` | A glob on the node's name: `*` is any run of characters, `?` one character. The whole name must match; case matters. `"Button*"`. |
| `topLevel` | `true`: a screen, that is a direct child of the document that is not a reusable component. `false`: any other node. |
| `inside` | `{"kind": "component" \| "frame" \| "screen", "name": glob}`. The node has an ancestor of that kind, optionally with a name that matches. Any ancestor counts, not only the parent. A `screen` is a top-level frame that is not a component. |
| `has` | A property name or a list: the node has all of them set (`["fill"]`). |
| `lacks` | A property name or a list: the node has none of them set. |

### `expect`

Each item has a `check`. A rule can list several; each failing check raises its own issue.

Checks that read node properties take `property`, a node key as it is in the document JSON (`width`, `height`, `cornerRadius`,
`gap`, `padding`, `fontSize`, `opacity`, `layout`, `justifyContent`, ...; `strokeWidth` is the stroke's thickness), or a **list**
of them, each checked on its own.

| `check` | Fields | Passes when |
| --- | --- | --- |
| `equals` | `property`, `value` | the property equals `value`. A property that is not set does not. |
| `in` | `property`, `values` | the property equals one of `values`. Not set: fails. |
| `multipleOf` | `property`, `value` | the number is a multiple of `value` (`value` > 0). |
| `range` | `property`, `min` and/or `max` | the number is within the range, ends included. |
| `colorVariable` | `property` (`fill` or `stroke`, default `fill`), `prefix` or `group` (optional) | every paint of the property is a color variable (`$brand/primary`), and with `prefix` (`"brand/"`) its name starts with it; `group: "brand"` is `prefix: "brand/"`. A raw color such as `#FF0000` fails, and so does a variable outside the prefix. Gradient stops are checked too. No fill or stroke: passes. |
| `textStyle` | `oneOf` (optional) | a text node uses a text style, and with `oneOf` one of those. Nodes that are not text pass. |
| `name` | `regex` | the node's name matches the regular expression (found anywhere in the name; anchor with `^` and `$`). An unnamed node has the empty name. |
| `children` | `min` and/or `max` | the number of children is within the range. |

How values are read:

- **Numbers.** `multipleOf` and `range` check numbers. A property that is not set, is `fit_content` or `fill_container`, or is a
  variable that is not a number variable, has nothing to measure and is left alone. A number variable (`"$space-md"`) is
  read as its value in the default theme.
- **`padding` and `cornerRadius` have four sides.** For `multipleOf`, `range` and the `in` set, **a value passes only when all
  four sides satisfy it**: `padding: [8, 8, 8, 12]` is not a multiple of 8 because of the last side. `equals` compares the four
  sides too, and `16` means `[16, 16, 16, 16]`. A message shows the sides when they differ: `[8, 8, 8, 12]`.
- **`fontSize`** is the node's own size, or when it has none the size of its linked text style.
- **`equals` and `in`** also compare text values (`"layout": "vertical"`).

### `fix`

A fix is expressed as the same batch operations the built-in fixes use (`update`), so `fix_lint` and the Design Check's **Fix
All** apply it as one undo step.

| `fix` | Does |
| --- | --- |
| `{"set": {"cornerRadius": 8}}` | sets those properties on the node that failed. Any node property but `id`, `type` and `children`. |
| `{"link": "variable"}` | needs a `colorVariable` check. Sets the fill (or stroke) to the color variable nearest in color, among the variables inside the check's `prefix`. A variable outside the prefix is replaced by the nearest one inside. Only a single, plain solid color is fixed, and nothing if no variable fits. A design system's variables count. |
| `{"link": "textStyle"}` | needs a `textStyle` check. Links the text style nearest in font size (the smaller on a tie), among `oneOf` when given, and clears the node's own size. |

An issue whose fix cannot be worked out is raised without a fix; `fix_lint` leaves it for you.

### Messages

`message` is a template. These placeholders are filled in:

| Placeholder | Is |
| --- | --- |
| `{id}` | the rule's id |
| `{name}` | the node's name |
| `{type}` | the node's type |
| `{property}` | the property that failed |
| `{value}` | its value (`none` when it is not set) |
| `{expected}` | what the check wanted: `8, 12` for `in`, `390` for `equals`, `12 to 24` / `at least 12` / `at most 24` for ranges |

Anything else in braces is left as written. Without a `message` the text is `{name}: {property} is {value}, expected {expected}`.

A custom issue's line reads `[severity] <rule id> (<source>) <node id>: <message>`; a built-in issue's does not name a source.

## Invalid rules

A rule that does not validate is skipped and **reported**; the other rules in the same file, and in every other file, still run.
Silverpoint never fails or crashes over a rule file. A problem names the file (`team.json`, `document`, or
`library:<alias>`), the rule id (`-` when there is none, for example not valid JSON) and the reason:

```text
broken.json: bad-sev: unknown severity "loud" (error, warning, info)
team.json: -: not valid JSON: ...
```

The reasons: not valid JSON, a missing or repeated id, an id that is not kebab-case or is a built-in rule's name, an unknown
severity, check kind, match key, rule key or property, a bad regular expression, an empty `expect`, a `fix` of the wrong shape (or
a `link` fix without its check), and a rule that is not an object.

Where problems show: the Design Check section lists them under the issues, `lint` appends them to its output,
`silverpoint score` prints them to stderr, and `silverpoint lint-rules` prints them and exits 1.

## From the command line

*Not in the beta yet: the `silverpoint` command-line tool doesn't ship in the download, so this section describes it for builds made from source.*

```text
silverpoint lint-rules [<file.silverpoint>] [--json] [--check <file.json>]
```

- Without options it lists every rule in force, built-in rules included, one line each: `id<TAB>source[ (overrides X)][ (disabled)]<TAB>severity`.
  With a document it uses that document's rules and its libraries'; without one, the user folder's.
- Problems go to stderr, as `<file>: <id|->: <reason>`. The exit status is 0 when there are none and 1 when there are any (the
  valid rules are still listed).
- `--check <file.json>` validates one rule file and prints `ok: N rules`. It exits 1 with the problems on stderr when the file
  has any, or cannot be read.
- `--json` prints the rules and the problems as one JSON object.
- Any other option is a usage error (exit 2).

`silverpoint score` and `silverpoint call <file> lint` run the custom rules too; `score` counts them under their own rule ids in
`lint.counts`, and each issue has a `source`.

## From the agent and MCP

- `lint` and `fix_lint` run built-in and custom rules together. `rules` limits them to some ids, built-in or custom.
- `save_lint_rule {scope, rule, confirm}` drafts a rule from a convention you state ("all cards have 16 padding"). It first
  **previews**: the rule, whether it is valid, which lower layer it would override, whether a higher one would shadow it, and the
  issues it would raise on the current document. Nothing is written until you confirm. In the app the agent shows the rule on a
  card with **Save** and **Discard**; over MCP (and `silverpoint call`) it writes only with `confirm: true`. `scope: "document"` stores it
  in the document as one undo step; `scope: "user"` writes `<lint folder>/<id>.json`. If a file in the user folder already
  defines the id, that file is rewritten in place (the rule replaced, its other rules kept) and the preview names it. An invalid
  rule is an error and nothing is written.
- `set_builtin_lint_rule {rule, enabled}` turns a built-in rule off or on for the document, as one undo step.

## Sample rules

The six below cover the conventions teams ask for first. Each is a rule file you can drop into the lint folder as it is.

**Buttons use the 8 or 12 radius.** Name-pattern match, a set, and a fix that picks 8.

```json
{"rules": [
  {
    "id": "button-radius",
    "severity": "warning",
    "message": "{name} radius {value} should be one of {expected}",
    "match": {"type": "frame", "name": "Button*"},
    "expect": [{"check": "in", "property": "cornerRadius", "values": [8, 12]}],
    "fix": {"set": {"cornerRadius": 8}}
  }
]}
```

**Every screen frame is 390 wide.** A screen is a top-level frame (components excluded).

```json
{"rules": [
  {
    "id": "screen-width",
    "severity": "error",
    "message": "{name} is {value} wide; screens are {expected}",
    "match": {"type": "frame", "topLevel": true},
    "expect": [{"check": "equals", "property": "width", "value": 390}],
    "fix": {"set": {"width": 390}}
  }
]}
```

**No text under 12pt in screens.** `inside` scopes the rule to screens; the size comes from the text style when the node has none.

```json
{"rules": [
  {
    "id": "screen-min-text",
    "severity": "warning",
    "message": "{name} is {value}pt; text in screens is at least 12pt",
    "match": {"type": "text", "inside": {"kind": "screen"}},
    "expect": [{"check": "range", "property": "fontSize", "min": 12}],
    "fix": {"set": {"fontSize": 12}}
  }
]}
```

**Colors come from brand/\* variables.** Fills and strokes; the fix links the nearest `brand/` variable.

```json
{"rules": [
  {
    "id": "brand-colors",
    "severity": "warning",
    "message": "{name}: {property} is {value}, use {expected}",
    "expect": [
      {"check": "colorVariable", "property": "fill", "prefix": "brand/"},
      {"check": "colorVariable", "property": "stroke", "prefix": "brand/"}
    ],
    "fix": {"link": "variable"}
  }
]}
```

**Icon names are kebab-case.** A regular expression on the name.

```json
{"rules": [
  {
    "id": "icon-kebab-case",
    "severity": "info",
    "message": "icon \"{name}\" should be named in kebab-case, like arrow-left",
    "match": {"type": "icon"},
    "expect": [{"check": "name", "regex": "^[a-z0-9]+(-[a-z0-9]+)*$"}]
  }
]}
```

**Spacing is a multiple of 8.** `gap` and `padding` of frames; the padding passes only if all four sides do.

```json
{"rules": [
  {
    "id": "spacing-8",
    "severity": "info",
    "message": "{name}: {property} {value} is not a multiple of {expected}",
    "match": {"type": "frame"},
    "expect": [{"check": "multipleOf", "property": ["gap", "padding"], "value": 8}]
  }
]}
```

## In Settings

**Settings ▸ Skills ▸ Lint rules** lists the rules in force for the frontmost document, as `silverpoint lint-rules` does: the
built-in ones, your folder's, linked libraries' and the document's own, by id, each with its severity, where it comes from
(Built-in, Your folder, Library and its alias, This document) and what it overrides. A file that does not validate is a row of its
own, with the file, the rule id and the reason. With no design open, only the built-in rules and your folder's are listed.

Each built-in rule has a switch: turning it off turns the built-in rule off for the frontmost document only, as one undo step
(the same change `set_builtin_lint_rule` makes), and the document keeps the choice. A document made with a newer Silverpoint
is read-only, so its switches are off. Custom rules have no switch: replace or delete them instead.

**Add Rule…** writes a starter file into your folder (`new-rule.json`, then `new-rule-2.json`…, never over a file that is
there): the `icon-kebab-case` sample above at severity `info`, with the file's name as its id, and shows it in Finder to edit.
**Reveal Folder** shows your folder in Finder, made if it is missing, and **Reload** reads the rules again.

## Not covered yet

- Rules are JSON only; there is no YAML.
- The `noLint` expectation of `silverpoint score --expect` still names the built-in rules only; a custom id is not accepted there
  yet. (Custom issues are counted in `lint.counts` and can be read from the score.)
- Settings lists the rules and switches the built-in ones, but a rule's JSON is edited in its file (or by the agent).
- A rule judges one node at a time: no checks across siblings or between screens.
