---
title: "The saved agent conversation"
description: "A document keeps its agent conversation in its own package, so it travels with the file: close the document or quit, open it again, and the panel shows the conversation exactly as it was, and the agent goes on from where it stopped (BACKLOG 33). The code is AgentConversationArchive (the format) and…"
summary: "A document keeps its agent conversation in its own package, so it travels with the file: close the document or quit, open it again, and the panel shows the conversation exactly as it was, and the agent goes on from where it stopped (BACKLOG 33). The code is AgentConversationArchive (the format) and…"
layout: doc
toc: true
---

A document keeps its agent conversation in its own package, so it travels with the file: close the document or quit, open
it again, and the panel shows the conversation exactly as it was, and the agent goes on from where it stopped (BACKLOG 33).
The code is `AgentConversationArchive` (the format) and `DesignAgent+Persistence.swift` (restoring and saving) in
SilverpointAgent; SilverpointCore only carries the folder's bytes (`DocumentPackage.agentFolder`, `DesignStore.agentFiles`).

## Files

```
Poster.silverpoint/
  document.json
  agent/
    conversation.json
    images/<sha256>.png
```

- `agent/conversation.json`: the conversation, format version 1 (below). Encoded with sorted keys, so saving a conversation
  that did not change writes the same bytes.
- `agent/images/<sha256-hex>.<ext>`: each image the model was sent (screenshots, pictures of the selection, images a tool
  returned), named by the SHA-256 of its bytes, so an image sent twice is stored once (the package's assets pattern).

A conversation with nothing in it writes no `agent/` folder, and a template or a design-system package never gets one.
The conversation is saved with the package at each save (autosave included): it is document data, not an edit, so it has
no undo step. A change to it (a turn ends, a rating, a card's state, a queued or handed-back message, New Conversation,
Compact) marks the document edited through `DesignStore.changedWithoutUndo`; restoring it on open does not.

## Format 1

| Field | What it holds |
|---|---|
| `formatVersion` | `"formatVersion": 1` for this format; a reader ignores a folder of any other version. |
| `provider`, `model` | What the last turn ran on (informational; nothing depends on it). |
| `items` | The panel's items in order: `id`, `type` (`user`, `assistant`, `thinking`, `tool`, `notice`, `failure`, `questions`, `directions`, `skillProposal`, `lintRuleProposal`, `toolApproval`, `webSearch`, `reportedIssue`), that type's fields, and the `detail` and `citations` any item may carry. |
| `feedback` | The ratings, by the id of the item that ends the rated turn. |
| `skillCards`, `lintRuleCards`, `approvalCards`, `resolvedCards` | Where each card stands, by its item id. |
| `history` | The model history exactly as it was sent: `role` and `content` blocks (below). |
| `queued`, `unsent` | Messages sent while a turn ran, still waiting, and those a turn handed back to the composer. |
| `usage`, `lastRoundUsage`, `estimatedCost` | The conversation's tokens and cost so far, and the last request's usage (how full the context is). A usage has `input`, `output`, `cacheRead`, `cacheWrite` and `webSearchRequests`, plus two optional keys written only when present: `cacheWrite1h` (the one-hour share of `cacheWrite`) and `reportedCost` (the provider's own cost in US dollars, e.g. OpenRouter's). The cost's `kind` is `reported` (with `dollars`: every request's cost was reported by the provider), `estimated` (with `dollars`, from the price table, in part when only some requests reported), `unknown` or `notBilled`. |
| `sentFacts` | The turn facts' "last sent" record (BACKLOG 47): where each turn opens in `history` and the design-system and context sections it left the model with. |

The composer's draft is not saved.

The history is KBLLM's `Message`, whose `MessageContent` has four cases. They are five block types, one to one, plus the
placeholder:

| Block `type` | KBLLM `MessageContent` | Fields |
|---|---|---|
| `"text"` | `.text(String)` | `text` |
| `"image"` | `.image(data:mimeType:)` | `sha256`, `mimeType` (the bytes are `agent/images/<sha256>.<ext>`) |
| `"imageOmitted"` | `.text(placeholder)` | `mimeType` (an image the save left out; see Images) |
| `"toolCall"` | `.toolCall(ToolCallRecord)` | `id`, `name`, `arguments` (or `argumentsBase64` for bytes that are not UTF-8) |
| `"toolResult"` | `.toolResult(callID:, ToolResult)` | `callID`, `toolName`, `isError`, `content` (or `contentBase64`) |

An example (the tests decode it with the real decoder):

```json
{
  "approvalCards" : {},
  "estimatedCost" : { "dollars" : 0.0213, "kind" : "estimated" },
  "feedback" : {
    "6F1C2A1E-2B1B-4C7A-9E43-1D2F3A4B5C60" : { "rating" : "good", "reasons" : [] }
  },
  "formatVersion" : 1,
  "history" : [
    {
      "content" : [
        { "text" : "# This turn\nMode: Just do it\n\nMake a card", "type" : "text" },
        { "mimeType" : "image/png", "sha256" : "9f2c0d8e4b1a7c3e5f6a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e", "type" : "image" }
      ],
      "role" : "user"
    },
    {
      "content" : [
        { "text" : "Adding a card.", "type" : "text" },
        { "arguments" : "{\"operations\":\"[]\"}", "id" : "toolu_1", "name" : "batch_design", "type" : "toolCall" }
      ],
      "role" : "assistant"
    },
    {
      "content" : [
        { "callID" : "toolu_1", "content" : "{\"created\":[]}", "isError" : false, "toolName" : "batch_design", "type" : "toolResult" }
      ],
      "role" : "tool"
    },
    {
      "content" : [
        { "text" : "Here is the image you asked for.", "type" : "text" },
        { "mimeType" : "image/png", "type" : "imageOmitted" }
      ],
      "role" : "user"
    },
    {
      "content" : [ { "text" : "Made a card.", "type" : "text" } ],
      "role" : "assistant"
    }
  ],
  "items" : [
    { "hasImage" : true, "id" : "0D7E8C55-0F3A-4E7B-8D61-2B9A0C1D2E30", "text" : "Make a card", "type" : "user" },
    { "detail" : "arguments:\n{}", "id" : "1A2B3C4D-5E6F-4A7B-8C9D-0E1F2A3B4C50", "name" : "batch_design", "status" : "succeeded", "statusText" : "1 created", "summary" : "Card", "type" : "tool" },
    { "id" : "6F1C2A1E-2B1B-4C7A-9E43-1D2F3A4B5C60", "text" : "Made a card.", "type" : "assistant" }
  ],
  "lastRoundUsage" : { "cacheRead" : 0, "cacheWrite" : 0, "input" : 1500, "output" : 40, "webSearchRequests" : 0 },
  "lintRuleCards" : {},
  "model" : "claude-opus-5-5",
  "provider" : "anthropic",
  "queued" : [],
  "resolvedCards" : [],
  "sentFacts" : [ { "message" : 0 } ],
  "skillCards" : {},
  "unsent" : [],
  "usage" : { "cacheRead" : 0, "cacheWrite" : 0, "input" : 2500, "output" : 80, "webSearchRequests" : 0 }
}
```

## Compatibility

- A document with no `agent/` folder (saved before conversations were kept, or with none) opens as before, with an empty
  conversation and no notice.
- A `conversation.json` that is unreadable (corrupt, or a field this version cannot read), or whose `formatVersion` is newer
  than 1, is ignored: the document opens normally, and the panel shows one notice that the saved conversation could not be
  read. The folder is kept byte-identical, unchanged at every save, until a turn runs (which replaces it with format 1) or
  New Conversation is confirmed (which removes it). The document always opens: the conversation never stops it.
- An image file that is missing, or whose bytes do not match its hash, reads as an `imageOmitted` block.

## Images

The images a document keeps are capped at 20 MiB (20,971,520 bytes) in all. At each save the images are counted newest
first, each once by its content hash (an image sent twice counts at its newest place); the first that does not fit and every
older one are saved as `imageOmitted` instead. The model gets the placeholder text
`[Image omitted: an image was here but was dropped from the saved conversation to save space.]` in their place, so it
knows an image was there. The oldest go first; the live conversation keeps its images until the document is reopened.

## Settings switch

Settings ▸ Agent ▸ Saved conversation has the switch "Keep the agent conversation in the document", on by default
(`AgentPreferences.keepConversation`, the defaults key `silverpoint.agent.keepConversation`). Turn it off for files you
share. Off, nothing is written, a conversation change does not mark the document edited, a document opened with an
`agent/` folder restores nothing, and the next save of any open document removes its `agent/` folder (the switch is read
at every save).

## Compact

Compact runs only when the person asks: the ⋯ menu's "Compact Conversation…", or the Compact button of the notice that
shows when the last request read 80% or more of the model's context (`AgentContextLimits`, one bundled table). It is never
an automatic model call. The confirmation shows the cost first: the input it sends (the last request's input total, or the
history's text characters ÷ 4 before any usage was reported) and the estimated cost from the price table (tokens only for
a provider that does not bill per token). On Compact, one model call summarizes every turn but the last two turns; the
history becomes one user message opening `# Conversation so far` with that summary, then the last two turns verbatim. The
older tool results and images go; the panel keeps every item and adds one notice. `sentFacts` keeps the last two turns'
openings, without their sections, so the next turn sends every section in full again. It changes the start of the history,
so the prompt cache misses once. A failed call or an empty summary changes nothing and adds a failure to the panel. It is
not offered while a turn runs, with two turns or fewer, or on a CLI backend (the CLI keeps its own context).

## Providers

The stored history is provider-neutral: KBLLM's four content cases are text, tool calls (`toolCall`), tool results
(`toolResult`) and images, and nothing provider-specific (thinking signatures, encrypted search results) ever enters it
(web search findings go in as text). A conversation saved on one provider continues unchanged on another provider, or
another model of the same one: its next request is the same messages, nothing dropped or added. `provider` and `model` are
updated at the next turn's end, and the context limit follows the current model.

## CLI backends

On a CLI backend (Claude via your Claude Code, Codex via your Codex CLI) the panel's items are saved the same way, but the
CLI's own session cannot be resumed after a reopen: the first turn after the reopen starts a new session (a new process,
no `--resume`; a new Codex thread). Its message opens with `# Conversation so far`, then the last 20,000 characters of the
restored transcript (`transcriptText`), a blank line and the turn's usual message. That prefix goes only to the CLI: it is
never in the history, the panel or the log's `user` line. Later turns, and conversations that were not restored, send the
usual message.

## Byte preservation

The history is saved exactly as it was sent, block for block and byte for byte (tool arguments and results as their
bytes), with the turn facts' record, so a reopened conversation's next request is the stored history plus the new turn: the
same request body, byte for byte, as if the document had stayed open (the tests compare the real KBLLM Anthropic body). The
prompt cache's prefix therefore survives a reopen. Across turns, consecutive requests are compared message by message with
only the `cache_control` keys removed (the cache breakpoint moves to the newest message each request; nothing else may),
which is item 47's normalization.
