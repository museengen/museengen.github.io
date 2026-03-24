# Bundle 001 — Locked MCU Parameter Rejection

## Scenario

MA_Cloud proposes to lower `mcu_fall_trigger_threshold` from 0.75 → 0.70.

This parameter has `max_step_fraction_per_update: 0.0` in the parameter
registry, which disables all remote mutation. The PSI must reject at
**State 1 (Allowlist)** with reason code `PARAMETER_LOCKED`.

## Why this is the first bundle

This is the simplest, most unambiguous rejection in the entire PSI
state machine. It requires no shadow window, no epistemic gate, no
two-key signals — just the allowlist check. If this bundle fails, the
most fundamental PSI gate is broken.

Establishing this bundle first proves the replay runner infrastructure
is working before adding more complex scenarios.

## Safety gate tested

**State 1 — Allowlist gate: `max_step_fraction_per_update == 0.0` lock**

MCU safety parameters are locked against remote mutation by default.
The only way to change them is via explicit user unlock + clinical
review workflow (for `mcu_hypoxia_hard_threshold_spo2`) or physical
access. This lock must be enforced before any other gate is evaluated —
high-quality evidence, valid two-key signals, and stable allostatic
baseline must not be able to unlock a locked parameter.

## Expected outcome

```
decision:         REJECTED
rejection_stage:  ALLOWLIST_FAIL
reason_codes:     [..., "PARAMETER_LOCKED"]
```

## What a failure of this bundle means

If this bundle fails, one of the following has happened:

1. `max_step_fraction_per_update == 0.0` is no longer being checked,
   or is being checked after other gates that might short-circuit it.
2. The allowlist check has been moved or removed.
3. The parameter registry YAML is not being loaded correctly.

Any of these represents a **critical safety regression**. The MCU
safety path must not be alterable via remote update.

## Related parameters

Both MCU parameters are locked (`max_step_fraction: 0.0`):
- `mcu_fall_trigger_threshold` (this bundle)
- `mcu_hypoxia_hard_threshold_spo2`

A corresponding bundle for `mcu_hypoxia_hard_threshold_spo2` should
be added as bundle 002.
