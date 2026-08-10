# The Check Catalog

20 rules, 8 categories. Severity is `error` (blocks the verdict — exit 1) or `warning` (advisory — reported, verdict unaffected unless `--strict`).

Each rule lists the failure mode it exists because of.

---

## 1 · schema — is this a bench config at all?

### `schema.channels` · error
Config must define a non-empty channel list.
*Failure mode: an empty or malformed export linting "clean" and gating a campaign on nothing.*

### `schema.name` · error
Every channel requires a unique name field.
*Failure mode: two channels silently sharing identity in downstream tooling.*

## 2 · unique — unambiguous signal identity

### `unique.name` · error
Duplicate channel names are rejected.
*Failure mode: a renamed channel colliding with an existing one; sequences now drive the wrong line.*

## 3 · pin — one physical pin, one owner

### `pin.conflict` · error
Two channels may not allocate the same physical pin.
*Failure mode: a stimulus shorted into a measurement line. Destroys hardware. This is the canonical bench-killer — two engineers, one spreadsheet, one pin.*

## 4 · limits — inside the ratings, always

### `limits.voltage` · error
A channel's drive voltage must stay under its own rating (`voltage_v ≤ max_voltage_v`).
*Failure mode: nominal drive configured above the rail the hardware can take.*

### `limits.positive` · error
Voltage and current limits must be positive numbers.
*Failure mode: a negative limit from a units slip (mA entered as A) disabling the very check meant to protect the channel.*

## 5 · interlock — hazardous means cuttable

### `interlock.missing` · error
Channels rated ≥ 24 V (power or drive) must reference an interlock loop.
*Failure mode: a 28 V bus with nothing that cuts it when a run goes wrong. Destroys hardware — and this is the one safety officers ask about first.*

### `interlock.missing/low` · warning
Lower-voltage drive channels without an interlock are flagged advisory.
*Failure mode: the "harmless" 12 V actuator line that wasn't.*

### `interlock.undefined` · error
An interlock reference must resolve to a defined loop.
*Failure mode: a channel pointing at `estop_loop_2` when only `estop_loop_1` exists — protection that looks present in the config and is absent on the bench.*

### `interlock.cuts` · error
An interlock's cut-list must reference defined channels.
*Failure mode: the e-stop cuts `bus_pwr` — a typo for `bus_power` — and therefore cuts nothing.*

## 6 · timing — data that means something

### `timing.nyquist` · error
Sample rate must be ≥ 2× the channel's fastest signal frequency.
*Failure mode: sampling a 400 Hz signal at 700 Hz. The data looks fine. It is aliased. Weeks of campaign — void. The worst failures invalidate data without touching hardware.*

### `timing.oversampling` · warning
Less than 5× oversampling is flagged for review.
*Failure mode: Nyquist-compliant data that still can't reconstruct the waveform anyone needed.*

### `timing.rate` · error
Sample rates must be positive.

### `timing.duration` · error
Step durations must be positive.

### `timing.budget` · error
Total sequence time must fit inside `campaign_window_s`.
*Failure mode: discovering the overrun when the bench times out at hour eleven of a twelve-hour window.*

## 7 · stimulus — never overdrive

### `stimulus.overdrive` · error
A step's `amplitude_v` must stay under the target channel's `max_voltage_v`.
*Failure mode: 14 V into a 12 V-rated channel. One keystroke over the rail. Destroys hardware.*

### `stimulus.passive` · warning
Stimulating an input-only channel is flagged advisory.
*Failure mode: a stimulus step aimed at a measurement channel — at best a no-op, at worst a fight between two drivers.*

## 8 · sequence — steps that point at real things

### `sequence.reference` · error
Every step must reference a defined channel.
*Failure mode: a step pointing at a channel renamed last revision. The bench halts mid-campaign — or worse, doesn't.*

### `sequence.empty` · warning
Sequences with no steps are flagged for review.
*Failure mode: the "ran fine" campaign that ran nothing.*

### `sequence.steps` · error
Step structure must validate against the schema (known action, required fields present).
*Failure mode: a misspelled action silently skipped by a lenient runner.*
