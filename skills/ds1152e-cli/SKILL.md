---
name: ds1152e-cli
description: Operate, test, debug, or document the RIGOL DS1152E oscilloscope through the public rigol CLI, including catalog get/set/action, acquisition, measurements, waveform export, triggers, MATH, FFT, and connected DG1022 validation. Use for DS1152E hardware work, rigol command changes, scope capture analysis, cross-instrument tests, or deployment; distinguish DS1000D-only commands and exclude firmware updates.
---

# DS1152E CLI

## Establish Context

Resolve the repository as the directory two levels above this Skill's real path. Read `README.md` before operating the scope and `docs/validation.md` before making coverage claims. For command semantics, inspect `rigol commands show NAME`; consult `docs/official/DS1000E_DS1000D_ProgrammingGuide_EN.pdf` when needed.

The validated scope is serial `DS1ET192605874`. Prefer its identity response over USB labels and use the serial selector because `/dev/usbtmcN` is dynamic. Reject the eight DS1000D-only logic-analyzer commands on this DS1152E.

## Use The Public CLI

Use `rigol`, preferring the repository's `.venv/bin/rigol` when present. Start connected work with:

```bash
rigol list
rigol --serial DS1ET192605874 info
rigol --serial DS1ET192605874 config
```

Use `measure` and `waveform` for high-level operations. Use catalog-backed `get`, `set`, and `action` for configuration and trigger/MATH work. Use `raw` only when no typed catalog or high-level workflow exists and after checking the official command catalog.

Do not call transport classes, open USBTMC directly, or send SCPI through another library when a CLI command exists. Do not implement or invoke firmware update, bootloader, factory-load, or reflash behavior. Require explicit user confirmation for destructive reset/factory actions already guarded by the CLI.

## Preserve Scope State And Data Integrity

Capture `config` before broad connected tests and restore settings changed by the task. Use `waveform --stop` when a stable stopped acquisition is required; rely on the CLI to resume only if the scope had been running.

Use allowed trigger-mode combinations from the typed catalog. Do not invent a shared trigger parameter across EDGE, PULSE, SLOPE, VIDEO, PATTERN, DURATION, or ALTERNATION modes.

For cross-channel phase, acquire synchronized 8192-point MAXIMUM records. Independent 600-point NORMAL transfers are not reliably aligned. Treat MATH/FFT waveform exports as raw display bytes where the programming interface lacks vertical calibration or FFT frequency-axis metadata.

When driving the scope from DG1022, verify both instrument identities and wiring before enabling output. Keep signals within the configured probe factor, vertical scale, coupling, offset, and input limits. Ensure the generator cleanup disables outputs even when scope acquisition fails.

## Observe Across Time

A scope query or waveform export is one acquisition, not continuous visual observation. Never infer stability, absence of drift, phase lock, trigger reliability, or absence of intermittent faults from one frame.

For time-dependent claims:

- Take at least five independent acquisitions, and increase the count and total duration to cover the suspected failure timescale.
- Use varied, jittered intervals rather than only evenly spaced waits, so periodic drift is not repeatedly sampled at the same phase.
- Record timestamps and keep trigger, timebase, sample rate, memory depth, channel scale, coupling, and probe factor fixed or record every change.
- Re-arm acquisition between observations. For cross-channel timing, stop once per observation and export both channels from that same 8192-point MAXIMUM acquisition.
- Inspect the spread and sequence of results, not only their mean. Retain outliers and raw waveforms needed to reproduce them.

If any observation disagrees with the expected behavior, do not discard or average it away. Reproduce it and check instrument state, trigger source/level/mode, acquisition mode, sample-rate interpretation, channel alignment, clipping, noise, cabling, source settings, and command readback. Resolve the cause before reporting a pass; if source behavior and measurement artifact cannot be separated, report the result as unresolved.

## Verify Changes And Claims

Run `python -m pytest` after code changes. Use the connected acceptance groups in the adjacent DG1022 repository for acquisition, channel, trigger, MATH, FFT, and robustness coverage.

Distinguish catalog availability, register readback, waveform observation, and physical trigger proof. Do not claim Roll mode activation, calibrated waveform accuracy, physical VIDEO/PATTERN/DURATION triggering, or complete GUI parity beyond documented evidence.
