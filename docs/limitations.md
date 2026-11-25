# Limitations

This page documents what the model does not cover, the simplifying assumptions it relies on, and the boundaries of its intended use.

---

## Excluded Functionality

The model is an inertial signal-path simulation. It does not replicate the full device feature set.

| Category | What is not modeled |
|---|---|
| Communication | SPI bus-protocol transaction semantics, CRC framing, register-access timing, burst mode |
| Power management | Power-state sequencing, startup transient, low-power / sleep modes |
| Self-test & diagnostics | BIST, alarm thresholds, error flags, sensor fault injection |
| Calibration | User calibration register workflows, factory-cal restore, precision-null |
| Data-ready timing | Interrupt / DR pin, SYNC pulse timing, multi-device synchronization |
| Delta-angle / delta-velocity | On-chip coning and sculling integrators |
| Temperature sensor output | Die temperature is a model *input*, not an output channel |
| Shock events | Mechanical shock pulses beyond the vibration band |
| Long-term aging | Bias / SF stability beyond the operating session (>1000 hours) |
| Environmental sensitivities | Humidity, barometric pressure, magnetic field effects |

---

## Omitted Physical Effects

The following phenomena exist in the real device but are not represented in the model:

- Pipeline processing delay (0.064 ms)
- Rate random walk (+½ Allan slope, gyro) — dominant beyond τ ≈ 300 s
- Acceleration random walk (+½ Allan slope, accel) — dominant beyond τ ≈ 300 s
- Rate ramp (+1 Allan slope) — dominant beyond τ ≈ 300 s
- Thermal hysteresis (path-dependent bias shift with temperature cycling)
- Self-heating (~1.5 °C die rise from power dissipation)
- Turn-on bias drift
- Anisoinertia / angular-acceleration sensitivity
- Inter-sensor frame alignment (gyro-to-accel rotation)
- Saturation-recovery settling
- Cross-axis dynamic coupling — in the real device, excitation on one axis produces complex transient and steady-state responses on other axes; the model applies only a static misalignment matrix

---

## Modeling Simplifications

These effects *are* modeled, but with known approximations relative to the full physics:

| Mechanism | Full physics | What the model does |
|---|---|---|
| Verified input range | Hardware full-scale: ±40 g accel, ±480 °/s gyro | Model is verified over the narrower range ±150 m/s² (≈ ±15.3 g) and ±5 rad/s (≈ ±286 °/s); inputs beyond this range halt the simulation. See [Interfaces § Operating Envelope](interfaces.md#operating-envelope). |
| Analog bandwidth (gyro & accel) | Higher-order MEMS frequency response with multiple poles and phase structure | Single real pole at the published −3 dB frequency (gyro 1211 Hz, accel 1075 Hz) |
| Internal clock | Clock frequency tied to on-chip oscillator with thermal drift and aging | User-settable scalar parameter (Simulink limitation — no on-chip oscillator to derive from) |
| Crosstalk / inter-axis coupling | Complex mechanical, electrical, and readout-chain interactions produce axis-to-axis transfer functions | Static orthogonality-error matrix only; no dynamic cross-axis response |

---

## Assumptions

| Assumption | Implication |
|---|---|
| Spatially uniform die temperature | A single scalar temperature input drives all axes and both sensors; no thermal gradient across the package |
| No per-axis corner control on the top-level mask | In `±3σ multiplier` mode each of the 10 corner knobs `m` is scalar and broadcasts identically to X/Y/Z. Random-mode draws remain per-axis independent. |
| Quasi-static (swept) temperature | Temperature is a slowly-varying envelope; cycling and hysteresis effects are absent |
| Configuration set once per run | `IIR filter bandwidth code FILT_BW`, `Decimation rate DEC_RATE`, `Output resolution / word size [bits]`, and `Gyro resonator frequency f₀ [Hz]` are fixed at initialization; no mid-run register changes |
| Vibration within 55–370 Hz only | VRE is characterized for vibration content in this band; vibration energy outside this range is not captured by the model |
| G-sensitivity is isotropic | A single scalar coefficient is applied per axis-pair; no full 3×3 sensitivity matrix |
| Misalignment is symmetric and aggregate | One population σ drives all off-diagonal terms; no per-axis-pair differentiation |
| Allan variance modeled to τ ≤ 300 s | Noise regimes dominant beyond 300 s (+½ random walk, +1 rate ramp) are omitted |
| Output at ODR only | One output vector per ODR tick; no sub-sample timing or intra-sample transport delay |

---

## Related Pages

- [Behavior](behavior.md) — what the model reproduces
- [Fidelity](fidelity.md) — accuracy targets and acceptance criteria
- [Interfaces](interfaces.md) — operating envelope, signals, configuration

---

Copyright © 2026 Analog Devices, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](../LICENSE) for the full text.
