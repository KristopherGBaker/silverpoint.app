---
title: "Release notes"
description: "What changed in each build of the Silverpoint beta, newest first."
summary: "What changed in each build of the Silverpoint beta, newest first."
layout: article
---

## 0.1.0, the first beta

Build 1, November 5, 2026.

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
