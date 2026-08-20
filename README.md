# Modelica Skills for Claude Code

Claude Code skills for authoring and validating [Modelica](https://modelica.org/) models with [OpenModelica](https://openmodelica.org/).

## Skills

- **[author-modelica-model](skills/author-modelica-model/SKILL.md)** — given a request to create or modify a Modelica model, writes the `.mo` file, then runs it through OpenModelica's check and simulate checkpoints via the `omc` CLI, fixing and re-running until both pass (or reporting why they don't).
- **[author-modelica-model-mcp](skills/author-modelica-model-mcp/SKILL.md)** — the same authoring/check/simulate loop, but through OpenModelica's live OMEdit MCP session instead of `omc`: the model is composed and pushed straight into a running OMEdit, so you watch it appear in the GUI. Can save the result to a `.mo` file on request.

## Requirements

Both skills need OpenModelica installed; which flavor depends on which skill you use.

- **author-modelica-model** needs the `OPENMODELICAHOME` environment variable set (OpenModelica sets this automatically on install). The skill checks for this itself before doing anything else and stops with a clear error if OpenModelica isn't found, rather than guessing at an install path.
- **author-modelica-model-mcp** needs OMEdit running with its MCP server started, and that server registered as an MCP server in Claude Code. This is an undocumented, prototype OpenModelica feature — not a stable, documented interface like `omc` — so expect rough edges; the skill checks for the tools it needs via `ToolSearch` and stops with a clear error if they're not there, rather than guessing at a URL.

## Installation

This repo is a Claude Code plugin (and its own marketplace), so it installs with two slash commands:

```
/plugin marketplace add https://github.com/fuentes-technology/modelica-cookbook
/plugin install modelica-skills@modelica-cookbook
```

## Usage

Invoke `/modelica-skills:author-modelica-model` (the `omc`-CLI skill) or `/modelica-skills:author-modelica-model-mcp` (the live OMEdit skill), then describe what you want, e.g.:

- "Create a Modelica model for an RC circuit with a step voltage source, a resistor, and a capacitor."
- "Modify WaterHeatingSystem.mo so the thermostat setpoint is 70°C instead of 60°C."
- "Update MyPump.mo to use a shorter stop time — I only need the first 10 seconds."

Reach for the CLI skill when you want a stable, scriptable, headless flow; reach for the MCP skill when you want to watch the model get built live in OMEdit's GUI as it happens.

## A personal note

This is part of an ongoing experiment in using AI to author and validate Modelica models — how far it can go, where it breaks, what actually helps. If you've tried something similar, good or bad, I'd genuinely like to hear about it — please get in touch.

Thanks for stopping by — I hope you give them a try.
