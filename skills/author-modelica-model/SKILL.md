---
name: author-modelica-model
description: Authors a Modelica model (.mo file) and validates it with OpenModelica (check + simulate).
disable-model-invocation: true
---

Writes a Modelica model (.mo file), then tests it with OpenModelica's `omc`. See [`../shared/checkpoint-loop.md`](../shared/checkpoint-loop.md) for the checkpoint loop this skill follows.

## 1. Locate omc

Read the `OPENMODELICAHOME` environment variable — OpenModelica sets it on install, and `omc` lives at `$OPENMODELICAHOME/bin/omc` (`omc.exe` on Windows). If it's unset, stop here and tell the user OpenModelica isn't set up rather than guessing at an install path — don't spend effort writing a model that can't be validated.

Use the resolved path for every `omc` call below, and on Windows run it through the PowerShell tool, not Bash/git-bash. Done when `omc --version` prints a version string.

## 2. Locate the Modelica Standard Library

Run `loadModel(Modelica); getVersion(Modelica); getErrorString(); getModelicaPath(); getErrorString();` through `omc` as a single script. `getVersion(Modelica)` returning a version string means MSL is installed; on failure, read the first `getErrorString()` for the reason and tell the user rather than assuming the model can reference `Modelica.*` packages. `getModelicaPath()` returns the directories `omc` searches for libraries — the one containing a `Modelica <version>` folder is where MSL actually lives on disk; keep that path, it's needed in [step 4](#4-write-the-model). Done when you know whether MSL is available and, if so, its version and install path.

## 3. Resolve the model

Work through the shared checklist in [`../shared/resolve-modelica-model.md`](../shared/resolve-modelica-model.md), plus two things specific to this `omc`-CLI backend:

- The target `.mo` file path — reuse the existing file when the user says "modify" or "update", otherwise create it.
- A result-file prefix for the simulation, no extension — `omc` appends `_res.<format>` itself (e.g. prefix `Foo` → `Foo_res.csv`).

## 4. Write the model

Create or edit the `.mo` file with the resolved model, following [`../shared/model-authoring-style.md`](../shared/model-authoring-style.md) — browse the MSL install path from [step 2](#2-locate-the-modelica-standard-library) for components and connectors that fit.

## 5. Build the checkpoints script

Copy [`references/checkpoints.mos`](references/checkpoints.mos) to the scratchpad and fill in its placeholders from [step 3](#3-resolve-the-model). Delete the `{{LOAD_MSL}}` line if the model has no Modelica Standard Library dependency ([step 2](#2-locate-the-modelica-standard-library)). If no stop time and/or tolerance were given, delete the corresponding `stopTime=`/`tolerance=` named argument from the `simulate()` call entirely rather than leaving the placeholder unfilled. Use `fileNamePrefix`, not `resultFile` — `simulate()` has no `resultFile` parameter and fails outright if one is passed. Leave the check and simulate blocks — those always run.

## 6. Run the check checkpoint

Run `omc checkpoints.mos` with the scratchpad as the current working directory — `omc` writes its generated files wherever it's invoked from, not necessarily next to the script. Then read the sections in order. `===LOAD===` first: any error there means the file didn't load (bad path, syntax error) and the `===CHECK===` section that follows isn't meaningful; fix the load problem and rerun from [step 5](#5-build-the-checkpoints-script). Once load is clean, read `===CHECK===` against the pass criteria in [`../shared/checkpoint-loop.md`](../shared/checkpoint-loop.md). On failure, fix the model ([step 4](#4-write-the-model)) using the error text and rerun from [step 5](#5-build-the-checkpoints-script).

## 7. Run the simulate checkpoint

Read the `===SIMULATE===` section from the same run. Pass means the `simulate()` messages contain no `Error` and `<prefix>_res.csv` (the result-file prefix from [step 3](#3-resolve-the-model), with `_res.csv` appended by `omc`) now exists on disk. On failure, fix the model and rerun from [step 5](#5-build-the-checkpoints-script).

## 8. Report

Work through the shared checklist in [`../shared/report-checklist.md`](../shared/report-checklist.md), plus two things specific to this `omc`-CLI backend:

- The target `.mo` file path.
- The `omc` version used ([step 1](#1-locate-omc)) and the MSL version it was validated against ([step 2](#2-locate-the-modelica-standard-library)).

For the shared checklist's substantive result line, read `<prefix>_res.csv` before it's cleaned up ([step 9](#9-clean-up-build-artifacts)).

## 9. Clean up build artifacts

Whether or not the checkpoints ultimately passed, delete the compiler and simulation artifacts `omc` generated alongside `checkpoints.mos` in the scratchpad — the executable, `.c`/`.o`/`.json`/`.xml` files, and the result file. The `.mo` source file ([step 4](#4-write-the-model)) lives elsewhere and is unaffected.
