---
title: EnergyPlus Doc1
author: andy
date: 2025-09-07 22:03:00 +/-0080
categories: [doc]
tags: [energyplus]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---



# Required at run time

 - Weather file (EPW): EPW_PATH (absolute path you pass to the CLI).
 - EnergyPlus executable: EPLUS_EXE (path to .../energyplus or just energyplus if on PATH).

# Coupling inputs (only when the plugin is enabled)

out/exchange/coupling_in.json — the only external knobs your OpenFOAM side sets each timestep:

T_supply_C (float): supply air temperature setpoint (°C).

m_dot_kg_s (float): supply mass flow rate (kg/s).
Example:

{ "T_supply_C": 16.0, "m_dot_kg_s": 0.30 }

# Model-internal parameters (set in the script)
## Geometry & zone

Room size: 5 m × 4 m × 3 m (volume used for ACH).

Surfaces: 4 walls, roof, floor (simple single-layer construction).

Global rules: vertex order/coordinate system (GlobalGeometryRules).

## Schedules (simple constants)

Always On (1.0).

SAT 13C (13 °C scheduled supply-air temperature).

MinOAFrac 0.2 (used by the OA controller).

## HVAC topology & components

OutdoorAirSystem with Controller:OutdoorAir (economizer off; min/max OA flow = 0 / Autosize).

DX cooling coil (single speed) with condenser air OutdoorAir:Node.

Electric heating coil.

Fan:SystemModel (autosized flow; basic power/per-flow setting).

Air terminal: AirTerminal:SingleDuct:Uncontrolled (single zone).

Zone Splitter → terminal inlet; Zone Mixer ← zone return.

## Setpoint Managers

Scheduled 13 °C on the supply outlet node.

MixedAir manager to make the coils “see” that SAT before the fan.

## Sizing (autosize drivers)

Sizing:Zone: supply design temps (13 °C cooling, 40 °C heating), humidity ratios (0.008).

Sizing:System: central SATs, plus required preheat/precool temperatures & humidity ratios.

## Site & simulation control

Site:Location (generic numbers in the script; actual weather comes from EPW).

RunPeriod (Jan 1–2), Timestep (6 per hour), Version.

## Building (solar distribution etc.).

Diagnostics (optional: display warnings/extras if you included that block).

## Curves

Flat performance curves for DX coil capacity/EIR and PLF (all coefficients = 0 except constant = 1).
