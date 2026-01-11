+++
title = "Expectations"
date = 2021-01-26

[taxonomies]
tags = ["guix", "outreachy"]
+++

# How it started: potential timeline

By the time Outreachy applications are open, all potential applicants can see
a list of all projects available and their potential timeline. After recording
a first contribution - or possibly more - to the desired project, it's time to
start working on the final application. It's required that the applicant
provide a timeline as well, pointing the goals and tasks, and when they could
be finished. In my application, this was the timeline I presented:

*Dec.   1, 2020 - Internship starts*

1- Subcommand guix git log will display "hello world".

2- Learn how to use Guile-Git.

*Dec.  11, 2020 - Initial feedback due*

2- Continue learning how to use Guile-Git, and also improving it.

*Dec.  18, 2020 - College classes end*

3- Walk over the commit tree history

4- Manage with several channels.

*Jan.  12, 2021 - Mid-point feedback due*

5- Option for sorting by date, channels, etc.

6- Add regexp support: `guix git log --grep=`.

7- Add commit formatting: `guix git log --format=`.

8- Add commit limiting: `--after`, `--before`, `--author`.

*March  2, 2021 - Final feedback due*

*March  2, 2021 - Internship ends*

# How it's going

As for now, the goals and tasks have remained almost the same, but the
schedule I presented changed considerably. See how things have been going so
far, and bear in mind that my weeks start on Sunday and end on Saturday.

*Week #1* Dec 1st  - Dec 5th

• Create my own Guix repository on Gitlab.

• Write a blog post (both for Guix and for my personal blog).

• Tweak and modify Guix source code, in order to learn
how things are built, as well as start using `guix repl`.

• Create the `guix git log` subcommand.

• The subcommand above should show the checkout path.

*Week #2* Dec 6th  - Dec 12th

• Improve code from past week.

• Add fictitious options to the subcommand, and `guix git log --help` should
display these options.

*Week #3* Dec 13th - Dec 19th

• Retrieve a few commits from the checkout path and display them.

• Add the `--oneline` option, and have the commits shown like `git log
--oneline` format.

*Weeks #4 and #5* Dec 20th - Jan 2nd

• Manage with different channels.

• Add `--format[FORMAT]`, and FORMAT can be any of `oneline`, `medium`
or `full`.

*Week #6* Jan 3rd  - Jan 9th

• `guix git log --channel-cache-path` should show the path to all
channels.

• The subcommand should show commits from all channels.

• Add `--pretty=<string>`

*Week #7* Jan 10th - Jan 16th

• Add `--grep=REGEXP`.

*Week #8* Jan 17th - Jan 23th

• Compare time taken by 'git log' and 'guix git log'.

• Write tests.

*Week #9*  Jan 24th - Jan 30th

• Learn more and try to use lazy evaluation.

Looking back at the planned timeline, you can see that only the commit
limiting and sorting haven't been done. At the moment, we're trying to
improve and optimize the current code.
