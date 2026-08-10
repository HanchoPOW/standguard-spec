# Contributing

The catalog grows one way: **real failure modes**.

## Proposing a new check

Open an issue with:

1. **The failure mode** — what actually happened on a bench, in a campaign, or in review. Anonymize freely; keep the physics. "This could theoretically go wrong" is not grounds for a check.
2. **The config pattern** that should have caught it — a minimal JSON snippet that violates the proposed rule.
3. **The cost** — destroyed hardware, invalidated data, blocked campaign, or review churn. This determines severity (`error` vs `warning`).

## Severity bar

- **`error`** — the failure mode destroys hardware, invalidates data, or blocks a campaign. Verdict must be exit 1.
- **`warning`** — the failure mode is real but situational; a human should see it and decide.

## Spec changes

Changes to `spec/bench-config.schema.json` must be backward-compatible (new optional fields only) or ship as a new major version of the spec. Tooling in the wild gates real hardware on this document — no silent breakage.

## Style

Checks are named `category.rule`, lowercase, dot-separated. Descriptions are written for the engineer holding the multimeter at 11 pm, not for the marketing page.
