<p align="center">
	<img src="assets/ADI-Logo-RGB-FullColor.svg" alt="Analog Devices logo" width="280" />
</p>

# ADIS16607-2 Simulink Model

[Quickstart](docs/quickstart.md) | [Interfaces](docs/interfaces.md) | [Behavior](docs/behavior.md) | [Behavior Reference](docs/behavior-reference.md) | [Fidelity](docs/fidelity.md) | [Limitations](docs/limitations.md) | [CHANGELOG](CHANGELOG.md)

This toolbox delivers a medium-fidelity Simulink model of the ADIS16607-2 IMU for algorithm development, control design, and system-level simulation.

## What Is This Model?

The ADIS16607-2 Simulink model is a product-facing simulation component for users who need realistic IMU behavior in model-based workflows without waiting for full hardware campaigns.

It is intended for:

- Product-focused IMU model for simulation workflows
- Built for users who already work in Simulink
- Designed to represent practical sensor behavior for early and mid-stage development

## To Start Using The Model

1. Install the `ADIS16607_IMU_R2024b` toolbox using either of the two available channels:
   - MATLAB Add-On Explorer: in MATLAB, select Add-Ons > Explore Add-Ons, search for `ADIS16607`, and add `ADIS16607_IMU_R2024b`.
   - GitHub Releases: on the [repository Releases page](https://github.com/analogdevicesinc/cmd-adis16607-2024b/releases), download the `ADIS16607_IMU_R2024b_<version>.mltbx` asset attached to the latest release and open it in MATLAB.

   Refer to [Install Add-Ons](https://nl.mathworks.com/help/matlab/matlab_env/get-add-ons.html) for the MathWorks reference on the install workflow.
2. Install the shipped example — an accelerometer mission profile overlaid across several simulated ADIS16607-2 device samples — into your current working folder and open it:

   ```matlab
   ADIS16607.openExample()
   ```

3. Follow [Quickstart](docs/quickstart.md) for a complete first-run flow, including how to install the example bundle elsewhere.

## Toolbox Contents

- ADIS16607-2 model artifacts included in the toolbox
- One packaged example, launched via `ADIS16607.openExample(...)`
- User documentation under this release root

## Fidelity And Scope

The model targets medium fidelity and reproduces key datasheet-aligned behavior for architecture and algorithm evaluation.

It is not a replacement for hardware validation, production characterization, or certification evidence.

The model's verified input range is narrower than the hardware full-scale: ±150 m/s² (≈ ±15.3 g) and ±5 rad/s (≈ ±286 °/s), vs. hardware ±40 g and ±480 °/s. Inputs beyond the verified range halt the simulation — see [Interfaces § Operating Envelope](docs/interfaces.md#operating-envelope).

Detailed scope is documented in [Behavior](docs/behavior.md), [Fidelity](docs/fidelity.md), and [Limitations](docs/limitations.md).

## Environment

- Requires MATLAB R2024b and Simulink
- Platform: Windows 64-bit only
- Toolbox name: ADIS16607_IMU_R2024b
- Toolbox version: v1.0.0-ga

> **Note:** This toolbox is built and tested for MATLAB R2024b only. For MATLAB R2025b, install the `ADIS16607_IMU` toolbox instead (available from [`analogdevicesinc/cmd-adis16607`](https://github.com/analogdevicesinc/cmd-adis16607/releases)); it contains the same model.

## Documentation

- [Quickstart](docs/quickstart.md)
- [Interfaces](docs/interfaces.md)
- [Behavior](docs/behavior.md)
- [Behavior Reference](docs/behavior-reference.md) — detailed signal-processing chain, equations, and per-stage implementation detail
- [Fidelity](docs/fidelity.md)
- [Limitations](docs/limitations.md)
- [CHANGELOG](CHANGELOG.md)

## Support

Start with [Quickstart](docs/quickstart.md) and [Limitations](docs/limitations.md).

For issues, bug reports, or new model requests:

- Submit a GitHub Issue in this repository.
- Email simulationandmodeling@analog.com.

---

Copyright © 2026 Analog Devices, Inc.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for the full text.
