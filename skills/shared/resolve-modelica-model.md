# Resolve the model

Shared by every `author-modelica-model*` skill. Pin down what to write before touching anything:

- The fully-qualified model name — reuse the existing model when the user says "modify" or "update", otherwise create a new one.
- Whether the target already exists — if the user said "create" but it already exists, confirm with them before overwriting it rather than assuming their phrasing is right. If it already defines other models and the user's request doesn't make clear which one to touch, ask.
- The model's components, parameters, variables, and equations (or `connect` statements, for a composed model) from the user's request.
- A stop time and tolerance for the simulation, if the user gave them — otherwise let the validation backend use its own defaults.

The calling skill adds any backend-specific target details (a file path, a result-file prefix, ...) alongside this checklist rather than here, since those vary per backend.
