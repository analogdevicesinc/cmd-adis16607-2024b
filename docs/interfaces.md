# Interfaces

This page defines the model's operating envelope, input and output signals, and configuration parameters.

---

## Operating Envelope

| Quantity | Model input range | Hardware full-scale |
|---|---|---|
| Acceleration (per axis) | ±150 m/s² (≈ ±15.3 g) | ±40 g |
| Angular rate (per axis) | ±5 rad/s (≈ ±286 deg/s) | ±480 deg/s |
| Temperature | −40 °C to +85 °C | −40 °C to +85 °C |

> [!IMPORTANT]
> The model's verified input range is narrower than the hardware full-scale. The hardware supports ±40 g and ±480 °/s; the model is scoped to the range over which it was verified (±15.3 g, ±286 °/s). Inputs outside the model input range raise an error and halt the simulation, even when the value would still be within the hardware full-scale.

---

## Inputs

| Signal | Units | Description |
|---|---|---|
| Acceleration (3-axis) | m/s² | True specific-force applied to the sensor |
| Angular rate (3-axis) | rad/s | True angular rate applied to the sensor |
| Temperature | °C | Die temperature driving all thermal dependencies |

The acceleration input is specific force, not kinematic acceleration — gravity is not added by the model. A stationary sensor must be driven with the reaction to gravity, not a zero vector.

---

## Outputs

| Signal | Units | Description |
|---|---|---|
| Acceleration (3-axis) | m/s² | Modeled accelerometer output including all error sources |
| Angular rate (3-axis) | deg/s | Modeled gyroscope output including all error sources |

Outputs are always in engineering units (double-precision values on the LSB grid). The `WordSize` parameter (16 or 24 bits) controls quantization resolution, not the output format.

> [!WARNING]
> **Angular-rate units are asymmetric: input is rad/s, output is deg/s.**
> Acceleration is m/s² in both directions, but the gyro input port takes
> **rad/s** while the gyro output port emits **deg/s** — matching the ADIS16607
> datasheet's native output units.

### Integrating the outputs

`gyroOut` and `accelOut` update at the ODR (`f_S / (DEC_RATE + 1)`; see [Signal-Chain Registers](#signal-chain-registers)), which is slower than the input rate. When you connect an output to a consumer that runs at a different (usually faster) sample time, insert a **Rate Transition** block on the output signal — otherwise Simulink will either insert one implicitly or raise a sample-time mismatch in strict configurations. The two output ports share the same sample time, so they can be combined (Bus Creator, Mux) without a rate transition between them.

---

## Configuration Parameters

### Signal-Chain Registers

These parameters mirror hardware registers documented in the ADIS16607 datasheet.

| Parameter | Mask name | Datasheet reference | Description |
|---|---|---|---|
| Gyro resonator frequency | `GyroResonatorFreqHz` | Gyroscope Sensor Resonant Frequency | MEMS resonant frequency f₀ (typ. 78 kHz). Sets the conversion rate f_S = f₀ / 8. Fixed in hardware; exposed in the model for part-to-part variation studies. |
| Decimation rate | `DecRate` | `DEC_RATE` register | Decimation/averaging factor. Output data rate ODR = f_S / (DEC_RATE + 1). Default 3 → D = 4 → ODR = f_S / 4. |
| Filter bandwidth | `FiltBw` | `MSC_CTRL[9:8]` | Bi-quad IIR low-pass filter bandwidth code (0–3). Cutoff frequencies scale linearly with f_S. |
| Output word size | `WordSize` | `USER_DATA_CFG` bit 15 (`WORD_SIZE`) | Output resolution: `16` or `24` bits. Affects LSB weight and quantization noise floor. |

#### `FiltBw` Code Table

| Code | −3 dB cutoff |
|---|---|
| 0 | disabled |
| 1 | 554.5 Hz |
| 2 | 102.3 Hz |
| 3 | 20.4 Hz |

> [!NOTE]
> Cutoffs are given for the typical gyroscope resonator frequency of 78 kHz.

---

### Draw Modes (Stochastic Configuration)

The model contains **10 independent per-run stochastic terms** (5 gyro + 5 accelerometer) that represent part-to-part manufacturing variation. Each term is drawn **once at simulation initialization** and held constant for the entire run — it represents one physical device realization.

Every term has two controls:

1. **Draw-mode selector** — chooses how the value is determined.
2. **Multiplier knob** (*m*) — active only in `±3σ multiplier` mode.

#### Available Draw Modes

| Mode | Mask value | Behavior |
|---|---|---|
| `random` | Default | A **truncated-normal random draw** clipped at ±3σ. Each simulation run produces a different device realization from the population distribution. Use for Monte Carlo studies. |
| `±3σ multiplier` | Deterministic | The drawn value is set to **m × 3σ**, where *m* ∈ [−1, +1] is the corresponding multiplier knob. Use for worst-case or corner analysis. |

Multiplier knob interpretation:

| *m* value | Resulting draw | Use case |
|---|---|---|
| +1 | +3σ (worst-case positive) | Upper spec-limit corner |
| −1 | −3σ (worst-case negative) | Lower spec-limit corner |
| 0 | Nominal (zero error / unity gain) | Ideal-device baseline |
| Any value in (−1, +1) | Proportional intermediate | Partial-corner sweeps |

> [!TIP]
> The multiplier knob broadcasts identically to all three axes (X/Y/Z) of its term. Per-axis corner control is not supported.

#### Gyro Draw-Mode Parameters

| Stochastic term | Draw-mode prompt | Multiplier prompt | What it draws |
|---|---|---|---|
| Static bias | Gyro static-bias draw mode | Gyro static-bias m (±3σ) | Fixed zero-rate offset applied for the entire run |
| Bias over-temperature drift | Gyro bias over-temp drift draw mode | Gyro bias over-temp drift m (±3σ) | Amplitude and offset of the bias-vs-temperature curve |
| Scale-factor tolerance | Gyro SF-tolerance draw mode | Gyro SF-tolerance m (±3σ) | Fixed scale-factor error applied for the entire run |
| Scale-factor over-temperature drift | Gyro SF over-temp drift draw mode | Gyro SF over-temp drift m (±3σ) | Amplitude of the scale-factor-vs-temperature curve |
| Misalignment | Gyro misalignment draw mode | Gyro misalignment m (±3σ) | Off-diagonal cross-axis coupling (orthogonality error) |

#### Accelerometer Draw-Mode Parameters

| Stochastic term | Draw-mode prompt | Multiplier prompt | What it draws |
|---|---|---|---|
| Static bias | Accel static-bias draw mode | Accel static-bias m (±3σ) | Fixed zero-g offset applied for the entire run |
| Bias over-temperature drift | Accel bias over-temp drift draw mode | Accel bias over-temp drift m (±3σ) | Amplitude and offset of the bias-vs-temperature curve |
| Scale-factor tolerance | Accel SF-tolerance draw mode | Accel SF-tolerance m (±3σ) | Fixed scale-factor error applied for the entire run |
| Scale-factor over-temperature drift | Accel SF over-temp drift draw mode | Accel SF over-temp drift m (±3σ) | Amplitude of the scale-factor-vs-temperature curve |
| Misalignment | Accel misalignment draw mode | Accel misalignment m (±3σ) | Off-diagonal cross-axis coupling (orthogonality error) |

#### Typical Usage Patterns

**Monte Carlo analysis** — Set all draw modes to `random`. Run *N* simulations; each produces a statistically independent device from the population.

**Worst-case corner analysis** — Set specific draw modes to `±3σ multiplier` with *m* = ±1 to examine the extremes of a single error source while holding others at nominal (m = 0) or random.

**Nominal/ideal device** — Set all draw modes to `±3σ multiplier` with all multipliers at 0. All per-run stochastic errors are zeroed; only dynamic noise sources (angle random walk (ARW), velocity random walk (VRW), and bias instability) remain active.

---

## Related Pages

- [Behavior](behavior.md) — signal-path description and stochastic structure
- [Fidelity](fidelity.md) — accuracy criteria and verification approach
- [Limitations](limitations.md) — what is excluded

---

Copyright © 2026 Analog Devices, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](../LICENSE) for the full text.
