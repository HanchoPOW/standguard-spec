# StandGuard Bench Config Spec

**An open specification for describing hardware-in-the-loop test benches — and the 20 checks that catch the mistakes that destroy hardware before current flows.**

Every rule in this catalog comes from a failure mode that has actually burned a bench: a transposed pin, an amplitude one keystroke over the rail, an e-stop loop nobody wired, a sampling rate that silently aliased six weeks of campaign data.

The most expensive bench bugs are typed, not soldered. This spec exists so they get caught by a linter instead of a purchase order.

## What this repo is

- **`spec/bench-config.schema.json`** — the open JSON Schema for describing a bench: channels, pins, rails, interlocks, sequences.
- **`CHECKS.md`** — the full check catalog: 20 rules across 8 categories, each with severity, rationale, and the failure mode it prevents.
- **`examples/`** — a passing bench config and a failing one, so you can see exactly what the linter catches.

The reference linter (`standguard.py` — one auditable file, zero dependencies, runs fully offline) is in early access at **[standguard.dev](https://standguard.dev)**. Anything that can emit JSON conforming to this spec can be linted — write configs by hand or export them from your bench tooling.

## The verdict contract

```
$ standguard.py lint bench.json --strict
```

- **exit 0** — acceptable. Apply power.
- **exit 1** — do not apply power. Every finding carries severity, rule ID, and the exact config location — human-readable on the bench, JSON for your pipeline.

Wire it into CI, or into the bench interlock checklist itself.

## A bench config in one look

```json
{
  "bench": { "name": "rig-A-avionics", "campaign_window_s": 3600 },
  "channels": [
    { "name": "bus_power", "pin": "P1.3", "kind": "power",
      "voltage_v": 28.0, "max_voltage_v": 32.0, "max_current_a": 10.0,
      "interlock": "estop_loop_1" }
  ],
  "interlocks": [
    { "name": "estop_loop_1", "type": "hardware_estop", "cuts": ["bus_power"] }
  ],
  "sequences": [
    { "name": "bit_sequence", "steps": [
      { "action": "energize", "channel": "bus_power", "duration_s": 5 }
    ] }
  ]
}
```

## The eight categories

| Category | Question it answers |
|---|---|
| `schema` | Is this a bench config at all? |
| `unique` | Can every signal be named unambiguously? |
| `pin` | Does every physical pin have exactly one owner? |
| `limits` | Does every drive stay inside its channel's ratings? |
| `interlock` | Can every hazardous channel be cut — by something that exists? |
| `timing` | Will the data mean anything (Nyquist), and will it fit the window? |
| `stimulus` | Will any sequence step overdrive a channel? |
| `sequence` | Does every step point at something real? |

Full rules with failure-mode rationale: **[CHECKS.md](CHECKS.md)**.

## Contributing

New checks are welcome — with one hard rule: **a proposed check must cite a real failure mode.** "This could theoretically go wrong" is not enough; "this burned a bench / voided a campaign / halted a program" is. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE). The spec is free to implement in any tooling, commercial or otherwise.
