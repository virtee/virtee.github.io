---
layout: post
title:  The Case for Confidential Workloads
author: Sergio López
categories: [Confidential Workloads]
---

If you're familiar with the concept of **Virtualization-based Confidential Computing**, you surely have heard about **Confidential Virtual Machines** and **Confidential Containers** (if you don't, I'll give a brief introduction to them in a second). What I'll be trying to demonstrate here is that there's both the space and the need for a third design: **Confidential Workloads**.

First, let's briefly introduce each design:

- **Confidential Virtual Machines** are Virtualization-based TEEs ([Trusted Execution Environments](https://en.wikipedia.org/wiki/Trusted_execution_environment)) that **execute a regular, mostly unmodified, guest operating system**. They are almost identical to regular Virtual Machines, except for the fact that they run in a Virtualization context that has HW-assisted Confidential Computing technologies enabled. Some Virtualization APIs such as [libvirt](https://libvirt.org) allow you to [launch Virtual Machines enabling such technologies](https://libvirt.org/kbase/launch_security_sev.html).

- **Confidential Containers** are **regular containers that are executed inside a Virtualization Based TEE**, which is running a **minimal guest operating system** that's dedicated solely to the task of executing those containers. The prime example of this design is [Kata Confidential Containers](https://github.com/confidential-containers)

- **Confidential Workloads** are **autonomous, mission specific workloads** that run inside a **dedicated Virtualization-based TEE**. They may make use of a minimal kernel, or have one statically-linked to it (unikernel case), but they **shouldn't depend on any other binary components**. There are already projects that follow this design, such as [Enarx](https://enarx.dev/) and [libkrun-tee](https://github.com/containers/libkrun/wiki/Trying-out-the-SEV-flavor-of-libkrun).
  

## Confidential Workloads Pros and Cons

### Advantages

- **Smaller TCB (Trusted Computing Base)**: ["The security of a system if often inversely proportional to it's size and complexity"](https://www.usenix.org/conference/14th-usenix-security-symposium/minimizing-tcb). Confidential Workloads enable the reduction of the TCB in both the Host and Guest contexts.

- **Static attack surface**: Once a Confidential Workload image is created and measured, it can't change. There's **no way to extend its functionality** beyond its current state without rebuilding the image. This implies **the attack surface is well-known in advance**, enabling the possibility of **enacting countermeasures in advance**.

- **Clear context boundaries**: While **integrating multiple workloads into a single TEE** may be tempting as a way to simplify some architectures, this strategy also **blurries the boundaries between contexts**, which makes harder to guarantee that the data is always properly contained. Having **clear context boundaries also enables the enforcement of certain policies while crossing them**, such as forcing the use of TLS for network connections.

- **Specialized Virtual Machine**: As a Confidential Workload image is static and **runs in a dedicated Virtual Machine**, the latter **can be optimized for that particular workload**, omitting components that are not required and prioritizing elements that are critical to it.

- **Custom storage models**: By **removing the need for a guest operating system** that sustains the workload, Confidential Workloads become able to **use arbitrary storage models** with different degrees of persistence, and **including the possibility of having no storage at all**.

### Disadvantages

- **Smaller workload density**: Not being able to consolidate multiple workloads in a single TEE implies a **higher consumption of physical resources**.

- **Higher transition costs**: While the Confidential Workloads design doesn't necessarily imply the need of rewriting a workload from scratch (projects like [libkrun-tee](https://github.com/containers/libkrun/wiki/Trying-out-the-SEV-flavor-of-libkrun) illustrate that it's possible to have a Confidential Workload design that provides a Linux ABI), **it doesn't allow the direct translation of a conventional workload into a confidential one** (whether this is actually good idea or not should be object of a different debate).


## Conclusions

With its unique combination of characteristics, advantages and disadvantages, Confidential Workloads bring to the table **an alternative that fulfills the confidentiality needs of certain workloads better than Confidential Virtual Machines and Confidential Containers**.

This allows me to conclude that **no Confidential Computing solution is fully complete without including the Confidential Workloads design among its deployment models**.
