+++
title = "My first contribution to the Linux kernel"
date= 2022-02-21
description = "From scratch to sending the patch."

[taxonomies]
tags = ["linux"]
+++

## Finding something to fix

There are many ways to start contributing to the Linux kernel, and one of them
is by fixing something, be it a coding style disagreement, compile errors or
static check warnings.

Apart from finding something to fix yourself, an alternative is to look at what
automated bots designed to look for bugs and warnings in the kernel have already
found[^1]. One of these bots is the *kernel test robot*[^2], and an advantage of
going through the warnings found by this bot is that it provides the ways to
reproduce the settings that led to the compilation warnings.

In my case, my Linux kernel mentor sent me a [warning reported by the kernel
test robot](https://lore.kernel.org/all/202201252155.rqBWi1tb-lkp@intel.com/)
so that I could work on it and fix it. As mentioned before, the email sent by
the bot contains details to help finding the warning:

- the Git tree;
- the specific commit that triggered the warning;
- a .config file to use when building the kernel
- compiler version;
- static analysis tool used and its version (in this case, sparse);
- steps to reproduce the warning using the tools above.

Some emails sent by the kernel test robot can be found [here](https://linuxlists.cc/profile/44393/kbuild_test_robot>).

## Fixing

After having found something to fix, it's necessary to find the right tree to
work on. The [MAINTAINERS](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/MAINTAINERS)
file on the kernel code contains this information, and there's also the
`./scripts/get_maintainer.pl` script to help with that.

Given that I had to fix a warning found under the `drivers/gpu/drm/amd/`
directory, I cloned the tree found at <https://gitlab.freedesktop.org/agd5f/linux.git>,
made the change accordingly, and recompiled to ensure that the previous warning
had been solved. After that, I commited the change.

## Sending the patch

To send the patch to the appropriate mailing lists, I used [kworkflow](github.com/kworkflow/kworkflow/)
and its `git send-email` wrapper feature. I had already configured my .gitconfig
file using it, so to send my last commit I just ran
~~~
$ kw mail -s -1
~~~
Finally, the email was sent, and all of the steps above led to my first
contribution to the Linux kernel [here](https://lore.kernel.org/lkml/20220202213856.409403-1-magalilemes00@gmail.com/).

## References

[^1]: https://bottest.wiki.kernel.org/
[^2]: https://01.org/lkp/documentation/0-day-test-service
