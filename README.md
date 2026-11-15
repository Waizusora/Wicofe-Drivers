# Wicofe-Drivers

Driver implementations, packaging, and integration material for Wicofe OS.

## Scope

- Optional device-driver implementations intended for the Wicofe kernel.
- DKM module packaging and build metadata.
- Hardware support matrices and driver-specific documentation.
- Driver validation, compatibility notes, and test fixtures.

## Boundaries

The core device model, driver lifecycle, resource ownership, and DKM loader belong to Wicofe-Kernel. The canonical module ABI must be documented and versioned before third-party modules are treated as compatible.

## Requirements

Each driver must state supported hardware IDs, architecture, required kernel/module ABI, initialization and teardown behavior, resource ownership, failure handling, and license provenance. Never claim hardware support without reproducible testing.

## Status

Initial repository baseline. DKM details remain subject to the kernel's module ABI design.

## License

Original project code targets MPL-2.0. Firmware blobs, vendor code, and imported drivers require separate provenance and license review.
