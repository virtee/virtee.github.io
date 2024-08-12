---
layout: page
---

VirTEE is an Open Community dedicated to building FLOSS components to enable the construction of Virtualization-based TEEs (Trusted Execution Environments) using technologies such as [AMD SEV-SNP](https://www.amd.com/system/files/TechDocs/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more.pdf), [Intel TDX](https://software.intel.com/content/www/us/en/develop/articles/intel-trust-domain-extensions.html) and [Armv9 Realms](https://www.arm.com/why-arm/architecture/security-features/arm-confidential-compute-architecture).

## Communication channels

- Chat: [#virtee on Matrix](https://matrix.to/#/#virtee:matrix.org)

## Current projects

- [kbs-types](https://github.com/virtee/kbs-types): Rust (de)serializable types for KBS

- [reference-kbs](https://github.com/virtee/reference-kbs): A reference implementation of the KBS attestation protocol

- [roadmap](https://github.com/virtee/roadmap): The official VirTEE planning and feature roadmap.

- [sev](https://github.com/virtee/sev): Rust library exposing APIs for the AMD SEV-SNP platform

- [sev-snp-measure](https://github.com/virtee/sev-snp-measure): A tool and library for calculating an AMD SEV-SNP expected virtual machine measurements.

- [sev-snp-measure-go](https://github.com/virtee/sev-snp-measure-go): A direct port of sev-snp-mesure for Go-lang integration.

- [snpguest](https://github.com/virtee/snpguest): A utility for managing AMD SEV-SNP enabled virtual machines.

- [snphost](https://github.com/virtee/snphost): A utility for AMD SEV-SNP enabled platforms administration.

- [tdx](https://github.com/virtee/tdx): Rust library exposing APIs for Intel Trusted Domain eXtensions (TDX).

*Do you have a project that you would see listed here? [Propose a change to this page!](https://github.com/virtee/virtee.github.io/blob/gh-pages/index.md)*

## Other resources

- [Using SEV with AMD EPYC™ Processors](https://www.amd.com/content/dam/amd/en/documents/epyc-technical-docs/tuning-guides/58207-using-sev-with-amd-epyc-processors.pdf)

- [SEV-SNP Platform Attestation Using VirTEE/SEV](https://www.amd.com/content/dam/amd/en/documents/developer/58217-epyc-9004-ug-platform-attestation-using-virtee-snp.pdf)

- [Using SEV with QEMU and libvirt](https://libvirt.org/kbase/launch_security_sev.html)

- [SUSE's AMD Secure Encrypted Virtualization (AMD-SEV) Guide](https://documentation.suse.com/sles/15-SP6/pdf/article-amd-sev_en.pdf)

- [Trying out the SEV flavor of libkrun](https://github.com/containers/libkrun/wiki/Trying-out-the-SEV-flavor-of-libkrun)

## FAQ

### What is a TEE?

According to the [CCC](https://confidentialcomputing.io/) (Confidential Computing Consortium), a TEE is as an environment that provides a level of assurance of the following three properties:

- **Data confidentiality**: Unauthorized entities cannot view data while it is in use within the TEE.
- **Data integrity**: Unauthorized entities cannot add, remove, or alter data while it is in use within the TEE.
- **Code integrity**: Unauthorized entities cannot add, remove, or alter code executing in the TEE.

For more information, check [this whitepaper](https://confidentialcomputing.io/wp-content/uploads/sites/10/2023/03/CCC_outreach_whitepaper_updated_November_2022.pdf) published by the CCC.

### What is a Virtualization-based TEE?

It's a TEE that's constructed using [Hardware-assisted Virtualization](https://en.wikipedia.org/wiki/Hardware-assisted_virtualization), combined with other technologies (AMD SEV-SNP, Intel TDX, or Armv9 Realms) which enable the guest owner to verify the integrity and confidentiality of the Virtual Machine.


VirTEE is a member project of the Confidential Computing Consortium (CCC)

<a href="https://confidentialcomputing.io">
<img src="https://raw.githubusercontent.com/confidential-computing/artwork/main/ccc/confidential_computing_consortium-logo-horizontal-color.svg" width="25%" height="25%" alt="CCC Logo"/></a>
