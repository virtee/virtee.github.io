---
layout: page
---

VirTEE is an Open Community dedicated to building FLOSS components to enable the construction of Virtualization-based TEEs (Trusted Execution Environments) using technologies such as [AMD SEV](https://developer.amd.com/sev/) (and [SNP](https://www.amd.com/system/files/TechDocs/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more.pdf)), [Intel TDX](https://software.intel.com/content/www/us/en/develop/articles/intel-trust-domain-extensions.html) and [Armv9 Realms](https://www.arm.com/why-arm/architecture/security-features/arm-confidential-compute-architecture).

## Communication channels

- Chat: [#virtee on Matrix](https://matrix.to/#/#virtee:matrix.org)

## Current projects

- [sev](https://github.com/virtee/sev): Rust library exposing APIs for the AMD SEV platform

- [sevctl](https://github.com/virtee/sevctl): Administrative utility for AMD SEV

- [kbs-types](https://github.com/virtee/kbs-types): Rust (de)serializable types for KBS

- [reference-kbs](https://github.com/virtee/reference-kbs): A reference implementation of the KBS attestation protocol

- [oci2cw](https://github.com/virtee/oci2cw): Utility to transform OCI images into Confidential Workloads

*Do you have a project that you would see listed here? [Propose a change to this page!](https://github.com/virtee/virtee.github.io/blob/gh-pages/index.md)*

## Other resources

- [Using SEV with QEMU and libvirt](https://libvirt.org/kbase/launch_security_sev.html)

- [SUSE's AMD Secure Encrypted Virtualization (AMD-SEV) Guide](https://documentation.suse.com/sles/15-SP1/pdf/art-amd-sev_color_en.pdf)

- [Trying out the SEV flavor of libkrun](https://github.com/containers/libkrun/wiki/Trying-out-the-SEV-flavor-of-libkrun)

## FAQ

### What is a TEE?

According to the [CCC](https://confidentialcomputing.io/) (Confidential Computing Consortium), a TEE is as an environment that provides a level of assurance of the following three properties:

- **Data confidentiality**: Unauthorized entities cannot view data while it is in use within the TEE.
- **Data integrity**: Unauthorized entities cannot add, remove, or alter data while it is in use within the TEE.
- **Code integrity**: Unauthorized entities cannot add, remove, or alter code executing in the TEE.

For more information, check [this whitepaper](https://confidentialcomputing.io/whitepaper-02-latest/) published by the CCC.

### What is a Virtualization-based TEE?

It's a TEE that's constructed using [Hardware-assisted Virtualization](https://en.wikipedia.org/wiki/Hardware-assisted_virtualization), combined with other technologies (AMD SEV, Intel TDX or Armv9 Realms) that enable the guest owner to verify the integrity and confidentiality of the Virtual Machine.

