+++
title = "What's Guix"
date = 2021-01-26

[taxonomies]
tags = ["guix", "outreachy"]
+++

# Definition

[Guix](https://guix.gnu.org/) is both the name to an operating system distribution and a
package manager. As it is developed by the GNU Project, it follows
the principles of Free Software and thus all packages provided are
free software. One interesting thing is that it is mainly written in
[Guile Scheme](https://www.gnu.org/software/guile/), a functional language from the LISP family, which,
personally, has proven to be really nice to code with.

If you use another GNU/Linux distribution, Guix may be used as a
package manager, so that you can have access to all packages
available for the Guix operating system. I, for instance, have Guix
on top of Ubuntu.

## Installation

I installed Guix using the shell installer script that can be found
at this [link](https://guix.gnu.org/manual/en/html_node/Installation.html).

## Using Guix

As mentioned before, I use Guix as a package manager, meaning that
it manages installing, upgrading, removing packages and so
on. Below is a quick demonstration of a simple usage of Guix:

Suppose you want to install a flash cards app, you can search for
it with:

```
$ guix search flash cards
```

This way, information like version, license, webpage and a
description of packages matching what you're looking for will be
shown on your screen.

Then, after seeing what your options are, you decide to install
[Anki](https://apps.ankiweb.net/). This can be simply done too:

```
$ guix install anki
```

Notice how there's no need to have privileged access - which
means, no root privileges are necessary - to install a
package. Now, after installing it, you might wonder how this
package is definied in the Guix source code and you might even want
to **edit** it:

```
$ guix edit anki
```

For some reason, you want to remove a package. Then go
ahead and:

```
$ guix remove anki
```

And, finally, to quickly have a list all of the packages installed:

```
$ guix package -I
```

# My internship goal

Hopefully, after understanding the basics of Guix, you might wonder
how my internship fits into all this. My goal is to implement a
subcommand similar to 'git log', but specific to Guix.

Packages have different versions, and, for many reasons, sometimes
it's better to have one instead of another. Guix makes having older
versions of packages possible through the [time-machine](https://guix.gnu.org/manual/en/html_node/Invoking-guix-time_002dmachine.html) command. A
user can, for instance, provide a /commit hash id/ as an option to
this command (`guix time-machine --commit=<commit-id> -- build
<package>), and then build a package as it was defined back
then. The commit hash id is necessary as the package definitions are
in a Git repository.

With `guix git log`, Guix commit history will be available pretty
easily and it will be possible to retrieve information related to,
for instance, when a package was last upgraded. Some of the work
done so far is described in my [second blog post](/blog/present-and-future/).
