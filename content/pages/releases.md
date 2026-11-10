---
title: "Release notes"
description: "What changed in each build of the Silverpoint beta, newest first."
summary: "What changed in each build of the Silverpoint beta, newest first."
layout: article
---

## 2026.46.1

November 10, 2026.

The third beta build. Installing it starts the beta's 60 days again.

### New in this build

- **Slash commands in the composer.** Type / in the agent panel to see the commands. /variants, /critique, /fix-lint and /handoff set up a turn for you. /history, /attach, /export and /new act at once and say what they did. A document can carry its own commands, and you can add your own as command files.
- **Critique without changes.** /critique only looks at the design, with every backend. On Claude Code and Codex, that turn has no tool that can change the design.
- **A closer design review.** The review now judges in two stages and checks the design against what you asked for. Its item in the agent panel shows the outcome, a fidelity table and the personas it reviewed as. In a second round, it shows which findings were fixed. A finding that is still open keeps blocking.
- **Craft checks.** Nine new lint rules point out common weak spots: a flat hierarchy, an uneven heading rhythm, long lines, tight line spacing, cards inside cards, gray text on color, gradient text, emoji used as icons and gaps that are all the same. They only advise, and Fix Lint leaves them alone. Settings lists them in their own group, in English and Japanese.

### Also

- The agent steers away from generic looks, and plans color, surfaces and interactive states more carefully.
- A design with nested components no longer crashes Silverpoint while the agent edits it.
- If a reply reaches the model's output limit before the agent acts, the agent carries on instead of stopping silently.
- The design review has room to finish with reasoning models that think at length.

## 2026.45.2

November 8, 2026.

The second beta build. Installing it starts the beta's 60 days again.

### New in this build

- **Design review.** Turn it on in Settings ▸ Agent, or for one turn with **Review this turn** in the composer. After a turn that changed the design, a second model reviews it. The agent then fixes whatever the review says must change, in up to two rounds. It works with every backend, including Claude Code and Codex.
- **Ask first on Claude Code and Codex.** Those backends now ask questions and offer directions as cards too, and can report a problem with Silverpoint.
- **Web search on chosen sites.** Settings ▸ Agent can limit the agent's web search to a list of sites, or block a list.
- **Spacing guides.** While you drag, layers snap to equal gaps between their neighbors, with markers and distances shown.
- **Shortcuts, Siri and Spotlight.** Export a frame, save a version, check a design, ask the agent, open a document or insert a component from Shortcuts. Spotlight finds your documents and components.
- **Images from your own service.** An image provider plugin, or custom headers and an optional key for an OpenAI-compatible image endpoint. The ✨ button shows what an image cost.
- **Your font pairings and palettes in Settings**, with each palette checked for WCAG AA contrast. The agent can save a font pairing for you.
- **Code import reads more.** iOS storyboards and XIBs come in as frames, with segues as prototype links. SwiftUI lists and `ForEach` come in as rows.
- **Devices.** A landscape iPhone Duo frame can put its system strip on either side, and the prototype player can fold and unfold a closed and open pair.
- **A tidier toolbar** with Shape and Drawing menus, and a collapsed sidebar you can drag back out.

### Also

- Export Code no longer blocks the app, and a slow plugin can be cancelled.
- Passwords and tokens for connected MCP servers are kept in the Keychain.
- Image tracing, region detection and large first layouts no longer hold up the window.
- A hook that times out, or is still running when you quit, ends together with anything it started.
- Early: on macOS 27, the canvas takes direct touch (select, move, resize, draw, scroll and pinch).

## 2026.45.1, the first beta

November 5, 2026.

Silverpoint is a native Mac design canvas with an AI designer built in. Describe a screen and the agent builds it on the canvas; edit it by hand or keep talking to it. It is free while it is in beta.

### In this build

- **The design agent.** It builds frames, components and variables in your document, asks first or just works, and every turn is one undo step. It runs on what you bring: your Claude Code or Codex, an API key, Apple's on-device model or a local server.
- **A real canvas.** Auto layout, components and variants, text styles and variables, vector editing, comments and prototyping.
- **Every version kept.** Name a checkpoint, compare any version with your design side by side or as an overlay, and restore all of it or only some frames.
- **An MCP server for your own agent.** Claude Code, Codex or any MCP client can design in your open documents with the same tools (Settings ▸ MCP).
- **Design systems as packages**, **skills and lint rules** that teach and check the agent, **hooks** after saves and exports, and **command plugins**.
- **Publish a website** from a design as an Inkwell theme, in more than one language.
- **Export** frames as PNG, PDF or SVG. Code export and import (SwiftUI, UIKit, Compose, React, HTML) and Figma import work too, but are less tested: expect rough edges.

### Good to know

- Each beta build works for 60 days after it is built. Installing a newer one starts the 60 days again; after that, designs open read-only until you update. Your files never change because of it.
- The app checks for updates once a day; **Silverpoint ▸ Check for Updates…** checks now.
- English and Japanese only. Only the fonts installed on your Mac are available. No sync or collaboration yet.
