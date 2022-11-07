---
layout: post
title: Attestable, Confidential Workloads with libkrun and AMD SEV-SNP
author: Tyler Fanelli
categories: [Confidential Workloads, libkrun, SEV-SNP]
---

**NOTE:** Before reading this post, you may be interested in what confidential workloads are and why/how they are used. To address these questions, I recommend you check out a previous blog post on the VirTEE blog from Sergio Lopez entitled "[The Case for Confidential Workloads](https://virtee.io/the-case-for-confidential-workloads/)".

With our latest contributions, libkrun (along with "[crun](https://github.com/containers/crun/blob/main/README.md)", "[podman](https://docs.podman.io/en/latest/)", and "[oci2cw](https://github.com/virtee/oci2cw/blob/main/README.md)") has become the first open source platform to offer attestable confidential workloads on the AMD SEV, SEV-ES, and SEV-SNP TEE environments. In this post, I'll be giving an overview of the AMD SEV-SNP attestation process and showing how to run an attested confidential workload with libkrun in the SEV-SNP environment.

## Background Information

- **AMD SEV-SNP**: SEV-SNP (Secure Encrypted Virtualization - Secure Nested Paging) is the third-generation SEV architecture offered by AMD. It builds on the previous two SEV generations (SEV and SEV-ES), allowing for encrypted in-use data (RAM) and register state while introducing memory integrity protection to prevent a number of malicious hypervisor attacks. You can read more about SEV-SNP in "[this white paper from AMD](https://www.amd.com/system/files/TechDocs/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more.pdf)".
- **Attestation**: A process intended to establish trust between a client running a confidential workload and the platform in which the workload is being run on. Attestation is a process that confirms that only code/data that is known to the client and intended to be used is included in the TEE workload, and that the workload is running on a verified TEE architecture. An explanation of attestation in a bit more detail can be found "[here](https://globalplatform.org/attestation-and-tee-cybersecurity-controls-with-privacy-for-cloud-access/)".
- **libkrun**: A dynamic library allowing for programs to acquire the ability to run as processes in a partially isolated environment using KVM virtualization. With the libkrun-tee package, these processes can be run confidentially, where not even the root process can view into the workload's code/data.

## Why is attestation needed?

When intending to run a confidential workload on another system (e.g. on a machine from a cloud provider), it is reasonable for a client to inquire "How do I know this workload is actually running on a TEE, and how do I know that my workload (and ONLY my workload) are what is running inside this TEE?". For sensitive workloads, a client would like to ensure that there is no nefarious code/data being run, as this code/data can be violating the integrity of the TEE.

The purpose of an attestation is to cryptographically prove to a client that their application is running on a trusted TEE, and only the code/data of their program is being run inside the TEE. Thus, the integrity is not violated by any external factors.

## libkrun-tee's Attestation process on SEV-SNP

It is important to note that different TEE architectures (i.e. AMD SEV, SEV-SNP and Intel SGX, TDX) are attested differently. This post will only be describing libkrun's SEV-SNP attestation protocol.

libkrun utilizes the "[Key Broker Service (KBS)](https://github.com/confidential-containers/kbs/blob/main/docs/kbs_attestation_protocol.md)" protocol for attestation. The KBS protocol defines the communication between a Key Broker Client (KBC, in this case being the libkrun workload) and a trusted Key Broker Service (KBS). There is a simple "Request-Challenge-Attestation-Response" (RCAR) method to facilitate workload attestation.


### Before you begin: Register Workload

Please note that the Register Workload interface is not included in the KBS attestation model. Registering a workload is specific to libkrun and reference-kbs.

Before starting a confidential workload, we must first register some information beforehand of the workload ID, registered measurement (hex-encoded), configuration information, and the passphrase to eventually unlock the LUKS-encrypted disk (needed in order to run an application). A "measurement" is a SHA-512 hash of all of the memory regions that will be included in the workload VM's address space. This includes the firmware, kernel, workload application, etc. When this VM is then booted, the AMD PSP will do the same, and present a SHA-512 hash of the VM's memory regions in it's attestation report.

```json
{
    "workload_id": "my_workload",
    "launch_measurement": "hex-encoded-measurement",
    "tee_config": "{\"flags\":{\"bits\":63},\"minfw\":{\"major\":0,\"minor\":0}",
    "passphrase": "my_secret_passphrase", 
}
```

Once this workload is registered, a krun VM will then be able to communicate to the attestation server with its found measurement from the AMD PSP during attestation. The two measurements can then be compared.

### KBS Request

The initial communication to the KBS is established by the KBC by sending a JSON Request structure to the KBS. A KBS Request contains the following:

1. KBS protocol version number: The protocol version number being used by the KBC.
2. The TEE architecture that the workload is being run on.
3. Extra parameters - Miscellaneous data that can be used by the attestation server.

Currently, there is only one version of the KBS protocol. Thus, the KBS server can be expected to ignore the version field. In our current example, we are attesting the AMD SEV-SNP environment. SEV-SNP does not require any extra miscellaneous data in the request step, so the extra parameters field can also be ignored.

Here is an example KBS Request structure in JSON format.

```json
{
    "version": "0.0.0", // Ignored.
    "tee": "snp",       // AMD SEV-SNP.
    "extra-params": "", // No extra parameters.
}
```

### KBS Challenge

In receiving a Request from the KBC and validating its data, a KBS will reply with a Challenge structure. In SEV-SNP, a Challenge only contains a nonce word - a random string of data specific to the KBS. When a KBC requests its SEV-SNP attestation report from the Platform Security Processor (PSP), it is allowed to submit up to 64 bytes of data to be included in the report. In order to guarantee freshness, the nonce word (together with a created TEE public key for encrypting the passphrase) must be used by the KBC to build another SHA-512 hash. This hash (known as the "freshness hash") will eventually be rebuilt by the KBS and compared with the attestation report's version to ensure freshness (i.e. that the attestation report came from the PSP and was not pre-cached).

Here is an example KBS Challenge returned by the KBS in JSON format:

```json
{
    "nonce": "vo4jefw854jh5x8ff39f8fh47hf4908fc38u",
    "extra-params": "",     // Also unused in SEV-SNP.
}
```

### KBS Attest

The client and server both serve roles in the attest phase of KBS attestation.

The client will do the following:
1. Create a TEE public key. This key will be used by the attestation server to encrypt the passphrase of the LUKS-encrypted disk.
2. Hash the nonce word from the attestation server with the contents of the TEE public key to create the freshness hash to include in the attestation report.
3. Hex-encode the attestation report's bytes.
4. Serialize the attestation report along with the TEE public key, and send the contents to the attestation server.

If one feels inclined to include a certificate chain, they are able to do so in the "cert_chain" field of the Attestation struct. For libkrun and reference-kbs, we opt to elect the attestation server to fetch the certificate chain.

Here is an example KBS Attestation structure in JSON format.

```json
{
    "tee-pubkey": TeePubKey {
        "alg": "RSA",
        "k_mod": "base64-encoded-rsa-public-modulus",
        "k_exp": "base64-encoded-rsa-public-exponent",
    },
    "tee-evidence": {
        "report": "hex-encoded-attestation-report",
        "cert_chain": "[]", // Attestation server's responsibility for fetching the cert chain.
        "gen": "milan"
    }
}
```

The server will do the following:
1. Recreate the "freshness hash" from the cached nonce and given TEE public key, and compare the two to guarantee the attestation report is fresh.
2. Using the given SEV-SNP generation (at time of writing, this is either "milan" or "genoa"), retrieve the ARK and ASK from the AMD Key Distribution Server.
3. Retrieve the VCEK using the current TCB values.
4. Validate the certificate chain to prove that the given attestation report came from an AMD PSP.
5. Compare the measurement registered beforehand from the client with the measurement from the attestation report.

If all steps succeed, we can verify that the VM is measured and verified, and thus attestation is successful. If any of the steps failed, the server reports a failure to the client.

### KBS Response

The attestation server replies to the client with a pass/fail.

libkrun ONLY: If response is successful, the client can now use the Get Key interface to retrieve the passphrase to the LUKS-encrypted disk.

### Get Key

Like the Register Workload interface, this interface is not an official part of the KBS attestation protocol, but is specific to libkrun and reference-kbs.

The Get Key interface allows for a simple GET request to obtain the passphrase to the LUKS-encrypted disk. Note that this passphrase is encrypted with the previously-created TEE public key and must be decrypted. Once done, a client can now use the passphrase to unlock the disk and run the workload/application.

## Conclusion

Attestation allows for a confidential client to ensure that:

1. Their workload is running on trusted TEE hardware.
2. The software of their workload is measured and verified (i.e. not containing nefarious code/data that can violate the confidentiality of the workload).

With the latest changes in "[libkrun](https://github.com/containers/libkrun)" and "[reference-kbs](https://github.com/virtee/reference-kbs)", one can now use these to create attestable, confidential workloads on SEV-SNP.

## Demonstration

Below is a demonstration of running an attested, confidential podman container. First, a regular (i.e. unencrypted, not using any TEE confidentiality) container is created using podman. The memory contents of this container are read and the secret is examined.

Then, using that same podman container (known as "hellodemo" in the demonstration), we invoke oci2cw to:
1. Format a LUKS-encrypted disk to store the application, disallowing the running of the application until the disk is unlocked.
2. Using a JSON configuration file (hellodemo.json) and a created passphrase, register the container with the reference-kbs attestation server.

We then use podman (with --runtime=krun) to begin executing the newly-created confidential workload (known as "hellodemo-cw" in the demonstration) and attest the workload. Once the workload is successfully attested/started, we examine the confidentiality by trying to read the containers secret from the VM's memory. We subsequently see that we are unable to find the secret when grep'ing for it (because the krun VM's memory is encrypted).

[![asciicast](https://asciinema.org/a/569023.svg)](https://asciinema.org/a/569023)
