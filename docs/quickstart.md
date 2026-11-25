# Quickstart

Get the ADIS16607-2 Simulink model running in under five minutes. This page walks through installation, first simulation, and basic configuration.

---

## Prerequisites

| Requirement | Details |
|---|---|
| MATLAB | R2024b (tested) |
| Simulink | Required |
| Additional toolboxes | None |
| Platform | Windows 64-bit only |

> **Note:** This toolbox is built and tested for MATLAB R2024b only. For MATLAB R2025b, install the `ADIS16607_IMU` toolbox instead (available from [`analogdevicesinc/cmd-adis16607`](https://github.com/analogdevicesinc/cmd-adis16607/releases)); it contains the same model.

---

## Step 1 — Install the Toolbox

The toolbox is distributed through two equivalent channels. Choose whichever fits your workflow.

Option A — MATLAB Add-On Explorer:

1. In MATLAB, select Add-Ons > Explore Add-Ons.
2. Search for `ADIS16607`.
3. Select `ADIS16607_IMU_R2024b`, then select Add to install.

Option B — GitHub Releases:

1. On the [repository Releases page](https://github.com/analogdevicesinc/cmd-adis16607-2024b/releases), download the `ADIS16607_IMU_R2024b_<version>.mltbx` asset attached to the latest release.
2. Open the downloaded file in MATLAB to install.

Both channels deliver the same toolbox. Refer to [Install Add-Ons](https://nl.mathworks.com/help/matlab/matlab_env/get-add-ons.html) for the MathWorks reference on the install workflow.

---

## Step 2 — Open the Packaged Example

The toolbox ships one packaged example — an accelerometer mission profile (hover → climb → forward → brake → back → descend) overlaid across several independent simulated ADIS16607-2 device samples.

```matlab
ADIS16607.openExample()
```

This copies the example bundle into an `ADIS16607_02_Example` subfolder of your current working directory, `cd`s into it, adds its `helper/` and `models/` folders to the MATLAB path, and opens the script; edit the copy freely. Pass a destination path (`ADIS16607.openExample('C:\path\to\destination')`) to install it elsewhere.

Running the script produces a three-tile figure (X, Y, Z) of accelerometer output overlaid on the true commanded specific-force, plus a per-sample static-bias table.

---

## Step 3 — Run the Model Standalone

To use the model block in your own system:

1. Create a new Simulink model or open an existing one.
2. Open the Simulink Library Browser and locate the `ADI ADIS16607` library.
3. Drag the `ADIS16607-2 IMU` block into your model.
4. Connect inputs (all three are required; for a static, upright sensor at room temperature the recommended defaults are `trueAngularRate = [0;0;0]`, `trueAcceleration = [0;0;9.80665]`, `temperature = 25` — driving `trueAcceleration = [0;0;0]` means free-fall, not a stationary sensor):
   - **`trueAngularRate`** — 3×1 vector, units: rad/s
   - **`trueAcceleration`** — 3×1 vector, units: m/s²
   - **`temperature`** — scalar, units: °C
5. Connect outputs:
   - **`gyroOut`** — 3×1 vector, units: deg/s (quantized to 16- or 24-bit LSB grid)
   - **`accelOut`** — 3×1 vector, units: m/s² (quantized to 16- or 24-bit LSB grid)
6. Run the simulation.

> [!WARNING]
> **Angular-rate units are asymmetric: `trueAngularRate` is rad/s, `gyroOut` is deg/s.**
> Acceleration is m/s² in both directions, but the gyro input takes **rad/s**
> while the gyro output emits **deg/s** — matching the ADIS16607 datasheet's
> native output units.

### Solver and sample-time requirements

The block has no continuous states; the internal signal chain runs on a discrete `f_S = f₀ / 8` clock (default `9750 Hz`, ≈ `102.6 µs`). Any solver is supported — variable-step or fixed-step, continuous or discrete — because the input ports inherit sample time and the block rate-transitions internally. If you select a fixed-step solver, `1/f_S` (or a divisor of it) is the recommended step; otherwise use a variable-step solver and let Simulink handle the rate transition. Outputs update at the ODR (`f_S / (DEC_RATE + 1)`), not at solver rate — see [Behavior Reference § Digital Signal Chain](behavior-reference.md#digital-signal-chain) and [Interfaces](interfaces.md#signal-chain-registers).

---

## Step 4 — Configure the Block

Double-click the block to open its mask. Key parameters:

| Mask prompt | Default | What it controls |
|---|---|---|
| Decimation rate DEC_RATE | 3 | Decimation factor. ODR = f_S / (DEC_RATE + 1). Default gives D = 4. |
| IIR filter bandwidth code FILT_BW | 0 | IIR filter bandwidth code (0 = disabled, 1–3 = progressively narrower). |
| Output resolution / word size [bits] | 16 | Output word width: `16` or `24` bits. Affects quantization and output format. |
| Gyro resonator frequency f0 [Hz] | 78000 | MEMS resonator frequency (Hz). Sets conversion rate f_S = f₀ / 8. |

Draw-mode parameters control per-run stochastic behavior (bias, scale-factor tolerance, misalignment). See [Interfaces](interfaces.md) for the full parameter reference.

---

## Step 5 — Choose a Simulation Mode

### Nominal (Ideal Device)

Set all draw modes to `±3σ multiplier` and all multiplier knobs to `0`. Only dynamic noise sources (ARW, VRW, bias instability) remain active.

### Monte Carlo

Set all draw modes to `random`. Each simulation run produces a statistically independent device realization from the population distribution.

### Worst-Case Corner

Set specific draw modes to `±3σ multiplier` with multiplier `m = ±1` to examine the extremes of a single error source while holding others at nominal.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Block not found in Library Browser | Toolbox not installed or path not refreshed | Re-install the toolbox; run `rehash toolboxcache` |
| Simulation errors on start | Input signal dimensions incorrect | Ensure `trueAngularRate` and `trueAcceleration` are 3×1 vectors; `temperature` is scalar |
| Output is constant zero | All inputs are zero and draw modes are set to nominal | Apply a non-zero input or enable random draw modes |
| Input validation error | Input exceeds the supported operating envelope | Check that angular rate is within ±5 rad/s, acceleration within ±150 m/s², temperature within −40 to +85 °C |

---

## Related Pages

- [Interfaces](interfaces.md) — full signal and parameter reference
- [Behavior](behavior.md) — signal-path description and stochastic structure
- [Fidelity](fidelity.md) — accuracy targets and verification approach
- [Limitations](limitations.md) — what is excluded from the model

---

Copyright © 2026 Analog Devices, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](../LICENSE) for the full text.
