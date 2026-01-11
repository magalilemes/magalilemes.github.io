+++
title = "cd ~/linux"
date = "2022-02-21"
description = "Working directory is now the Linux kernel."

[taxonomies]
tags = ["linux"]
+++

During this year, I will be contributing to the Linux kernel. More specifically,
I'll add more tests to the AMD graphics driver DML library using the KUnit
framework.

# AMD Graphics Driver in the Linux Kernel

AMD graphics drivers live under the `drivers/gpu/drm/amd` directory in the Linux
kernel code. DRM stands for Direct Rendering Manager, and its folder intends to
provide an interface for interacting with GPUs. Inside DRM, we can find the AMD
directory, supporting AMD graphic cards.

# Display

AMD display driver has essentially two pieces: Display Core (DC) and Display
Manager (DM)[^1]. The first one is OS-agnostic and handles hardware programming
and resource management. The second one, though, is OS-dependent and has
implementations of amdgpu base driver and DRM.

# Display Mode Library (DML)

In `drivers/gpu/drm/amd/display/dc/dml`, we find the Display Mode Library, which
"handles clock, watermark, and bandwidth calculation for
[DCN](https://www.kernel.org/doc/html/latest/gpu/amdgpu/display/dcn-overview.html)"[^2].
This is a mathematical library that AMD graphic drivers use. In its
[Makefile](https://gitlab.freedesktop.org/agd5f/linux/-/blob/amd-staging-drm-next/drivers/gpu/drm/amd/display/dc/dml/Makefile),
it is also described as a "'utils' sub-component of DAL. It provides the general
basic services required by other DAL subcomponents."

# KUnit

One way to ensure that things are working the way they are supposed to is
through testing. Naturally, that also applies to software applications, and when
it comes to the Linux kernel, it couldn't be different[^3]: kselftest and KUnit
are two tools used to write tests, each serving a different purpose.

- **kselftest** uses scripts and programs to test features in the userspace.

- **KUnit** is a framework that provides unit testing, which is the process of
testing small components of code. This allows developers to test the behavior of
pieces of code inside the kernel source code, instead of testing an entire
feature.

# Capstone Project
Due to DML's mathematical nature, introducing unit testing to this library is a
reasonable idea, and KUnit seems like a perfect fit for this purpose. As part of
my university capstone project, I'll investigate how some functions in DML work,
add unit tests, refactor these functions and possibly make them robust and
well-structured.


[^1]: https://www.kernel.org/doc/html/latest/gpu/amdgpu/display/index.html
[^2]: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=061bfa06a42a9f6cd192bae61a4bf0e746eccb39
[^3]: https://www.kernel.org/doc/html/latest/dev-tools/testing-overview.html
