---
name: author-modelica-model
description: Authors a Modelica model (.mo file) and validates it with OpenModelica (check + simulate).
disable-model-invocation: true
---

Writes a Modelica model (.mo file), then tests it with OpenModelica's `omc`. The model has to load before it can pass the two checkpoints that matter:

- **check**: parses and verifies the model is balanced
- **simulate**: runs it end to end

A model isn't done until it loads cleanly and passes both checkpoints. If any step fails, fix the problem and rerun. Never hand back code that hasn't been checked. Stop after three fix/rerun cycles total without a full pass, and report the remaining error to the user instead of looping indefinitely.

## 1. Locate omc

Read the `OPENMODELICAHOME` environment variable — OpenModelica sets it on install, and `omc` lives at `$OPENMODELICAHOME/bin/omc` (`omc.exe` on Windows). If it's unset, stop here and tell the user OpenModelica isn't set up rather than guessing at an install path — don't spend effort writing a model that can't be validated.

Use the resolved path for every `omc` call below, and on Windows run it through the PowerShell tool, not Bash/git-bash. Done when `omc --version` prints a version string.

## 2. Locate the Modelica Standard Library

Run `loadModel(Modelica); getVersion(Modelica); getErrorString(); getModelicaPath(); getErrorString();` through `omc` as a single script. `getVersion(Modelica)` returning a version string means MSL is installed; on failure, read the first `getErrorString()` for the reason and tell the user rather than assuming the model can reference `Modelica.*` packages. `getModelicaPath()` returns the directories `omc` searches for libraries — the one containing a `Modelica <version>` folder is where MSL actually lives on disk; keep that path, it's needed in [step 4](#4-write-the-model). Done when you know whether MSL is available and, if so, its version and install path.

## 3. Resolve the model

Pin down what to write before touching a file:

- The target `.mo` file path and the fully-qualified model name — reuse the existing file/model when the user says "modify" or "update", otherwise create both.
- Whether the target file already exists on disk — if the user said "create" but a file already sits at that path, confirm with them before overwriting it rather than assuming their phrasing is right. If it already defines other models and the user's request doesn't make clear which one to touch, ask.
- The model's components, parameters, variables, and equations (or `connect` statements, for a composed model) from the user's request.
- A result-file prefix for the simulation, no extension — `omc` appends `_res.<format>` itself (e.g. prefix `Foo` → `Foo_res.csv`).
- A stop time and tolerance for the simulation, if the user gave them — otherwise omit these arguments later and let `omc` use its own defaults.

## 4. Write the model

Create or edit the `.mo` file with the resolved model. Match the style, units, and naming already used in the file or in sibling models in the project rather than inventing new conventions. Don't reinvent the wheel: browse the MSL install path from [step 2](#2-locate-the-modelica-standard-library) for components and connectors that fit, and reuse them instead of writing custom equations for something MSL already models.

## 5. Build the checkpoints script

Copy [`references/checkpoints.mos`](references/checkpoints.mos) to the scratchpad and fill in its placeholders from [step 3](#3-resolve-the-model). Delete the `{{LOAD_MSL}}` line if the model has no Modelica Standard Library dependency ([step 2](#2-locate-the-modelica-standard-library)). If no stop time and/or tolerance were given, delete the corresponding `stopTime=`/`tolerance=` named argument from the `simulate()` call entirely rather than leaving the placeholder unfilled. Use `fileNamePrefix`, not `resultFile` — `simulate()` has no `resultFile` parameter and fails outright if one is passed. Leave the check and simulate blocks — those always run.

## 6. Run the check checkpoint

Run `omc checkpoints.mos` with the scratchpad as the current working directory — `omc` writes its generated files wherever it's invoked from, not necessarily next to the script. Then read the sections in order. `===LOAD===` first: any error there means the file didn't load (bad path, syntax error) and the `===CHECK===` section that follows isn't meaningful; fix the load problem and rerun from [step 5](#5-build-the-checkpoints-script). Once load is clean, read `===CHECK===`. Pass means `checkModel` reports 0 errors — warnings don't fail the checkpoint, but surface them in the report. On failure, fix the model ([step 4](#4-write-the-model)) using the error text and rerun from [step 5](#5-build-the-checkpoints-script); don't run the simulate checkpoint on a model that hasn't passed check.

## 7. Run the simulate checkpoint

Read the `===SIMULATE===` section from the same run. Pass means the `simulate()` messages contain no `Error` and `<prefix>_res.csv` (the result-file prefix from [step 3](#3-resolve-the-model), with `_res.csv` appended by `omc`) now exists on disk. On failure, fix the model and rerun from [step 5](#5-build-the-checkpoints-script).

## 8. Report

Always produce a summary that includes:

- The target `.mo` file path and whether it was created or modified.
- The `omc` version used ([step 1](#1-locate-omc)).
- The MSL version it was validated against ([step 2](#2-locate-the-modelica-standard-library)).
- How many fix/rerun cycles it took (if any).
- Pass/fail for whichever steps ran, with the raw error text for anything that ultimately failed.
- If simulation passed, a substantive line about the result — e.g. the final value of a key variable, or how a state changed over the simulated time span — not just pass/fail. Read `<prefix>_res.csv` for this before it's cleaned up ([step 9](#9-clean-up-build-artifacts)).

## 9. Clean up build artifacts

Whether or not the checkpoints ultimately passed, delete the compiler and simulation artifacts `omc` generated alongside `checkpoints.mos` in the scratchpad — the executable, `.c`/`.o`/`.json`/`.xml` files, and the result file. The `.mo` source file ([step 4](#4-write-the-model)) lives elsewhere and is unaffected.
