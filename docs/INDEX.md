# Driver Documentation Index

This directory documents optional Wicofe device-driver implementations and their packaging/integration.

## Scope
- Individual device drivers and hardware support matrices.
- DKM module packaging metadata and build instructions.
- Hardware identifiers, resources, interrupts, DMA, and lifecycle behavior.
- Driver validation, compatibility notes, and test fixtures.

## Ownership
Kernel device model, resource arbitration, driver lifecycle, and DKM loader are owned by [Wicofe-Kernel](https://github.com/Waizusora/Wicofe-Kernel). This repository owns driver implementations, not the kernel's core driver framework.

## Required driver record
Each driver must state hardware IDs, supported architectures, required kernel/DKM ABI, initialization/teardown behavior, resource ownership, failure modes, test evidence, firmware requirements, and license provenance. Do not claim hardware support without reproducible validation.

## Technical design
- [Driver subsystem implementation and integration design 0.1](DRIVER_SUBSYSTEM_DESIGN_0.1.md)

- [Central documentation index](https://github.com/Waizusora/Wicofe-Doc/blob/main/INDEX.md)
- [Repository documentation map](https://github.com/Waizusora/Wicofe-Doc/blob/main/REPOSITORY_DOCUMENTATION_MAP.md)
