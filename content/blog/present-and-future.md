+++
title = "Present and future"
date = 2020-12-22

[taxonomies]
tags = ["guix", "outreachy"]
+++

# What has been done so far

I can't believe it's been three whole weeks. Time has gone by really
quickly. I feel like I've been learning a lot: there are so many new
ways to approach a problem. In order to keep a certain uniformity in
the code, I tend to spend some time reading and analyzing the Guix
source code. This way, I also get to learn some new procedures.

## Motivation

My internship task is to implement a subcommand so that all Guix
commit history can be retrieved easily. Using this subcommand, a
user could know in which commit a package was added or
updated. This will be available with 'guix git log'.

To achieve my goal, I have been using the [Guile Git library](https://gitlab.com/guile-git/guile-git), which
provides bindings to the [libgit2 C library](https://libgit2.org/libgit2). With this module, it's
easy to have repository and commits information.

## Results

First things first, the code I am currently working on can be found
on this [link](https://gitlab.com/magalilemes/guix). As I'm writing this, if you run the code with
`./pre-inst-env guix git log --oneline`[^1], five commits will be
shown on the screen, presented in a format as if it were `git log
--oneline` itself.
If, instead of the hard-coded 5, used for now to speed up my tests,
the number of commits shown is changed to, let's say, 10000 then we
start seeing some interesting results. For instance, suppose I want
to see commits that have 'nano' in their message title:

```
$ ./pre-inst-env guix git log --oneline | grep nano
eeb3db0 gnu: nano: Update to 5.4.
e6b2a3e gnu: nano: Update to 5.3.
5abbf43 gnu: nano: Update to 5.2.
0b4327f gnu: Add nanomsg.
5b893f4 gnu: nano: Update to 5.1.
c09f22a gnu: nano: Update to 5.0.
dfc4265 gnu: nano: Update to 4.9.3.
```

It's easy to see in which commit the nano text editor was last
updated, or updated to a specific version, or when the nanomsg
library was first added to Guix.

## A stone in the middle of the road[^2]

Although these three weeks have been amazing and I have learned a
lot, the road has not always been easy. I faced a few problems
while trying to make the subcommand work as it is right now.

Of course, there were silly, easy-to-solve problems that only
required a little bit of attention to find the solution. An example
was when I spent thirty minutes trying to understand why I was
getting an error message, and it turned out that I hadn't imported
the required module to use a certain function.

The hardest problem I had was *segmentation fault* occurring when
running a piece of code I wrote. What I wanted to do was quite
simple: I wanted to write a function that, given a Git repository
path, would return all the Git commits found there. The code I
wrote seemed pretty simple, so having it lead to a segfault
wasn't something I expected. Here's what I was trying to do, using
Guix REPL, which has proved to be great for testing how things
work - or figuring out why they don't work.

```
$ guix repl
GNU Guile 3.0.4
Copyright (C) 1995-2020 Free Software Foundation, Inc.

Guile comes with ABSOLUTELY NO WARRANTY; for details type `,show w'.
This program is free software, and you are welcome to redistribute it
under certain conditions; type `,show c' for details.

Enter `,help' for help.
scheme@(guix-user)> (use-modules (guix git) (git) (guix channels) (ice-9 match) (srfi srfi-1))
scheme@(guix-user)> (define cache (url-cache-directory (channel-url %default-guix-channel)))
scheme@(guix-user)> cache
$1 = "/home/magali/.cache/guix/checkouts/pjmkglp4t7znuugeurpurzikxq3tnlaywmisyr27shj7apsnalwq"
scheme@(guix-user)> (let* ((repo (repository-open cache))
                        (latest-commit
                            (commit-lookup repo (reference-target (repository-head repo)))))
                    (let loop ((commit latest-commit)
                                (res (list latest-commit)))
                        (match (commit-parents commit)
                            (() (reverse res))
                            ((head . tail)
                                (loop head (cons head res))))))
Segmentation fault (core dumped)
```

And the solution to that is actually a workaround, as I stepped
on a [bug](https://gitlab.com/guile-git/guile-git/-/issues/21) in the Guile Git library that [hasn't been fixed](https://lists.gnu.org/archive/html/guix-devel/2020-12/msg00226.html)
yet. Instead of opening the repository inside let*, if the Git
repository is defined in a global variable, then I can get the
commits, like I wanted to.

# Next steps

Up to now, what `guix git log` does is walk the commit history,
visiting only the first parent of a commit, ignoring any other
parent and thus leaving behind other commits. So, on my to-do list,
I have  to fix this, finding a better way to walk the commit
history, including every commit.
There are also two other improvements that I should do:

- Manage with different channels, as Guix can be extended using
different channel[^3]. Right now, the subcommand can only
handle with the default channel.

- Add some other formatting options. At the time of this writing,
there's only `guix git log --oneline` but the plan is to expand
it so that we can also have different formatting with `guix git
log --format=<format>`, and `<format>` could be any of `short`,
`medium` or `full`.

[^1]: https://guix.gnu.org/manual/devel/en/html_node/Running-Guix-Before-It-Is-Installed.html 

[^2]: https://allpoetry.com/In-The-Middle-Of-The-Road

[^3]: https://guix.gnu.org/manual/en/html_node/Channels.html
