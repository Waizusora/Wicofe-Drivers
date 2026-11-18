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

## Documentation

See [docs/INDEX.md](docs/INDEX.md) for this repository's technical documentation. The [central repository documentation map](https://github.com/Waizusora/Wicofe-Docs/blob/main/REPOSITORY_DOCUMENTATION_MAP.md) identifies canonical ownership and cross-repository boundaries.

## Wicofe-wide principles

Wicofe is an independent Unix-like, POSIX-compatible operating system and software ecosystem, designed as an alternative to established and widely adopted software ecosystems and technologies, including GNU. Native components must use Wicofe interfaces and must not require Linux libc or GNU compatibility layers. Prefer permissive licenses; avoid GPL/AGPL, and consider LGPL only when no suitable permissive alternative exists. Reuse or fork upstream software when implementing it from scratch would be disproportionately difficult. See the [canonical project principles](https://github.com/Waizusora/Wicofe-Docs/blob/main/PROJECT_PRINCIPLES.md).
