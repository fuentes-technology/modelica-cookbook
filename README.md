# Modelica Skills for Claude Code

Claude Code skills for authoring and validating [Modelica](https://modelica.org/) models with [OpenModelica](https://openmodelica.org/).

## Skills

- **[author-modelica-model](skills/author-modelica-model/SKILL.md)** — given a request to create or modify a Modelica model, writes the `.mo` file, then runs it through OpenModelica's check and simulate checkpoints, fixing and re-running until both pass (or reporting why they don't).

## Requirements

OpenModelica installed, with the `OPENMODELICAHOME` environment variable set (OpenModelica sets this automatically on install). The skill checks for this itself before doing anything else and stops with a clear error if OpenModelica isn't found, rather than guessing at an install path.

## Installation

This repo is a Claude Code plugin (and its own marketplace), so it installs with two slash commands:

```
/plugin marketplace add https://github.com/fuentes-technology/modelica-cookbook
/plugin install modelica-skills@modelica-cookbook
```

## Usage

Invoke it with `/modelica-skills:author-modelica-model`, then describe what you want, e.g.:

- "Create a Modelica model for an RC circuit with a step voltage source, a resistor, and a capacitor."
- "Modify WaterHeatingSystem.mo so the thermostat setpoint is 70°C instead of 60°C."
- "Update MyPump.mo to use a shorter stop time — I only need the first 10 seconds."

## A personal note

This is part of an ongoing experiment in using AI to author and validate Modelica models — how far it can go, where it breaks, what actually helps. If you've tried something similar, good or bad, I'd genuinely like to hear about it — please get in touch.

Thanks for stopping by — I hope you give them a try.
