# Smart Inverter Toolbox for MATLAB/Simulink

A modular electromagnetic-transient (EMT) smart inverter model for MATLAB/Simulink. The toolbox provides configurable grid-support functions, real-time operating-mode selection, automatic controller tuning, automatic filter calculation, inverter capability enforcement, and dynamic switching between active- and reactive-power support functions.

The model is intended for distribution-network studies, controller benchmarking, heterogeneous smart-inverter studies, real-time simulation, and power-hardware-in-the-loop workflows.

## Main Features

- Three-phase grid-connected voltage-source inverter model.
- Synchronous-reference-frame control using measured PCC voltage and inverter current.
- Automatic calculation of controller gains and filter parameters.
- Constant power factor control.
- Constant reactive power control.
- Volt-VAr control with eight selectable curves.
- Volt-Watt and overvoltage Volt-Watt support.
- Active-power, reactive-power, and voltage-support priority options.
- Real-time switching between supported control modes.
- Apparent-power and current capability enforcement.
- Suitable for EMT simulation and real-time implementation.

## Software Requirements

- MATLAB
- Simulink
- Simscape Electrical / Specialized Power Systems, when required by the surrounding network model

Use a MATLAB release compatible with the supplied Simulink model and library files.

## Getting Started

### 1. Add the toolbox to the MATLAB path

Clone or download the repository, open MATLAB in the repository directory, and add the toolbox folders to the MATLAB path.

```matlab
projectRoot = pwd;
addpath(genpath(projectRoot));
savepath;
```

If the repository includes a startup script, run it before opening an example model.

```matlab
run('startup.m');
```

### 2. Open the Simulink model

Open the supplied example model or drag the Smart Inverter Toolbox block from the library into an existing Simulink distribution-network model.

```matlab
open_system('Smart_Inverter_Toolbox.slx');
```

Replace `Smart_Inverter_Toolbox.slx` with the actual model filename in the repository.

### 3. Enter the three required initial parameters

The user only needs to specify the following inverter and grid quantities in the toolbox mask.

| Parameter | Description | Unit |
|---|---|---:|
| `S_rated` | Rated apparent power of the smart inverter | VA |
| `V_LL_rms` | Rated phase-to-phase RMS grid voltage | V |
| `f_grid` | Grid operating frequency | Hz |

Example:

```matlab
S_rated = 50e3;   % 50 kVA
V_LL_rms = 400;   % 400 V line-to-line RMS
f_grid = 50;      % 50 Hz
```

After these values are entered, the toolbox automatically calculates the internal base quantities, filter values, controller parameters, and gains required by the inverter model. Manual tuning is not required for normal use.

## Toolbox Inputs

The toolbox receives the electrical measurements and power references required by the selected operating mode. Depending on the supplied model version, these signals may be provided through Simulink input ports, a bus object, or mask-linked workspace variables.

Typical signals include:

- PCC three-phase voltage.
- Inverter three-phase current.
- Available or commanded active power.
- Reactive-power or power-factor reference.
- Real-time mode-selection variables.

The PCC voltage and inverter current are transformed to the synchronous `dq` reference frame. Active and reactive power are calculated internally as

```text
P = (3/2)(v_d i_d + v_q i_q)
Q = (3/2)(v_q i_d - v_d i_q)
```

When the phase-locked loop aligns the `d` axis with the PCC voltage vector, the power commands are converted to current references as

```text
i_d* = (2/3) P* / v_d
i_q* = -(2/3) Q* / v_d
```

## Real-Time Configuration Variables

Five integer configuration variables can be changed while the simulation is running. These inputs allow the inverter operating mode, priority policy, reactive-power direction, Volt-Watt function, and Volt-VAr curve to be changed without rebuilding or restarting the model.

### `rpm` — Reactive Power Mode

| Value | Mode | Description |
|---:|---|---|
| `1` | CPF | Constant power factor operation |
| `2` | VV | Volt-VAr operation |
| `3` | CRP | Constant reactive power operation |

Examples:

```matlab
rpm = 1;   % Constant power factor
rpm = 2;   % Volt-VAr
rpm = 3;   % Constant reactive power
```

For `rpm = 2`, select the required Volt-VAr curve using `csel`. The `rps` variable is used only in CPF and CRP modes.

### `pps` — Power Priority Selection

| Value | Priority | Description |
|---:|---|---|
| `1` | APP | Active-power priority |
| `2` | RPP | Reactive-power priority |
| `3` | VS | Voltage-support priority |

```matlab
pps = 1;   % APP
pps = 2;   % RPP
pps = 3;   % VS
```

The selected priority determines how the inverter allocates its available apparent-power capability when simultaneous active- and reactive-power commands approach the inverter rating.

### `rps` — Reactive Power Selection

This input selects inductive or capacitive reactive-power operation in CPF and CRP modes.

| Value | Selection | Description |
|---:|---|---|
| `1` | IND | Inductive reactive-power operation |
| `-1` | CAP | Capacitive reactive-power operation |

```matlab
rps = 1;    % Inductive
rps = -1;   % Capacitive
```

`rps` is ignored when `rpm = 2` because the Volt-VAr curve determines the direction and magnitude of reactive-power support from the measured PCC voltage.

### `vwm` — Volt-Watt Support Mode

| Value | Mode | Description |
|---:|---|---|
| `0` | OFF | Volt-Watt support disabled |
| `1` | VW | Volt-Watt support enabled |
| `2` | OVW | Overvoltage Volt-Watt support enabled |

```matlab
vwm = 0;   % Disabled
vwm = 1;   % Volt-Watt
vwm = 2;   % Overvoltage Volt-Watt
```

The Volt-Watt function modifies the active-power command according to the measured PCC voltage and the selected Volt-Watt characteristic.

### `csel` — Volt-VAr Curve Selection

`csel` selects one of the eight implemented Volt-VAr curves.

| Value | Selected Curve |
|---:|---|
| `1` | Volt-VAr curve 1 |
| `2` | Volt-VAr curve 2 |
| `3` | Volt-VAr curve 3 |
| `4` | Volt-VAr curve 4 |
| `5` | Volt-VAr curve 5 |
| `6` | Volt-VAr curve 6 |
| `7` | Volt-VAr curve 7 |
| `8` | Volt-VAr curve 8 |

```matlab
rpm  = 2;   % Enable Volt-VAr mode
csel = 4;   % Select Volt-VAr curve 4
```

The selected curve defines the voltage breakpoints, deadband, slope, and reactive-power limits used by the Volt-VAr controller.

## Configuration Summary

| Variable | Valid Values | Function |
|---|---|---|
| `rpm` | `1`, `2`, `3` | CPF, Volt-VAr, or CRP selection |
| `pps` | `1`, `2`, `3` | APP, RPP, or VS priority |
| `rps` | `1`, `-1` | Inductive or capacitive operation in CPF/CRP |
| `vwm` | `0`, `1`, `2` | Volt-Watt disabled, VW, or OVW |
| `csel` | `1`–`8` | Volt-VAr curve selection |

## Recommended Simulink Implementation

The five configuration variables can be supplied using any Simulink signal source that produces integer-valued commands.

Suitable options include:

- Constant blocks for fixed studies.
- Dashboard switches or knobs for manual real-time testing.
- Step blocks for scheduled mode changes.
- Signal Editor or From Workspace blocks for scripted scenarios.
- Stateflow for supervisory control.
- OPAL-RT, dSPACE, or other real-time I/O channels for hardware experiments.

Use integer or enumerated signals where possible. If floating-point blocks are used, ensure the values are exactly equal to the supported selections.

### Example using Constant blocks

Create five Constant blocks and assign:

```matlab
rpm  = 2;
pps  = 3;
rps  = 1;
vwm  = 1;
csel = 4;
```

This configuration enables Volt-VAr operation, uses voltage-support priority, enables Volt-Watt support, and selects Volt-VAr curve 4. The `rps` value is present but is ignored while `rpm = 2`.

### Example of a mode change during simulation

A Step block can switch the reactive-power mode from CPF to Volt-VAr at `t = 5 s`.

```text
Initial value: 1
Final value:   2
Step time:     5 s
```

Similarly, a second Step block may enable Volt-Watt support during the simulation.

```text
Initial value: 0
Final value:   1
Step time:     5 s
```

## Typical Operating Configurations

### Constant power factor only

```matlab
rpm  = 1;
pps  = 1;
rps  = 1;
vwm  = 0;
csel = 1;   % Not used in CPF mode
```

### Constant reactive power with capacitive operation

```matlab
rpm  = 3;
pps  = 2;
rps  = -1;
vwm  = 0;
csel = 1;   % Not used in CRP mode
```

### Volt-VAr operation

```matlab
rpm  = 2;
pps  = 3;
rps  = 1;   % Ignored in VV mode
vwm  = 0;
csel = 5;
```

### Combined Volt-VAr and Volt-Watt support

```matlab
rpm  = 2;
pps  = 3;
rps  = 1;   % Ignored in VV mode
vwm  = 1;
csel = 5;
```

### Overvoltage Volt-Watt support with CPF operation

```matlab
rpm  = 1;
pps  = 1;
rps  = 1;
vwm  = 2;
csel = 1;   % Not used in CPF mode
```

## Inverter Capability Enforcement

The toolbox enforces the inverter apparent-power capability:

```text
(P*)^2 + (Q*)^2 <= S_rated^2
```

The available reactive-power capability at the current active-power operating point is

```text
Q_cap = sqrt(S_rated^2 - (P*)^2)
```

When the requested active and reactive powers exceed the inverter rating, the selected `pps` policy determines how the feasible command is allocated.

## Automatic Parameter Calculation

From `S_rated`, `V_LL_rms`, and `f_grid`, the initialization routine calculates the quantities required by the inverter model. These include, as applicable to the supplied model version:

- Voltage, current, impedance, and power base values.
- Angular grid frequency.
- Output-filter component values.
- Current-controller gains.
- Phase-locked-loop parameters.
- Power measurement and control-filter coefficients.
- Saturation and inverter capability limits.
- Internal normalization and per-unit conversion factors.

The calculated values are loaded automatically when the model initializes. Users should not overwrite these internal parameters unless they are intentionally modifying the toolbox design.

## Outputs and Logged Signals

Recommended signals to monitor or log include:

- PCC voltage magnitude.
- Three-phase PCC voltage.
- Three-phase inverter current.
- Active power `P`.
- Reactive power `Q`.
- Active-power reference `P*`.
- Reactive-power reference `Q*`.
- `d`- and `q`-axis current references.
- Selected reactive-power mode.
- Selected Volt-Watt mode.
- Active inverter priority mode.
- Current or apparent-power limiter status.

Use Simulink Data Inspector, Scope blocks, or signal logging to compare different modes and curves.

## Suggested Test Procedure

1. Enter the inverter rating, line-to-line RMS voltage, and grid frequency.
2. Start with `rpm = 1`, `vwm = 0`, and a suitable power-priority selection.
3. Verify steady-state active and reactive power.
4. Apply an overvoltage or undervoltage disturbance at the grid side.
5. Change `rpm`, `vwm`, or `csel` during the simulation.
6. Compare PCC voltage, active power, reactive power, and recovery behavior.
7. Confirm that the apparent-power limit is not violated.
8. Repeat the test for different feeder resistance-to-reactance ratios and inverter locations.

## Troubleshooting

### The model does not initialize

- Confirm that all toolbox folders are on the MATLAB path.
- Confirm that `S_rated`, `V_LL_rms`, and `f_grid` are positive numerical values.
- Run the initialization or startup script manually.
- Check that required Simulink and Simscape products are installed.

### The selected mode does not change

- Confirm that the real-time input uses one of the exact supported integer values.
- Check whether the input is connected to the correct toolbox port.
- Confirm that no upstream block is overriding the value.
- Use a Display block to verify the signal reaching the toolbox.

### Volt-VAr operation does not respond

- Set `rpm = 2`.
- Confirm that `csel` is between `1` and `8`.
- Verify that the PCC voltage crosses the selected curve breakpoints or deadband.
- Check whether the reactive-power command is being limited by the inverter rating.

### `rps` has no effect

`rps` is active only when `rpm = 1` or `rpm = 3`. It is intentionally ignored in Volt-VAr mode.

### Volt-Watt operation does not respond

- Confirm that `vwm` is set to `1` or `2`.
- Verify that the PCC voltage enters the corresponding Volt-Watt activation region.
- Check the active-power availability and lower active-power limit.
- Confirm that the selected `pps` policy permits active-power modification.

### Active or reactive power is lower than commanded

The inverter capability limiter may be active. Check whether

```text
sqrt((P*)^2 + (Q*)^2) > S_rated
```

and review the selected power-priority mode.

## Repository Structure

A recommended repository layout is

```text
.
├── README.md
├── LICENSE
├── CITATION.cff
├── models/
│   ├── Smart_Inverter_Toolbox.slx
│   └── Smart_Inverter_Library.slx
├── initialization/
│   ├── initialize_toolbox.m
│   └── calculate_parameters.m
├── examples/
│   ├── example_CPF.slx
│   ├── example_VoltVAr.slx
│   ├── example_VoltWatt.slx
│   └── example_mode_switching.slx
├── scripts/
│   └── run_example.m
├── data/
│   └── curve_parameters.mat
└── docs/
    └── figures/
```

Adjust the filenames in this README to match the final public repository.

## Citation

If this toolbox supports published research, cite the corresponding paper and software release. Add the final bibliographic information after publication.

```bibtex
@software{smart_inverter_toolbox,
  title  = {Smart Inverter Toolbox for EMT and Real-Time Studies},
  author = {Toolbox Authors},
  year   = {2026},
  url    = {https://github.com/YOUR-USERNAME/YOUR-REPOSITORY}
}
```

## License

Add the selected open-source license in the repository `LICENSE` file. Clearly state any restrictions associated with third-party Simulink blocks, external toolboxes, or hardware-interface components.

## Acknowledgment

The toolbox was developed for evaluating heterogeneous smart-inverter grid-support functions and adaptive voltage-support strategies in distribution networks. Its implemented control functions were benchmarked against an established smart-inverter EMT model and used in simulation and real-time PHIL studies.
