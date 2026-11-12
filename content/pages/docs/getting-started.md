---
title: "Getting started"
description: "Install Silverpoint, make your first design with the agent, and connect Claude Code or Codex to it over MCP."
summary: "Install Silverpoint, make your first design with the agent, and connect Claude Code or Codex to it over MCP."
layout: doc
toc: true
---

## Requirements

- **System:** macOS 26 or later.
- **Price:** free during the beta.

## Download

Download the beta from [silverpoint.app/download](https://silverpoint.app/download/), open the DMG and drag Silverpoint to
Applications. The app is notarized and keeps itself up to date: it checks once a day (Settings ▸ General ▸ Updates), and
**Silverpoint ▸ Check for Updates…** checks now.

## Your first design

1. Open Silverpoint. The first time, a setup sheet opens over the Launch window. Press **Set up the agent** and pick how it
   runs: Claude (via your Claude Code), Codex (via your Codex CLI), an API key, Apple's on-device model or a local server.
   **Check** tries it once; nothing is sent to a model before you press it. Skip works on every step.
2. Choose whether the agent asks first or just does it, then press **Try it**: a new design opens with a sample brief in the
   agent's composer. Press Send when you are ready. (Finish closes the sheet instead.)
3. Describe what you want next, such as "a settings window for a podcast app", and watch it draw frames on the canvas.
   Each change is saved in Version History, so you can always go back.
4. The Launch window's **Get started** checklist lists six more things to try and ticks each one as you do it. Dismiss it
   with its ×; Settings ▸ General ▸ **Show Get started checklist** brings it back, and **Help ▸ Welcome to Silverpoint…**
   opens the setup sheet again.
5. A design opened from the Launch window (a recent, Open…, a template) takes the Launch window's size and position, so
   enlarge it first if you want a bigger window. When the last document window closes, the Launch window comes back
   (unless Settings or another window that stays open on its own is up); **Window ▸ Home** (⌃⌘H) shows it any time.

The [agent conversation](../agent-conversation/) is saved with the document, so it travels with the file.

## Connect Claude Code or Codex

In the app, **Help ▸ Connect Claude Code or Codex…** opens Settings ▸ MCP, which serves the open documents' design tools
and shows the line to add to your client.

Add it to Claude Code with the line Settings ▸ MCP shows, then ask Claude Code to design in your open document.

Then read on:
[skills](../skills/) teach the agent your conventions, [templates and styles](../templates-and-styles/) start new documents,
and [design-system packages](../design-system-packages/) share components between them.

Next: the [tutorial](../tutorial/) designs an app and its website from start to finish.
