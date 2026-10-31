---
title: "Skills and house rules"
description: "A skill is a Markdown guide the agent loads on demand with readskill (how to build a landing page, how your brand writes, how a dashboard was laid out), or, when it is marked load: always, has in its system prompt from the first turn. Silverpoint ships some; you can add your own and override the bu…"
summary: "A skill is a Markdown guide the agent loads on demand with readskill (how to build a landing page, how your brand writes, how a dashboard was laid out), or, when it is marked load: always, has in its system prompt from the first turn. Silverpoint ships some; you can add your own and override the bu…"
layout: doc
toc: true
---

A **skill** is a Markdown guide the agent loads on demand with `read_skill` (how to build a landing page, how your
brand writes, how a dashboard was laid out), or, when it is marked `load: always`, has in its system prompt from the
first turn. Silverpoint ships some; you can add your own and override the bundled ones without touching the agent's core
prompt. The same skills reach the in-app agent, MCP clients and the `silverpoint` command line.

## Format

A skill is a Markdown file with an optional front matter block:

    ---
    title: Brand voice
    summary: How Acme writes: short, plain, no jargon. Read before writing any copy.
    ---
    # Brand voice
    Write in short sentences. Say "use" not "utilize". ...

- `title` is the human name. `summary` is one line, shown in the agent's index of skills (it decides when the agent
  reads the skill), so make it say what the skill is for.
- The text after the front matter is what `read_skill` returns.
- **Malformed files are tolerated.** The title is the `title:` value; without one, the first `# ` heading; without
  one, the skill's name. It is never empty and reading never fails. Front matter that has no closing `---`, or lines
  without a `:`, is read as far as it makes sense. A file with no front matter at all is a skill with no summary.
- A skill's **name** (its slug) is its file name without `.md`, lowercased, with runs of anything but letters and
  digits turned into one dash: `Brand Kit.md` is `brand-kit`. `read_skill` also accepts the title.
- `load: always` (the value in any case, spaces around it ignored) makes the skill **always loaded** (below). Any other
  value, an empty one or no `load` key leaves it on demand.

## Always-loaded skills

A skill marked `load: always` has its full text in the agent's system prompt, so the agent has it from the first turn
without a `read_skill` call (it was easy to skip: the agent read `design-principles` in 4 of 17 editing conversations, never
for an existing app).

- **Every layer can mark one.** A bundled, package or user skill marks itself in its front matter. A document skill has no
  front matter of its own, so its body opens with one block instead (`---`, `load: always`, `---`, then the text); the agent
  sees the text after the block, and the document keeps the body as written (it saves byte for byte as before).
- **The override decides for itself.** When a higher layer replaces a skill, the winning file's own key counts: a user copy
  of a bundled always-loaded skill without `load: always` is on demand, and a document copy with the block is always loaded
  even if the version below it is not. A skill that is turned off is never loaded.
- **The limit follows the model** (see [The budget per model](#the-budget-per-model)): 20 percent of its context window in
  characters, and 64,000 characters of always-loaded text in total when the model's window is not known. The marked skills
  are walked in a fixed order (by layer: bundled, package, user, document; then by name), keeping a running total. A skill
  that still fits is included; one that would pass the limit stays on demand (**Over Limit**), and the walk goes on, so a
  later, smaller skill can still be included. The bundled three come to about 49,900.
- **Where it goes.** The included skills follow the guide indexes in the stable part of the system prompt, before the house
  rules and anything per turn, under `# Always-loaded skills`, each as `## Skill: <name>` and then exactly the text
  `read_skill` returns. Nothing in the section changes between turns, so the prompt cache holds (item 47), and the prompt
  version covers it. `read_skill` still works for an always-loaded skill and returns the same text; the index still lists it.
  The Claude Code and Codex backends get the same section in their appended prompt.
- **MCP and `silverpoint instructions`** name the included skills and tell the client to read them with `read_skill` before
  its first edit (names only: the instructions stay small, and a client's own model decides when to read).
- **The log** says which and why: in the `skills` line a marked skill has `"load": "always"` or `"load": "overLimit"`.
- **Settings ▸ Skills** shows an **Always** badge on an included skill and an **Over Limit** badge (with a note under the
  list) on one the limit left out.

### The budget per model

The always-loaded budget is 20 percent of the agent's model's context window, converted to characters at 4 characters per
token: `tokens × 4 × 0.2`, rounded down, and never more than 1,000,000 characters. The windows come from a bundled table,
`ModelContextWindows` (`SilverpointAgent`), one entry per model the pickers offer, each with its source (Anthropic's models
overview, OpenAI's model pages, Apple's TN3193):

- Anthropic: `claude-opus-5-5`, `claude-sonnet-5-5`, `claude-fable-5-1`, `claude-haiku-5-5`: 1,000,000 tokens → 800,000 characters.
- Claude (via your Claude Code): `sonnet`, `opus`, `haiku`: 1,000,000 tokens → 800,000 characters.
- Codex (via your Codex CLI): `gpt-6.1-sol`, `gpt-6-astra`, `gpt-6-sol`, `gpt-6-luna`: 1,050,000 tokens → 840,000 characters.
- Codex (via your Codex CLI): `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`: 400,000 tokens → 320,000 characters.
- OpenAI: `gpt-6.1-sol`, `gpt-6-astra`, `gpt-6-luna`: 1,050,000 tokens → 840,000 characters.
- Apple Intelligence: `apple-intelligence`: 4,096 tokens → 3,276 characters.

- **A model the table does not list** (a custom model id, an OpenAI-compatible server or preset, whatever model it names) has
  a window that is not known, so its budget stays at 64,000 characters. So does everything that cannot know the model: MCP
  (the client's model is its own), `silverpoint instructions` and the tests.
- **The in-app agent** uses the model it runs on for the system prompt (the Claude Code and Codex sessions too), the log's
  `skills` line and the prompt version: a large-window model includes a skill too big for 64,000.
- **Apple Intelligence's** small window gives 3,276 characters, so the bundled `schema`, `operations` and
  `design-principles` are Over Limit there (the rows and the `skills` line say so), and the agent runs
  [the compact core](#the-compact-core) instead: a short prompt with a reference that replaces `schema` and `operations`,
  and no always-loaded skills at all.
- **Settings ▸ Skills** works out the budget for the model the agent is set to and says so in a note under the list:
  `Always-loaded skills: up to 800,000 characters with claude-opus-5-5 (20 percent of its 1,000,000-token context).`, or
  `(its context size is not known)` for a model the table does not list. The rows, the badges' help, the viewer and the Over
  Limit note and alert use that number.

### Load Always in Settings ▸ Skills

- **The pin.** A user or document skill's row has a **Load Always** pin between Edit and the switch, and the row's context
  menu has the same **Load Always** item. It shows the skill's own mark (a turned-off skill that is marked shows pinned) and
  adds or removes it. A bundled or package row has no pin: its state is read-only there, and **Customize…** makes a user copy
  that keeps the mark and can change it.
- **Only the `load` line changes.** In a user skill's file (whatever its name, `Brand Kit.md` included) the key goes into the
  existing front-matter block, just before its closing `---`, or a block `---` / `load: always` / `---` is added at the top;
  a `load:` line with another value is replaced in place. Turning it off deletes that line, and the block too when it was
  the only line in it, so on and then off gives back the original bytes. Line endings (CRLF stays CRLF), the other keys,
  their order and the body are untouched, and the file is written atomically, only when it changes. A document skill gets
  the same edit to the block at the top of its body, as one undo step: **Load Skill Always** or **Load Skill on Demand**.
- **An unclosed block is refused.** A skill whose first line is `---` with no closing `---` line is left exactly as it is:
  the pin shows the error naming the unclosed block, and nothing is written (a file that is not UTF-8 text likewise).
- **The budget.** With the default budget (a model whose window is not known), an always-loaded user or document row says
  `Uses 3,000 of 64,000 characters · 1,000 left`; an Over Limit row says `Needs 4,000 characters · 1,000 of 64,000 left`.
  With another model the limit is that model's (800,000 on Claude Opus 5.5). The badge's help on any row and the viewer say the same, and
  the viewer of an on-demand skill says how many characters it has and how many are left for always-loaded skills. The
  numbers are `SkillLibrary.alwaysLoadedBudget` and `alwaysLoadedSize`, the same count the prompt uses.
- **When it does not fit**, Load Always still writes the mark, and an alert says plainly that the skill is marked but stays
  on demand (Over Limit), how many characters it needs and how many are left, and to turn off Load Always on other skills to
  make room.
- **The document skill editor** has a **Load Always** checkbox that edits the block at the top of the text it shows, so Save
  writes the mark with the rest of the edit (one "Edit Skill" undo step); with an unclosed block the checkbox is off limits
  and the editor says why.
- **Import…** copies a file byte for byte, its mark included, and its message ends with the imported files that are
  marked: `Marked Load Always: tone.md.`

### The Skills header: count and filter

Beside Reveal in Finder and Import…, the header shows how many skills there are and how many are off (`9 skills · 1 off`,
or `9 skills` when none is), over every row, and a **Filter skills** field. Typing narrows the rows to those whose title,
name (slug) or summary contains the text, in any case and with or without accents (`resume` finds "Résumé layouts");
with no match the list says `No skills match “x”.`, and clearing the field shows every row again. The filter is only what
the window shows: nothing is saved.

## The compact core

A model whose window in `ModelContextWindows` is at most **32,000** tokens runs the compact core (`CompactCore`,
`SilverpointAgent`); today that is only Apple Intelligence (4,096 tokens). The full prompt with its always-loaded skills is
about 60,000 characters and the full tool catalog about 79,000, many times such a window, so the in-app agent sends it a
request cut down to one small edit. Every other model, a model the table does not list, MCP, the Claude Code and Codex
backends and `silverpoint instructions` get exactly what they got before.

- **The prompt** (`AgentPrompt.compactSystem`): who the agent is, reply in the person's language, the turn is one undo step;
  the few rules that matter most (inspect before editing with `get_document` at depth 1 and `get_nodes` by id, change only
  what is asked, real copy, one accent color, contrast 4.5:1 for body text checked with `inspect_layout`); the asking rule
  (one `ask_user` question, only when the edit cannot be done without it); the enabled guides' names on one line
  (`Guides (read_skill): …`, read only when a topic guide is essential); a **reference** that replaces `schema` and
  `operations` (the node types `frame`, `text` and `rectangle`, their common keys, and one `batch_design` insert, update and
  delete; 1,079 characters); and the house rules last, cut at **400 characters** with `…`. It has no always-loaded skills, no
  style index, and no section for the document's kind, connected tools, the profile or web search.
- **The tools**, in this order: `get_document` (the outline), `get_nodes` (nodes by id), `find_nodes` (a node by name or
  text), `batch_design` (the edit), `find_empty_space` (where a new top-level frame goes), `inspect_layout` (clipping and
  contrast without vision: the on-device model has none, so `screenshot` is not offered), `read_skill` (a topic guide,
  rarely) and `ask_user` (one question). `make_options` and `propose_directions` are multi-step, and the reference replaces
  `list_operations`; `report_issue` and the rest are left out. Each definition is derived from the shared catalog's by one
  function (`CompactCore.definition`): its description's first sentence (at most 160 characters; `inspect_layout`'s one
  sentence is longer, so it ends at its last clause boundary within them) and its parameters with every description removed.
  The tools run unchanged, and results stay small because the prompt asks for depth 1 and nodes by id. A call to any other
  tool is refused as `Unknown tool <name>.`, and external MCP tools are not offered in the compact core.
- **Compact Conversation** on such a model sends the same compact prompt and tools.
- **Visible:** the log's `skills` line has `"core": "compact"`, and the prompt version (the `conversation` and `skills`
  lines, a reported issue) is the compact prompt's own. Settings ▸ Skills says
  `Apple Intelligence runs the compact core: a short prompt and 8 tools, without always-loaded skills.` in place of the Over
  Limit note.
- **The measured size.** The first request, serialized as one compact JSON document (system text, the messages' text, and
  each tool's name, description and parameters), must take at most half of 4,096 tokens at a conservative 3 characters per
  token, 6,144 characters, leaving the other half for the turn: an empty canvas and "Make a pricing card" is **5,179**
  characters; 20 nodes with 2 selected, "Make the selected title larger" and 2,000 characters of house rules is **5,670**
  (the same request on Claude Opus 5.5 is 124,891). Of the first, the prompt is 2,353 characters and the 8 tools 2,532
  (`CompactCoreFitTests`).

## Bundled skills

Silverpoint ships 17 skills (`Packages/SilverpointKit/Sources/SilverpointAgent/Resources/Skills`). `schema`, `operations` and
`design-principles` are marked `load: always`, so they are in the prompt; every topic and platform guide is read on demand. The
agent's prompt and the MCP instructions say which to read: the topic guide that fits (`diagrams` for diagrams, flowcharts,
architecture and state machines), and for an app the platform guide by the device frame or the wording (iPhone or iPad, Mac,
Android or Pixel) or by the code target (SwiftUI or UIKit, Compose).

| Skill | What it covers |
| --- | --- |
| `schema` | every node type and property (always loaded) |
| `operations` | how `batch_design` works (always loaded) |
| `design-principles` | always loaded: a core every design follows (one most important thing per frame, spacing on a rhythm of the app's base unit, layout frames, components, linked text styles, variables, 4.5:1 body text, one dominant color with accents), then **New designs** (name a direction; follow a `Direction:` line, else state your own density, layout and motion; pair faces, pick a scale, stay off the default look) and **Existing apps** (the app's own direction, faces, scale and palette win), and the brand palette rule (`brand-drift`) |
| `ios-app` | iPhone and iPad: frames, safe areas, 44 pt targets, Dynamic Type, semantic colors, bars, sheets, Liquid Glass, SF Symbols, concentric radii |
| `macos-app` | Mac windows: the frame, the toolbar in the title bar, sidebar and split views, 28 pt controls, the macOS text styles, menus, density |
| `android-app` | Material 3: the frame and size classes, 48 dp targets, the type roles, color roles and tonal surfaces, shapes, navigation, FABs |
| `mobile-app` | an index that points to `ios-app` and `android-app` |
| `landing-page` | marketing and promotional pages |
| `web-app` | app shells, sidebars, tables, charts |
| `slides` | presentation slides |
| `design-system` | tokens, components and documenting them |
| `poster` | posters: a named design philosophy first, frame sizes, composition, type as image |
| `generative-art` | seeded, rule-based art from Silverpoint shapes: the seed and parameters first, deterministic variation |
| `diagrams` | architecture, flowcharts, state machines, ER models, trees, layer stacks, dependency graphs, data flows and sequences: when not to draw, choosing the type, the budget (about 9 nodes, 12 connectors, 2 accents) and splitting, style (palette variables, hairlines, one accent, a horizontal legend, the three diagram presets), connector rules, the place, `connect`, `layout_diagram`, screenshot, fix workflow, a checklist and notes per type |
| `existing-app` | surveying an attached app or repo, and which design principles hold for it |
| `export` | on demand: exporting code (SwiftUI, React, Compose, UIKit, HTML), importing code and diagrams, and publishing a website (an Inkwell theme) |
| `version-history` | explaining and reverting changes between versions |

The platform skills are written in Silverpoint's own words from Apple's Human Interface Guidelines and the Material 3 design
tokens, and the expressive ones from the ideas of public design skills; the research notes are in `docs/research/design-guidance/`.
`diagrams` takes its ideas from Cathryn Lavery's diagram-design (MIT), written in our own words, so it needs no notice.

## Where skills come from

| Layer | Where | Edited by |
| --- | --- | --- |
| Bundled | inside the app (`Resources/Skills`) | Settings ▸ Skills ▸ Customize… copies one into your folder as an override |
| Package | `skills/*.md` of each design system package the document links (`docs/design-system-packages.md`) | the package's author; read-only here (Settings ▸ Skills ▸ Customize… copies one into your folder as an override) |
| User | `~/Library/Application Support/Silverpoint/Skills/*.md` | you; the agent's `save_skill` with scope `user`; Import… |
| Document | `skills` in the `.silverpoint` package's `document.json` | the in-app editor; `save_skill` with scope `document` |

The folder can be moved with the `SILVERPOINT_SKILLS_DIR` environment variable (read by the app, `silverpoint` and
the tests, which point it at a temporary folder). Only `*.md` files in it count (not subfolders or hidden files).

**Precedence.** The same name in several layers resolves to the highest one: **document beats user, user beats a linked
package, and a package beats bundled.** A resolved skill records its source and, when it replaces a lower layer's, which one
(Settings shows Overrides Bundled, Overrides Package or Overrides User; `silverpoint skills` prints `(overrides bundled)`, for
example `package (overrides bundled)`). Settings' **Customize…** on a bundled or package skill copies it into the user folder
as an override and opens the copy (a copy of an always-loaded skill keeps `load: always`); **Reset** beside that copy's badges (or in the row's context menu) moves the copy to the Trash, so the original
applies again, edited or not. Document skills belong to one document: another document sees the user's or the bundled
one. Package skills belong to the documents that link the package: they are in `silverpoint skills <doc>`, `read_skill` and the
agent's list while the document links the package, and gone when it does not (installing a package adds nothing by itself).

**Collisions inside one folder.** `Brand Kit.md` and `brand-kit.md` are the same name. The first file name in byte
order wins; the other is skipped and reported as a warning (on stderr in `silverpoint skills`, in the Settings card).
The resolved list never holds two skills with one name.

**Document skills** are stored in `document.json` under `skills`:

    "skills": {"brand-voice": {"title": "Brand voice", "summary": "...", "body": "..."}}

The key is absent when a document has none, so documents from before skills existed open and save exactly as they
were (including `.vellum` ones).

## Turning skills off

A disabled skill is hidden from the agent's index and from `read_skill`, whatever its source; a bundled skill can be
disabled too. The list is a user setting kept in your folder as `disabled.json`, a JSON array of names:

    ["slides", "brand"]

Settings ▸ Skills writes it with each skill's switch, and `silverpoint` and MCP read it, so a skill that is off is off
everywhere. A missing folder or a `disabled.json` that is not a JSON array of strings counts as empty: nothing is
disabled and nothing fails.

**Style presets** (`get_style`) are layered too, but with their own folder and no document layer: see
[templates-and-styles.md](../templates-and-styles/).

## Viewing a skill

Every row of Settings ▸ Skills (bundled, package, user or document) has **View**: a read-only sheet with the skill's title,
name, source, summary and how it reaches the agent (Always loaded, On demand or Over limit), and its text rendered as
Markdown (Preview only; the text is exactly what `read_skill` returns). When another layer has the same skill, a line says
**Overrides <layer>** (on the version that wins) or **Overridden by <layer>** (on the one it replaces), with a **View <layer>
Version** button that shows that version in the same sheet. Viewing never writes a file or touches the document. Bundled and
package rows have **Customize…** (it writes the user copy and opens it); user and document rows keep **Edit**.

## What the conversation log records

Each in-app conversation's log (`~/Library/Logs/Silverpoint/agent/`, one JSONL file per conversation) says which skills
the agent was offered and which it read, on both backends (the API and Claude Code):

- **A `skills` line** comes right after the `conversation` line:
  `{"event":"skills","skills":[…],"promptVersion":"…"}`. It has one entry per resolved skill, turned-off ones included,
  in slug order:
  - `slug` and `title`;
  - `source`: `bundled`, `package`, `user` or `document`, the layer that won the slug;
  - `enabled`: false when the skill is turned off (not offered);
  - `hash`: the first 12 hex characters of SHA-256 over its title, summary and body, each followed by a 0 byte, so an
    edit shows up without the log keeping the text;
  - `load`, only for a skill marked `load: always`: `always` when its text is in the prompt, `overLimit` when the
    model's always-loaded limit left it on demand. An on-demand skill has no `load`.

  `promptVersion` is the version over these skills (below). On a model that runs [the compact core](#the-compact-core)
  the line also has `"core": "compact"`, and the version is the compact prompt's.
- **It is written again only on a change.** At each later turn's start the skills are resolved again. When anything in
  the list differs (a skill turned on or off, a user skill added, edited or removed, the document's skills changed, a
  package linked or unlinked), or the model moved onto or off the compact core, a new `skills` line comes before that
  turn's `user` line. When nothing changed, no line
  is written. The last line is what was offered when the conversation ended.
- **The prompt version** (the `conversation` line's `promptVersion`, each reported issue's, and what evaluations are kept
  by) is a hash of the standing prompt and the offered skills, wherever they come from. A skill turned off drops out. So
  two conversations with different skills have different versions. The bundled skills alone give the same version they
  always did, so older evaluations still match.
- **Skills read** come from the log's `read_skill` tool lines. A call that failed (`isError`) does not count. Each read
  takes its title and source from the `skills` line in force at the time.

Conversation History shows this:
- the selected conversation's header says "Offered 9 · read 2" and lists what was read, most read first, as
  `Schema (bundled) ×2, Brand voice (user) ×1`;
- each row notes "2 skills read", with the titles as its tooltip;
- the conversations list has a **Read skill** filter, and so does the Evaluations segment (turns graded from
  conversations that read the skill).

A log written before the `skills` line still opens. Its header says only "Read 1", and its reads show the slug with no
source. Older readers skip the line, as they skip any line they don't know.

## Saving skills from the agent: `save_skill`

`save_skill` takes `scope` (`document` or `user`), `name`, `title`, `summary`, `body`, an optional `load` and
`confirm`. It is one tool
in the shared catalog, so the in-app agent, MCP clients and `silverpoint call` all have it.

- **In the app** it never writes. The agent panel shows a card with the title, the scope and the full text (or, when
  the skill replaces another, a diff against that version) and Save and Discard. Save writes it: a document skill as
  one undo step, a user skill as `<name>.md` in your folder. Discard writes nothing. The next turn lists the new skill.
- **Over MCP and `silverpoint call`** the client is the one that confirms. Without `confirm: true` the tool returns a
  preview and writes nothing; with it, it writes. MCP can write **both** scopes (a user skill is a file in your folder,
  a document skill goes through the document's undo like any edit).
- **`load` keeps the mark.** Without it, rewriting a skill keeps the choice of the version it replaces (a marked user,
  package or bundled skill stays `load: always`), and a new skill is on demand. `"always"` marks it, `"onDemand"` clears
  it; any other value is a tool error and writes nothing. A marked document skill's body opens with the `load: always`
  block. The diff compares the mark too (`+load: always`, `-load: always`), and the card says **Load Always** when the
  saved skill would be marked.
- The result is `{written, scope, slug, title, replaces, text, diff, load}`: `replaces` is the layer of the version it
  replaces (or null), `text` the whole file, `diff` a unified diff against that version (or null), and `load` is
  `always` or `onDemand`, what the saved skill is.
- An empty body, a scope that is not `document` or `user`, or a name with no letters or digits is a tool error and
  writes nothing.

## House rules

Settings ▸ Skills has a **House rules** field: a few lines the in-app agent follows in every document. They are
appended to its system prompt under a `# House rules` heading, after the guides and the always-loaded skills. Empty (or only whitespace) adds
nothing, so the prompt is exactly what it would be without them. They are for the in-app agent only: they are not sent
to MCP clients or shown by `silverpoint instructions`. Skills are the main way to extend the agent; house rules are
for the one-liners.

## Command line

*Not in the beta yet: the `silverpoint` command-line tool doesn't ship in the download, so this section describes it for builds made from source.*

    silverpoint skills [<file.silverpoint>] [--json] [--show <name>]
    silverpoint tools [<file.silverpoint>] [--json]
    silverpoint instructions [<file.silverpoint>]

`skills` lists one line per skill, `slug<TAB>source[ (overrides X)][ (disabled)]<TAB>title`, or with `--json` an array
of `{slug, title, summary, source, overrides, enabled}`; `--show <name>` prints the text `read_skill` would return
(exit 1 for an unknown or disabled name). With a document the list includes its own skills. All three only read.
`tools` and `instructions` with a document list that document's skills in `read_skill`'s description and, for
`instructions` (and `mcp`'s `initialize` reply), in one added line naming the enabled skills that are not bundled.
`instructions` names the always-loaded skills (with a document, its own marked ones too) in its read-before-the-first-edit
sentence, in the prompt's order; a turned-off one is not named.
