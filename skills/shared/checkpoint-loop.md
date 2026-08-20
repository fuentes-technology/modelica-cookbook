# Checkpoint loop

Shared by every `author-modelica-model*` skill. The model has to load before it can pass the two checkpoints that matter:

- **check**: parses and verifies the model is balanced
- **simulate**: runs it end to end

A model isn't done until it loads cleanly and passes both checkpoints. If any step fails, fix the problem and rerun. Never hand back code that hasn't been checked. Stop after three fix/rerun cycles total without a full pass, and report the remaining error to the user instead of looping indefinitely.

**check** passes on 0 errors — warnings don't fail it, but surface them in the report. Never run the simulate checkpoint on a model that hasn't passed check.
