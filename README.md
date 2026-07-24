# Smart Inverter Toolbox for MATLAB/Simulink

A modular electro-magnetic-transient (EMT) smart inverter toolbox for MATLAB/Simulink. The toolbox provides configurable grid-support functions, real-time operating-mode selection, automatic controller tuning, automatic filter calculation, and real-time dynamic switching between the support functions.

The model is intended for distribution-network studies, controller benchmarking, heterogeneous smart-inverter studies, real-time simulation, and power-hardware-in-the-loop workflows.

## Main Features

- Three-phase grid-connected voltage-source inverter model.
- Synchronous-reference-frame control using measured PCC voltage and inverter current.
- Automatic calculation of controller gains and filter parameters.
- Constant power factor control.
- Constant reactive power control.
- Volt-VAr control with eight selectable curves.
- Volt-Watt and Only Volt-Watt support.
- Active-power, reactive-power, and vector-scaling priority options.
- Real-time switching between supported control modes.
- Apparent-power and current capability constraints.
- Suitable for EMT simulation and real-time implementation.

## Software Requirements

- MATLAB 2024a and above
- Simulink
- Simscape Electrical / Specialized Power Systems

## Getting Started

### 1. Add the toolbox to the MATLAB path

Clone or download the repository, open MATLAB in the repository directory, and add the toolbox folders to the MATLAB path.

```matlab
projectRoot = pwd;
addpath(genpath(projectRoot));
savepath;
```

### 2. Open the Simulink model

Open the supplied example model or drag the Smart Inverter Toolbox block from the library into an existing Simulink distribution-network model.

```matlab
open_system('Smart_Inverter_Toolbox.slx');
```

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
| `3` | VS | Vector-scaling priority |

```matlab
pps = 1;   % APP
pps = 2;   % RPP
pps = 3;   % VS
```

The selected priority determines how the inverter allocates its available apparent-power capability when simultaneous active and reactive-power commands approach the inverter rating.

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
| `2` | OVW | Only Volt-Watt support enabled |

```matlab
vwm = 0;   % Disabled
vwm = 1;   % Volt-Watt
vwm = 2;   % Only Volt-Watt
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

## Suggested Test Procedure

1. Enter the inverter rating, line-to-line RMS voltage, and grid frequency.
2. Start with `rpm = 1`, `vwm = 0`, and a suitable power-priority selection.
3. Verify steady-state active and reactive power.
4. Apply an overvoltage or undervoltage disturbance at the grid side.
5. Change `rpm`, `vwm`, or `csel` during the simulation.
6. Compare PCC voltage, active power, reactive power, and recovery behavior.
7. Confirm that the apparent-power limit is not violated.
8. Repeat the test for different feeder resistance-to-reactance ratios and inverter locations.
