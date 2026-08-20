---
name: author-modelica-model-mcp
description: Authors a Modelica model through OpenModelica's live OMEdit MCP session and validates it (check + simulate); can save the result to a .mo file on request.
disable-model-invocation: true
---

Authors a Modelica model purely through OpenModelica's OMEdit MCP server. Claude doesn't write a `.mo` file directly — the model is composed and pushed straight into the live OMEdit session, so the user watches it appear in the GUI. See [`../shared/checkpoint-loop.md`](../shared/checkpoint-loop.md) for the checkpoint loop this skill follows.

This is a genuinely undocumented, prototype feature of OpenModelica — tool names and exact behavior aren't confirmed anywhere public. Where a step below says "if X fails because Y, do Z", that's a defensive fallback, not confirmed behavior; note in the final report which fallbacks actually fired so the skill can be tightened from real tool output.

## 1. Locate the MCP tools

Use `ToolSearch` to find the deferred tool names for at least `checkModel`, `simulate`, `setSourceCode`, `createClass`, `getClassNames`, and `getSourceCode` (needed in [step 2](#2-resolve-the-model) and [step 5](#5-save-to-disk-optional)). If none turn up, stop and tell the user OMEdit needs to be running with its MCP server started and registered with Claude Code — don't guess a URL or attempt a raw HTTP call. Done when the schemas for those tools are loaded and callable.

## 2. Resolve the model

Work through the shared checklist in [`../shared/resolve-modelica-model.md`](../shared/resolve-modelica-model.md). There's no local file path to reason about here — identify the model purely by its fully-qualified class name. Call `getClassNames` (and `getSourceCode` if it's listed) to check whether that class already exists in the live session before deciding whether this is a create or an edit; if it exists and the user's request doesn't make clear they want it changed, confirm before overwriting it. If the user gave a stop time and/or tolerance, they get baked into the model's own `annotation(experiment(StopTime=..., Tolerance=...))` in [step 3](#3-compose-and-push-the-model) — the `simulate` MCP tool takes only a class name.

## 3. Compose and push the model

Compose the Modelica source for the resolved model in your response, following [`../shared/model-authoring-style.md`](../shared/model-authoring-style.md) — reference `Modelica.*` components directly, OMEdit already has MSL loaded in its own session.

Push it into the live session: call `createClass` (using the resolved model name and specialization, e.g. `model`) if [step 2](#2-resolve-the-model) found no existing class, then call `setSourceCode(className, code)` with the composed source. If the class already existed, skip `createClass` and call `setSourceCode` directly.

The MCP tool set also includes diagram-editing tools (`addComponent`, `addConnection`, `setElementModifierValue`, etc.) for building a composed model visually, component by component, instead of as raw equation text. Only reach for these if the user specifically asks for the model to be assembled that way — the default path here is textual, same as every other `author-modelica-model*` skill.

## 4. Check and simulate checkpoints

Call `checkModel(className)` and check the result against the pass criteria in [`../shared/checkpoint-loop.md`](../shared/checkpoint-loop.md). On failure, fix the model ([step 3](#3-compose-and-push-the-model)) using the error text and rerun from the `setSourceCode` call.

Once check passes, call `simulate(className)`. Pass means the response contains no error. On failure, fix the model and rerun from [step 3](#3-compose-and-push-the-model).

## 5. Save to disk (optional)

If the user wants the model as a `.mo` file — or asked for one to begin with — there's no OMEdit save/export tool in the discovered MCP set to do this directly. Instead, call `getSourceCode(className)` and write the returned source to the target path yourself with the Write/Edit tool.

This is a snapshot of the textual model only: GUI-only edits made after the pull (diagram layout, icon, connector placement) aren't captured unless you re-pull the source, or the user saves from OMEdit's own File > Save. Mention this in the report if the user is likely to keep editing visually afterward.

## 6. Report

Work through the shared checklist in [`../shared/report-checklist.md`](../shared/report-checklist.md), plus things specific to this MCP backend:

- The fully-qualified model name, and whether/where it was saved to disk ([step 5](#5-save-to-disk-optional)).
- The MCP tools used ([step 1](#1-locate-the-mcp-tools)).
- Which defensive fallbacks from [step 3](#3-compose-and-push-the-model) actually fired.

For the shared checklist's substantive result line, call `getSimulationResultVariables(className)` (or `showPlot`) after the passing simulate to pull a key variable's value or how it changed.

There's nothing else to clean up — nothing was written to the scratchpad, and the only local disk write is the optional `.mo` file from [step 5](#5-save-to-disk-optional).
