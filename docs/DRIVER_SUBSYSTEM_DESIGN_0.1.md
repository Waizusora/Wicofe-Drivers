# Wicofe Driver Implementation and Integration Design

- **Version:** 0.1
- **Status:** Draft architecture
- **Repository:** Wicofe-Drivers
- **Targets:** x86-64 and AArch64
- **License:** MPL-2.0 for original project code

## 1. Purpose and authority

This document defines how driver implementations in this repository are organized, reviewed, built, tested, and integrated. It does not define the kernel's device model or freeze the DKM ABI.

Canonical boundaries:
- **Wicofe-Kernel** owns device discovery interfaces, resource arbitration, interrupt and DMA primitives, driver lifecycle, module loading, and the DKM ABI.
- **Wicofe-Drivers** owns optional driver implementations, hardware-specific logic, driver metadata, support records, and driver-specific tests.
- **Wicofe-Specs** owns cross-repository contracts where an interface is shared.
- **Wicofe-ThirdParty** tracks imported code, firmware, and external provenance.

A driver must consume documented kernel interfaces. It must not create private competing definitions of kernel-owned structures or call undocumented kernel internals.

## 2. Driver classes and placement

A driver is classified by dependency and boot criticality, not by convenience.

| Class | Placement guidance |
|---|---|
| Boot-critical storage or bus driver | Built-in initially unless a verified early-module path exists |
| Core interrupt controller / timer | Normally built-in |
| Optional peripheral driver | DKM candidate |
| Experimental or unsupported hardware | Disabled or development-only |
| Driver with unresolved licensing/provenance | Must not be included in distributable builds |

The kernel configuration selects each component as built-in, module, or disabled. A module must not be required to load the filesystem that contains the only copy of that same module.

## 3. Layering

A hardware driver should be split conceptually into:

1. **Bus/transport binding** — PCI, USB, platform, virtio, or another documented bus.
2. **Hardware-specific core** — device registers, command protocol, queues, and device state.
3. **Kernel subsystem adapter** — block, network, input, audio, graphics, or another kernel-owned subsystem.
4. **Lifecycle and diagnostics** — probe, initialization, quiesce, removal, reset, and error reporting.

Hardware-specific code must not bypass the owning kernel subsystem to publish a second incompatible userspace interface. Userspace APIs belong to the corresponding system ABI/service design.

## 4. Probe and binding contract

A driver must only bind when all required identity and capability checks pass.

The probe path must:
- Match vendor/device identifiers and, where applicable, class, subsystem, revision, interface, and compatible strings.
- Validate required device capabilities rather than relying on model-name assumptions.
- Verify that required kernel interfaces and hardware resources are available.
- Request resources through kernel-owned APIs; never assume ownership of an MMIO range, I/O port, IRQ, DMA channel, or bus address.
- Validate register windows and device-reported lengths before access.
- Initialize hardware into a known state before exposing it to consumers.
- Publish the device only after initialization succeeds.

A failed probe must release all acquired resources and leave no published device or callback.

## 5. Resource, DMA, and interrupt rules

The kernel owns resource arbitration and the authoritative DMA/interrupt APIs.

Driver requirements:
- Treat device-provided lengths, descriptors, and completion data as untrusted.
- Check arithmetic and bounds before indexing buffers or register windows.
- Use the kernel DMA mapping API; do not assume CPU physical addresses equal DMA addresses.
- Honor device DMA masks, alignment, boundary, and coherency constraints.
- Follow documented cache-maintenance rules on non-coherent architectures.
- Keep interrupt handlers bounded and non-blocking where required by the kernel contract.
- Defer lengthy work to the kernel's approved work mechanism.
- Acknowledge and mask device interrupts according to hardware specifications.
- Ensure no interrupt or deferred callback can execute after teardown begins.

The exact APIs, locking primitives, interrupt context rules, and DMA mapping semantics are kernel-owned and remain subject to the relevant kernel specifications.

## 6. Lifecycle

The driver implementation must support the lifecycle required by its kernel binding:

- **Unbound:** no resources owned.
- **Probing:** resources may be acquired but device is not externally visible.
- **Active:** device is initialized and published.
- **Quiescing:** new operations are rejected; outstanding work is drained or cancelled.
- **Suspended:** hardware is in a documented low-power or stopped state, if supported.
- **Removing:** registrations and resources are being released.
- **Failed:** initialization or runtime failure has disabled the device.

For DKM drivers, module unload is governed by the kernel's DKM lifecycle rules. A driver must not claim unload safety merely because its remove callback returned. The kernel must also prove that IRQs, callbacks, work items, device references, and executing code are quiescent.

## 7. Error handling and recovery

Drivers must distinguish:
- Unsupported hardware or capability.
- Resource conflict.
- Transient I/O or timeout.
- Device removal or surprise disconnect.
- Hardware reset/recovery failure.
- Data-integrity or protocol error.

Retries must be bounded and must not duplicate non-idempotent device commands. On unrecoverable failure, the driver must stop publishing new work, fail outstanding requests through the owning subsystem, and preserve enough diagnostics to identify the device and failure stage.

Do not silently downgrade to unsafe DMA, polling, or reduced validation.

## 8. Hardware support record

Every driver directory must include a support record with:

- Driver name and purpose.
- Bus and hardware identifiers.
- Exact tested device models/revisions where known.
- Supported architectures and CPU requirements.
- Required kernel configuration and kernel/DKM interface version.
- Firmware files, versions, and redistribution terms.
- Supported features and known limitations.
- Probe, reset, suspend/resume, and removal coverage.
- Test environment and reproducible evidence.
- License identifier, source origin, modifications, and notices.

Use the following support labels consistently:
- **Implemented:** code exists; hardware validation may be incomplete.
- **Build-tested:** builds for the stated target.
- **QEMU-tested:** validated against a named emulated device and configuration.
- **Hardware-tested:** validated on the stated physical hardware.
- **Unsupported:** known not to work or not yet implemented.

Never label a device simply "supported" without stating the evidence level.

## 9. Repository layout

Recommended layout:

```text
drivers/
  <subsystem>/
    <driver-name>/
      README.md
      src/
      tests/
      firmware/          # only where redistribution is permitted
      metadata/          # once the DKM metadata contract is finalized
docs/
  INDEX.md
  DRIVER_SUBSYSTEM_DESIGN_0.1.md
  hardware/
    <driver-name>.md
```

Do not create placeholder driver directories that imply hardware support. Add a driver when there is a concrete implementation or a deliberately scoped experimental prototype.

## 10. Build and packaging

The build system must eventually produce architecture-specific artifacts and record:
- Target triple and architecture.
- Kernel configuration identity.
- DKM ABI version and required interface set.
- Compiler/toolchain version.
- Build options and enabled features.
- Source revision and reproducible build inputs.
- License/provenance manifest.

The final object format, relocation set, metadata encoding, signing envelope, and module filename convention are not selected by this document. They must follow the reviewed Wicofe-Kernel DKM specification. Do not ship a module based only on a successful link.

## 11. Security and licensing

A kernel driver executes with kernel privilege. Treat hardware responses, firmware, device descriptors, and imported source as untrusted inputs.

Requirements:
- Bounds-check all device-controlled lengths and indices.
- Avoid unchecked pointer arithmetic and integer overflow.
- Do not expose uninitialized kernel memory to devices or consumers.
- Keep writable and executable memory permissions consistent with kernel policy.
- Record firmware and imported-code provenance.
- Preserve required copyright and license notices.
- Do not assume MPL-2.0 covers third-party firmware or vendor code.
- Flag GPL-only or otherwise incompatible dependencies for an explicit project licensing decision; do not silently copy them.

## 12. Validation matrix

At minimum, driver-specific tests should cover:
- Positive and negative hardware-ID matching.
- Malformed or truncated device descriptors.
- Register-window and DMA-boundary validation.
- Timeout, reset, and repeated-error behavior.
- Interrupt storms and teardown races where applicable.
- Device removal during active I/O.
- Probe failure after each resource-acquisition stage.
- Suspend/resume if claimed.
- x86-64 and AArch64 build/test paths where supported.
- Fuzzing for parsers that consume device-controlled structures.

QEMU evidence and physical-hardware evidence must be recorded separately. A build-only test does not prove hardware operation.

## 13. Decisions still open

- Stable kernel driver registration and device model APIs.
- Exact DKM object format, metadata, relocation, and signing policy.
- Versioned subsystem operation tables.
- Driver-specific threading and work-queue contract.
- DMA and cache-coherency API details.
- Firmware loading and trust policy.
- Initial supported hardware list and priority order.

## 14. Completion criteria

A driver is ready for inclusion only when its support record is complete, resource ownership follows kernel contracts, error and teardown paths are tested, licensing is reviewed, and its claimed support level is backed by reproducible evidence.

**This document is a draft implementation guide. It does not declare any particular hardware driver implemented or supported.**
