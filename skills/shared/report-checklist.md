# Report checklist

Shared by every `author-modelica-model*` skill. Always produce a summary that includes:

- What was authored, and whether it was newly created or modified.
- How many fix/rerun cycles it took (if any).
- Pass/fail for whichever checkpoints ran, with the raw error text for anything that ultimately failed.
- If simulation passed, a substantive line about the result — e.g. the final value of a key variable, or how a state changed over the simulated time span — not just bare pass/fail.

The calling skill adds its own bullets alongside these for what was touched and where, which tooling/version was used, and how it pulled the substantive result, since those vary per backend.
