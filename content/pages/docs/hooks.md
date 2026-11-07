---
title: "Hooks: run a command after a save or an export"
description: "A hook runs a command on your Mac when Silverpoint has finished saving a document or exporting from it: copy tokens.json into a repo, format the exported Swift, stage the exported files. Hooks run after the event is done, off the main actor, and never block or fail the save or the export that trigg…"
summary: "A hook runs a command on your Mac when Silverpoint has finished saving a document or exporting from it: copy tokens.json into a repo, format the exported Swift, stage the exported files. Hooks run after the event is done, off the main actor, and never block or fail the save or the export that trigg…"
layout: doc
toc: true
---

A hook runs a command on your Mac when Silverpoint has finished saving a document or exporting from it: copy `tokens.json` into a repo,
format the exported Swift, stage the exported files. Hooks run **after** the event is done, off the main actor, and never block or fail
the save or the export that triggered them. What a hook prints, and how it ended, goes to a log you can open.

A hook is a command **you** (or your team) wrote. A document's hooks come from a file that may have come from someone else, so they
never run until you allow them: see [Trust](#trust).

## Where hooks live

| Source | Where | Trust |
| --- | --- | --- |
| **User** | `~/Library/Application Support/Silverpoint/hooks.json`, or the file `$SILVERPOINT_HOOKS_FILE` names | trusted: you wrote it |
| **Document** | the `hooks` key of the `.silverpoint` document, so a team's hooks travel with the file | never runs until you allow it for that document path |

The other two locations follow the same rule and can be moved with the environment (a pre-rename `VELLUM_` name works as a fallback):

| What | Default | Override |
| --- | --- | --- |
| User hooks file | `~/Library/Application Support/Silverpoint/hooks.json` | `SILVERPOINT_HOOKS_FILE` |
| Approvals | `~/Library/Application Support/Silverpoint/hook-approvals.json` | `SILVERPOINT_HOOK_APPROVALS_FILE` |
| Logs | `~/Library/Application Support/Silverpoint/HookLogs/` | `SILVERPOINT_HOOK_LOGS_DIR` |

The user file is `{"hooks": [hook, ...]}`. A document stores the list itself: `"hooks": [hook, ...]` next to `"children"`. A document
without hooks has no `hooks` key at all, so it saves byte for byte as before.

When one event matches hooks from both sources, **yours run first, then the document's**, one after the other, in the order written.
A disabled hook (`"enabled": false`) is skipped.

## A hook

```json
{
  "id": "sync-tokens",
  "event": "afterExport",
  "filter": ["tokens"],
  "command": ["/bin/cp", "tokens.json", "../my-app/Sources/Design/tokens.json"],
  "cwd": ".",
  "env": { "MODE": "release" },
  "timeout": 30,
  "enabled": true
}
```

| Field | Meaning |
| --- | --- |
| `id` | Required, unique in its file. Names the hook in the log, the notice, `list_hooks` and the environment. |
| `event` | Required: `afterSave`, `afterExport` or `afterExportCode`. |
| `filter` | Optional list of strings. `afterExport`: the kinds it runs for (`image`, `pdf`, `code`, `tokens`, `designSystem`, `handoff`, `site`). `afterExportCode`: the code target ids it runs for (`swiftui`, `html`, `compose`, `react`, `uikit`, ...). Absent means all. Not allowed on `afterSave`. |
| `command` | Required: the **argv**, a non-empty array of strings. It is **not** run by a shell: `["/bin/echo", "$HOME"]` prints the literal `$HOME`. The first item is a path; a bare name such as `git` is looked up in `PATH` (an app started from the Dock has a short one, so prefer a full path or `/usr/bin/xcrun`). |
| `shell` | `true` runs `/bin/sh -c` over the command: `command[0]` is the script and the rest are its `$0`, `$1`... Default `false`. |
| `cwd` | The working directory. Default: the folder the document is in. A relative path is relative to that folder; an absolute path (or `~/...`) is used as is. A document that is not saved yet runs your hooks from your home folder. |
| `env` | Extra environment variables (strings). They are added to the environment Silverpoint was started with. |
| `timeout` | Seconds, above 0. Default 60. |
| `enabled` | Default `true`. |

### Validation

Every hook is checked on its own. A hook that does not validate is **reported and skipped; the others still run**. The reasons:
a missing or repeated `id`, an unknown `event`, an empty or non-string `command`, a `timeout` that is not above 0, a `filter` kind
that does not exist (or any filter on `afterSave`), and a wrong type for `shell`, `cwd`, `env` or `enabled`. Each problem names the
source, the hook's id or its position, and the reason: `user hook #2 "typo": unknown event "afterSav" (use afterSave, ...)`. A user
file that is not valid JSON (or not `{"hooks": [...]}`) is one problem and no hooks, and the document's hooks are unaffected. The
problems show in Settings ▸ General ▸ Hooks and in the `errors` of `list_hooks`.

## Events

| Event | Fires | `files` in the payload |
| --- | --- | --- |
| `afterSave` | after the file is written, for a save **you** asked for: Save, Save As, and a Duplicate's first save. **Never for autosave** (the app autosaves in place continuously, so a hook per autosave would run all the time). `silverpoint call` and `mcp` fire it after each saved edit when given `--run-hooks`. | the saved package: `[documentPath]` |
| `afterExport` | after files were written, with the kind in `filter`: `image` (File ▸ Export… as PNG, JPEG or SVG; Export Frames as PNGs; the `export` tool; the inspector's **Export <name>…** and `export_presets` for a layer's PNG, JPG and SVG presets, as one export), `pdf` (Export… as PDF; Export Deck as PDF; **Export <name>…** and `export_presets` for its PDF presets), `code` (Export Code), `tokens` (Export Code or Tokens with a tokens format; `export_tokens`), `designSystem` (Export Design System Package; `export_design_system`), `handoff` (Export Developer Handoff; `export_handoff` **and** `hand_off`), `site` (a website theme and its content; File ▸ Publish Website…'s Publish, once, before its build; `export_site` with `mode: apply`; `silverpoint publish` runs no hooks) | the files written |
| `afterExportCode` | an `afterExport` of kind `code`, as well: it carries the code target and filters by it | the files written |

Details worth knowing:

- An export fires only when it wrote files. `export_tokens` and `export_code` without a `path` return text and write nothing, so they fire
  nothing; the `render` and `score` commands never fire a hook.
- A code export of several files (Compose, React, UIKit) lists every file of its folder in `files`.
- `hand_off`'s `files` are the bundle's `written` list made absolute (the tool's own list is relative to the bundle), and its
  `tokensPath` is the bundle's `tokens.json`. `export_handoff`'s `tokensPath` is its `tokens.json`. Both are `handoff` exports.
- Each event is announced by the shared tool catalog (`DesignTools`), so the in-app agent and the CLI fire the same events as the
  menu commands do.

## What the command gets

On **stdin**, one JSON object (slashes are not escaped, keys are sorted):

```json
{
  "event": "afterExportCode",
  "kind": "code",
  "documentPath": "/Users/me/Design/Pulse.silverpoint",
  "files": ["/Users/me/app/Design.swift"],
  "codeTarget": "swiftui",
  "tokensPath": null,
  "hookId": "format-swift"
}
```

`kind` is the export kind (`null` for a save), `codeTarget` is set for a code export, `tokensPath` for tokens, design system and
handoff exports, and `documentPath` is `null` for a document that is not saved yet. A field that does not apply is `null`.

The same values are in the environment, always set, empty when they do not apply:

| Variable | Value |
| --- | --- |
| `SILVERPOINT_HOOK_EVENT` | `afterSave`, `afterExport` or `afterExportCode` |
| `SILVERPOINT_HOOK_KIND` | the export kind |
| `SILVERPOINT_HOOK_DOCUMENT` | the document's path |
| `SILVERPOINT_HOOK_FILES` | the files, one per line |
| `SILVERPOINT_HOOK_CODE_TARGET` | the code target id |
| `SILVERPOINT_HOOK_TOKENS_PATH` | the tokens file |
| `SILVERPOINT_HOOK_ID` | the hook's id |

These are added to the environment Silverpoint was started with and your hook's `env`; the `SILVERPOINT_HOOK_*` values win.

## Running, timeouts, coalescing and the log

- **Off the main actor, after the event.** The save or export has finished and its result is available before the hook starts. A hook that
  fails to launch, exits non-zero or times out is recorded; it never throws into the save or export.
- **Timeout.** A hook runs as the leader of a process group of its own. One that runs longer than its `timeout` has its whole group sent
  SIGTERM, then SIGKILL if any of it is left two seconds later, and is recorded as `timed out`. So a program it started (a `shell: true`
  script's background job, a tool's helper) ends with it, unless that program left the group itself (`setsid`, a daemon). A hook that
  exits in time is not followed up: a background job it leaves behind keeps running.
- **Quitting.** Quitting Silverpoint ends every hook still running the same way (its whole group, SIGTERM, then SIGKILL after two
  seconds) before the app is gone, starts no more and shows no notice for them. Closing a document does not stop its hooks.
- **Coalescing.** A hook (per document and hook id) that is still running is not started again by the next save or export. The trigger
  waits, a second one replaces it, and **one** more run starts when the first finishes, with the latest payload. A burst of ten
  saves while a hook runs costs one extra run, not ten.
- **The log.** One text file per document in the logs folder, named after the document and a digest of its full path (so
  `a/Pulse.silverpoint` and `b/Pulse.silverpoint` have two logs). Each run appends an entry: the time, the hook id, its source and
  event, the argv, the exit status (`exit 0`, `exit 1`, `signal 9`, `timed out`, `failed to launch: ...`), and its stdout and stderr
  (the first 256 KB of each). Settings ▸ General ▸ Hooks ▸ **View Log** opens the frontmost document's log.
- **A notice.** A hook that fails, fails to launch or times out shows a small notice in the corner of the screen with **Show Log**.
  It takes no focus and goes away by itself. A hook that exits 0 shows nothing.

## Trust

A hook is a command that runs on your Mac with your permissions. So:

- **Your hooks file is trusted.** You wrote it.
- **A document's hooks never run until you allow them for that document.** When you open a document that has hooks, or when its
  hooks change (an edit in Settings, an undo, the file changing on disk), Silverpoint shows each hook's exact command (and shell
  flag, folder and environment) with **Allow** and **Don't Allow**. Allow records the hooks; Don't Allow records nothing and they
  stay off (and you are not asked again about the same hooks until they change).
- **The approval is one hash over the document's whole hooks list, per document path.** It is SHA-256 over the canonical
  (sorted-keys) JSON of the list in order, so it covers everything that decides what runs: each hook's id, event, filter, command,
  `shell`, `cwd`, `env` and `timeout`, and which hooks there are and in what order. It does **not** cover `enabled`, so turning a
  hook off and on keeps the approval. Editing any one hook, or adding, removing or reordering one, makes **all** of that document's
  hooks unapproved until you allow them again (the unchanged ones stop too). The same hooks in a copy at another path, or after a
  move or rename, are not approved (a new path needs a new approval; nothing is migrated). The approvals file holds one entry per
  document path: `{"approvals": {"/path/to/Doc.silverpoint": "<sha256>"}}`.
- **A change you make in Settings' hook form is approved as you save it**, when the document's list was approved before it (see
  [Settings](#settings)); any other change, an undo or a redo asks again.
- **Review** the state any time in Settings ▸ General ▸ Hooks: the document's hooks share one state, Approved or Not approved, and
  Review… shows the sheet again.
- **`silverpoint call` and `silverpoint mcp` run no hook unless you pass `--run-hooks`**, and then only your hooks and the document
  hooks you approved in the app. Without the flag nothing is read from or written to the approvals or the log.
- **The app's own MCP server never runs hooks**, whatever a client asks: another program is driving those tools. The in-app agent is
  you directing the app, so its exports run hooks like the menu commands do.
- **Shortcuts and Siri never run hooks either**: an App Intent (Export Frame, Save Version and the others) is the `intent` origin,
  another program driving the call, so an export it makes runs none, as over MCP.
- **No tool approves, writes or toggles a hook** (only you, in Settings). `list_hooks` is read-only: it returns each hook's id, event, filter, command,
  `shell`, `cwd`, `timeout`, `enabled`, `source` (`user` or `document`) and `approval` (`trusted`, `approved` or `notApproved`),
  and the hooks that did not validate. An agent can suggest a hook for you to add; it cannot add one.

## Settings

Settings ▸ General ▸ Hooks lists the hooks in force for the frontmost document with a **User** or **Document** badge, and for a
document hook whether the document's hooks are Approved (they share one state). **Add Hook…** opens the hook form. Each row has
**Edit** (the same form, filled in), **Run Now**, **Duplicate**, **Delete** and a switch. **Reveal in Finder** shows your hooks
file, **View Log** the document's log, and **Review…** the approval sheet. Hooks that did not validate are listed under the rows.
The group is in General because that card had the room; the document hooks use the same frontmost document as the Skills card.

### Add Hook… and Edit

The form has every field of a hook:

- **Save in:** your hooks or this document's (offered only for a saved document in front; a new document hook needs the path its
  approval is kept for). Editing keeps a hook where it is.
- **When:** after a save, after an export (tick the kinds; none ticked runs after every kind) or after a code export (code target
  ids, comma-separated; empty runs after every target). Switching to after a save clears the filter.
- **Command:** one line, split into the argv by the rule below; or, with **Run with the shell (/bin/sh -c)**, the line is the
  script itself, stored as the one `command` item as typed, and the shell expands it.
- **Working folder** (blank: the document's folder), **Environment** (names are letters, digits and `_`, not starting with a digit;
  each set once; a value may be empty), **Timeout** (1 to 3600 seconds, 60 by default) and **Turned on**.
- The **id** comes from the program's name (`/opt/tools/xcrun swift-format` gives `xcrun`), made unique in its list (`echo`,
  `echo-2`…). It follows the command while you add a hook and never changes once saved.

Each field shows what is wrong beside it, and **Save** stays disabled until nothing is: nothing is written before the hook
validates. A new hook writes only `id`, `event`, `command` and the fields you changed from their defaults. Editing starts from the
hook as written, so keys the form does not know (a `note`, your own keys) are kept, and saving an unchanged hook writes nothing.
**Edit as JSON…** closes the form and opens your `hooks.json` in its default app (created as `{"hooks": []}` if it is not there yet)
or, for a document hook, the document's hooks as a JSON array in a sheet that saves as one undo step. A hook the form cannot show (a
shell script with arguments, `shell: true` with more than one `command` item) opens with that explanation and Edit as JSON… only.

**The command line rule.** The line is split into words the way a small, predictable part of a shell would, and nothing is expanded:

- Outside quotes, whitespace (spaces, tabs, newlines) separates words.
- A single quote starts a literal part: everything up to the next single quote is taken as it is (`'$HOME'` is `$HOME`).
- A double quote starts a part in which only `\"` (a double quote) and `\\` (a backslash) are escapes; any other backslash stays
  with the character after it (`"a\nb"` is the four characters `a\nb`).
- Outside quotes, a backslash makes the next character literal (`x\ y` is the one word `x y`).
- Parts next to each other join into one word (`a"b c"d` is `ab cd`), and `''` or `""` is an empty word.
- No `$` expansion, globs, `~` or pipes: those need **Run with the shell**.
- Four errors, each shown under the field: an unterminated single quote, an unterminated double quote, a trailing backslash (one
  at the very end, escaping nothing), and an empty line.

A hook's argv is shown back the same way: a word with no whitespace, quote, backslash, `$` or backquote as it is, any other in
single quotes (a `'` inside written as `'\''`), so the line always splits back into the same argv.

### Duplicate, Delete and the switch

**Duplicate** puts a copy right after the hook, as written but for its id (`<id>-copy`, then `<id>-copy-2`…). **Delete** asks
first, naming the hook and its command. The switch rewrites `enabled`.

Your hooks are written to `hooks.json` whole (`{"hooks": [...]}`, pretty, sorted keys, replaced atomically; the folder and file are
created if missing), with every other hook kept as written, invalid ones too. A file Settings could not write back without losing
something is **refused and left as it is**: one with comments or that is not JSON, one that is not `{"hooks": [...]}`, or one with
another top-level key beside `hooks` (`unknown top-level key "extra" (only "hooks")`); Settings says why and offers Edit as JSON….
A document's hooks change through the document, one undo step each (Add Hook, Edit Hook, Duplicate Hook, Delete Hook, Turn On Hook,
Turn Off Hook).

**Trust.** A document's hooks list that you change in the form (add, edit, duplicate, delete or switch) is approved as you save it,
before the change reaches the document, but only when the list before the change was approved (an empty list counts): you wrote the
change, not the rest. A list that was not approved stays not approved and asks as before. Edit as JSON…'s sheet does not approve,
and undoing or redoing a form change leaves the list not approved until you review it. Your own hooks need no approval.

### Run Now

**Run Now** runs a hook once, as if its event had just happened, and shows how it went: the status (`exit 0`, `exit 1`,
`timed out`…), the time it took and the last 40 lines of its stdout and stderr. Nothing is saved or exported: an `afterSave` hook
gets the document's package as its one file (none without a document), an `afterExport` hook the first kind of its filter and no
files, an `afterExportCode` hook `kind: "code"` and the first target of its filter. Your hooks run without a document (no document
path, in your home folder unless `cwd` says otherwise); a document's need the document saved and its hooks approved (Run Now asks,
then runs). A hook that is switched off can still be run. The run goes to the log, and a failure shows the notice, like any other.

## Examples

Put each in `~/Library/Application Support/Silverpoint/hooks.json` (or the document's `hooks`).

**Copy `tokens.json` into a repo after a tokens export** (File ▸ Export Code or Tokens… ▸ Tokens: JSON). `$SILVERPOINT_HOOK_TOKENS_PATH`
is the file just written, so the hook needs a shell to expand it:

```json
{
  "hooks": [
    {
      "id": "sync-tokens",
      "event": "afterExport",
      "filter": ["tokens"],
      "shell": true,
      "command": ["cp \"$SILVERPOINT_HOOK_TOKENS_PATH\" /Users/me/code/my-app/Design/tokens.json"]
    }
  ]
}
```

With a fixed file next to the document you do not need a shell:
`{"id": "copy-tokens", "event": "afterExport", "filter": ["tokens"], "command": ["/bin/cp", "-f", "tokens.json", "../my-app/Design/tokens.json"]}`
runs `cp` in the document's folder.

**Run `swift-format` on the Swift just exported** (the SwiftUI code target). The files arrive newline-separated in
`$SILVERPOINT_HOOK_FILES` (and as an array on stdin), so a path with a space survives the `tr` and `xargs -0`:

```json
{
  "hooks": [
    {
      "id": "format-swift",
      "event": "afterExportCode",
      "filter": ["swiftui"],
      "shell": true,
      "command": ["printf '%s' \"$SILVERPOINT_HOOK_FILES\" | tr '\\n' '\\0' | xargs -0 /usr/bin/xcrun swift-format -i"],
      "timeout": 120
    }
  ]
}
```

**`git add` the exported files** (the code, tokens and handoff exports, into the repo they were written to):

```json
{
  "hooks": [
    {
      "id": "stage-exports",
      "event": "afterExport",
      "filter": ["code", "tokens", "handoff"],
      "shell": true,
      "command": ["printf '%s' \"$SILVERPOINT_HOOK_FILES\" | tr '\\n' '\\0' | xargs -0 /usr/bin/git -C /Users/me/code/my-app add --"]
    }
  ]
}
```

To stage everything in a folder instead, no shell is needed:
`{"id": "stage-all", "event": "afterExportCode", "command": ["/usr/bin/git", "add", "-A", "."], "cwd": "/Users/me/code/my-app/Sources/Design"}`.

## Not covered

- A shell by default: commands are argv; `shell: true` is an explicit choice (and shows in the approval sheet).
- `render` and `score` never run hooks.
- Moving or renaming a document does not carry its approval: a new path asks again.
- Other plugin kinds and a hook marketplace.
