# aubo-lathe-cell

Automation project: AUBO i10 robot + Tesla SCADA + dual-spindle CNC lathe with SCADA.

## Overview

This repository contains the engineering documentation and source code for a robotic cell that automates loading and unloading of a dual‑spindle CNC lathe using an AUBO i10 collaborative robot, Delta DVP PLC and a SCADA system.

The system supports parameterized setup of parts, two‑zone operation, automated doors and vises control, and integration with the CNC controller via Modbus TCP and custom M‑codes.

## System architecture

Main components:

- AUBO i10 collaborative robot (handling and loading/unloading).
- Dual‑spindle CNC lathe with JS‑D680S‑2 CNC controller.
- Delta DVP12SE PLC for I/O handling, doors, vises and safety logic.
- SCADA system for parameter input, monitoring and diagnostics.
- Pneumatic system for doors and vises.
- SCHUNK grippers and custom mechanical interfaces.

High‑level functions:

- Automatic loading/unloading of raw and finished parts.
- Two‑zone machining with separate vises and doors.
- Parameterized job setup from SCADA (part type, offsets, cycle parameters).
- Alarm handling and safe recovery procedures.
- FAT/SAT testing and commissioning documentation.

## Repository structure

```text
docs/
  technical/        - Technical and functional descriptions of the cell
  manuals/          - Vendor manuals (PLC, CNC, robot, etc.)
  hardware/         - Gripper catalogs, M-code specs and related docs
  commissioning/
    FAT/            - Factory Acceptance Test protocols and reports
    SAT/            - Site Acceptance Test protocols and reports

src/
  plc/              - Delta DVP programs and projects
  robot/            - AUBO i10 scripts and projects
  scada/            - SCADA configuration, screens and tag databases
  cnc/              - CNC programs, macros and custom M-codes

config/
  modbus/           - Modbus TCP register maps
  parameters/       - Machine, robot and process parameters
  io_mapping/       - I/O allocation tables for PLC, CNC and robot

hardware/
  electrical/       - Electrical schematics of the cell and CNC
  mechanical/       - Mechanical drawings, 3D models, brackets, fingers
  pneumatic/        - Pneumatic schematics for doors and vises

tests/
  plc/              - PLC test programs and scenarios
  robot/            - Robot motion and sequence tests
  system/           - Integrated system tests, checklists and logs

scripts/
  - Helper scripts for deployment, data export or diagnostics
