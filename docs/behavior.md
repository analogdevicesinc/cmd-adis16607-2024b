# Behavior

The model reproduces the ADIS16607 inertial signal path from transduction to digital output, for both the gyroscope and accelerometer channels. Per-run stochastic effects (bias, scale-factor tolerance, misalignment, over-temperature spread) are drawn once at simulation start and held constant for the run — each can be set to a specific deterministic value or sampled randomly. Time-evolving noise (ARW, VRW, bias instability) is always active and cannot be disabled.

For the full signal-processing chain with equations, block diagrams, and per-stage implementation details, see the [Behavior Reference](behavior-reference.md).

---

## Transduction and Gain

- Scale-factor error and scale-factor drift over temperature.
- Static nonlinearity (input-dependent gain distortion).
- Residual factory-calibration error (bias, SF, alignment).

## Cross-Axis Coupling

- 3×3 misalignment matrix per sensor type.
- Gyroscope linear-acceleration sensitivity (g-sensitivity matrix coupling acceleration into the rate channel).

## Vibration Effects

- Gyroscope vibration rectification error (broadband vibration → DC bias shift).
- Accelerometer vibration rectification error (RMS-driven envelope detector).

## Noise

- Angle random walk / velocity random walk (white noise, time-evolving).
- Bias instability (1/f process via Gauss–Markov bank, time-evolving).

## Bias and Offset

- Static bias offset (per-run draw).
- Residual bias drift over temperature.

## Analog Bandwidth

- Single real-pole frequency response per sensor type.

## Digital Signal Chain

- 3-stage cascade: fixed 2-sample moving average → configurable bi-quad IIR → D-sample averaging decimator.
- Output data rate set by decimation configuration.
- Group delay and filter settling transient after reconfiguration.
- Internal clock accuracy anchoring all sample rates and filter edges.

## ADC and Output Encoding

- Round-to-nearest quantization with saturation at full-scale.
- Selectable resolution mode (16-bit / 24-bit word width).
- Two's-complement output format.

---

## Related Pages

- [Behavior Reference](behavior-reference.md) — detailed signal-processing chain for each sensor channel
- [Interfaces](interfaces.md) — operating envelope, signals, configuration parameters
- [Fidelity](fidelity.md) — how closely the model tracks the datasheet
- [Limitations](limitations.md) — what is excluded and what assumptions apply

---

Copyright © 2026 Analog Devices, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](../LICENSE) for the full text.
