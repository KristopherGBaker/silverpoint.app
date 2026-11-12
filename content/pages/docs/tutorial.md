---
title: "Tutorial: an app and its website, start to finish"
description: "Design Tally, a habit tracker's iPhone app and its one-page website, with the agent, touching most of Silverpoint on the way."
summary: "Design Tally, a habit tracker's iPhone app and its one-page website, with the agent, touching most of Silverpoint on the way."
layout: doc
toc: true
---

You will design **Tally**, a habit tracker: two iPhone screens and a one-page website, mostly by asking the agent and
reading what it shows you. Each section names what you type or click and what you then see. Set the agent up first
([Getting started](../getting-started/)); any backend works.

The project's state at the end of each section is in the Silverpoint repository, `Examples/tutorial/01-start.silverpoint`
to `08-handoff.silverpoint`, rebuilt by `Examples/tutorial/recipe.sh` through the same tools the agent uses, so you can
open the state a section ends in and carry on from there.

## 1. Start from a template

Choose **File ▸ New from Template ▸ Mobile App (iPhone screens)**. A document opens with two empty iPhone frames side by
side, `iPhone 1` and `iPhone 2`. In the Layers list, rename them **Home** and **Habit**: the agent finds frames by name,
and so do the rest of these steps.

![The Mobile App template: two empty iPhone frames named Home and Habit on the canvas](/assets/images/tutorial/01-template.png)

## 2. The first turn: a style, Ask first and its cards

Open the agent with **View ▸ Show Agent** (⌘J). In the composer, type `/mode ask` and press Return: the agent now asks
before it designs (the composer's mode menu shows **Ask first**; `/mode do` is **Just do it**).

Type `/` to see every command, then send:

    /style warm-organic Design Tally's Today screen: today's habits, a progress summary and the streaks.

`/style` loads the Warm Organic preset for this turn: moss greens on pale celadon, Didot over Avenir Next. In Ask first
the agent does not draw yet. It ends its turn on a **question card**, "Silverpoint Agent asks", with suggested answers.
Pick or type yours in the card and press **Send**, or just reply below in the composer.

![The agent panel: the /style message, then a question card asking who Tally is for and what Today should lead with](/assets/images/tutorial/02-question-card.png)

Then it sketches three directions on the canvas, `Home · Option A`, `B` and `C`, and ends on a **directions card**,
"Silverpoint Agent proposes", each direction with its trade-offs. Click the one you want; the agent keeps that sketch and
designs it in full.

![Three sketched directions beside each other on the canvas, and the directions card in the agent panel](/assets/images/tutorial/02-directions-card.png)

The result is the Today screen in auto layout: a header, a progress card and the habit rows, every color a variable from
the style (`background`, `surface`, `ink`, `muted`, `accent`, each with a light and a dark value).

![The finished Today screen: greeting, a 3 of 5 progress card and three habit rows](/assets/images/tutorial/02-today.png)

To explore before you commit, select a frame and send `/variants 3`: the agent makes three options of it side by side
(the original and two copies, with `make_options`) and changes each in its own direction.

## 3. Components and variants

Every habit row looks the same, so make it a component. Send:

    Make the habit row a component with two variants, done and to do, and use it for every row. Add walking and "no phone after 10".

The agent adds a **Habit row** component set beside the screens, `Habit row · Done` and `Habit row · To do` (the variant
`state` is `done` or `todo`), and replaces the rows with five instances, each with its own label and streak. Edit a
component and every instance of it follows; ask "mark stretching done" and the agent switches that instance to the done
variant (`set_variant`), keeping its label.

![The Habit row component set: a done row with a filled check and a to-do row with an outlined one](/assets/images/tutorial/03-habit-row.png)

## 4. Variables and themes

Click the empty canvas and open the **Design** tab: with nothing selected, its **Variables** section has **Edit Variables
& Themes…**, and the style's colors are there, each with a value for `light` and `dark` on the `mode` axis. **Theme
Preview** switches the canvas between them.

Ask for one more:

    Add a streak color, amber in light and a warmer gold in dark, use it for the streak labels, and show the Habit screen in dark.

The agent defines `streak` with a value per mode, binds the streak labels to `$streak`, and sets the Habit frame's theme
to `mode: dark`, so you see both at once.

![Home in the light theme beside the Habit screen in the dark theme, and the Habit row set](/assets/images/tutorial/04-themes.png)

## 5. The website: auto layout and grids

Now the one-page site, on the same variables:

    Now Tally's one-page website, on the same variables: a hero, the art, three features.

The agent adds a 1440-wide **Site** frame. The frame stacks its sections in a vertical auto layout, and its nav and hero
are horizontal ones, so they keep their spacing as their text changes; the three features sit on a **grid** of three
`1fr` columns, each card filling its column, so the three are one width and their text wraps inside them; and a
12-column layout grid over the frame shows the guides it was built on.

![The website: Tally's nav, the hero headline with its art slot, and three feature cards on a three-column grid](/assets/images/tutorial/05-website.png)

## 6. Critique, the design review and lint

**The design review.** In Settings ▸ Agent ▸ Design review, choose a reviewer model. From then on, a turn that changes the
design is reviewed by that second model when it ends (turn the composer's **Review this turn** switch off to skip one).
With a reviewer set before the website turn of section 5, that turn ends with **Reviewing…**, then a review item,
"Reviewed by …", with its findings and **Outcome: Ship** or **Outcome: Fix**; a fix gets one follow-up turn of its own.

![The review item in the agent panel: two suggestions for the website and Outcome: Ship](/assets/images/tutorial/06-design-review.png)

**A critique.** Select Home and send `/critique contrast and hierarchy`. This turn only looks: it is offered only the
tools that read, so nothing on the canvas changes. It answers with strengths and issues, each with a fix, and ends on a
question card about what to change.

![The /critique turn: strengths, three issues with fixes, and a question card](/assets/images/tutorial/06-critique.png)

Answer "Yes, both" and the agent darkens the date and the progress caption to `ink`.

**Lint and the craft checks.** Send `/lint`: the findings show in a note above the composer (with nothing selected, the
Design tab's **Design Check** has **Run** too). `/fix-lint` applies every mechanical fix and says how many it applied and
what is left. The craft issues stay: they are judgment calls, never fixed for you.

**Version history.** Every turn is a version. **File ▸ Version History…** lists them by day; pick one to compare it with
now, and **Restore Version** puts it back (**Restore Selected Layers** only the frames you switch on). **File ▸ Save
Version…** (or `/version before shader`) names the current state.

![Version History over the window, listing the agent's turns as versions](/assets/images/tutorial/06-version-history.png)

## 7. A shader, from Claude Code over MCP

The agent in the panel is one way in; Claude Code and Codex are another. Choose **Help ▸ Connect Claude Code or
Codex…**: Settings ▸ MCP shows the line to add to your client. With the document open, ask Claude Code:

    In my open Silverpoint document, put the Mesh Gradient shader preset on Tally's hero art.

It calls the same tools the panel's agent does: `list_shaders` with presets, `set_shader` with `{"preset": "mesh-gradient"}`,
then `batch_design` to set the `Hero art` frame's fill to that shader. The arch on the website now holds the mesh
gradient. In the app, select a node and choose **Shader…** in its fill's paint menu to do the same by hand.

![The website with the Mesh Gradient shader filling the arched hero art](/assets/images/tutorial/07-shader.png)

## 8. Hand-off: code, the guide and a version

Select the Today screen and open the **Inspect** tab: it shows the code for the selection.
**File ▸ Export Code or Tokens…** (⌥⇧⌘E) writes SwiftUI, HTML and the other targets, with the variables as tokens.

![The Inspect tab beside the canvas, showing the code for the selected frame](/assets/images/tutorial/08-inspect.png)

Before handing off, ask the agent to write the document's guide for the developer:

    Write the design guide: what Tally is, the variables, the Habit row component, the grid and the type.

Then send `/handoff ~/code/tally` (or **File ▸ Hand Off to…**): Silverpoint writes the hand-off bundle into that folder,
and the note says where and offers **Copy Prompt**, the prompt to give your coding agent. `/version Hand-off` marks the
state you handed off.

## 9. Publish the website

Choose **File ▸ Publish Website…**. Under **Project**, **Choose…** an empty folder for a new Inkwell site, check the
**Files to write**, then press **Publish**: Silverpoint builds the site with your installed `inkwell`, and **Preview**
opens it.

## What changed

What changed: written against Silverpoint 2026.46.1, the first version of this tutorial.
