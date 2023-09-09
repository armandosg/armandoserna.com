---
title: "GIT: How to commit only parts of a file?"
description: "A tutorial for using git patch"
pubDate: "Sep 04 2021"
heroImage: "/git-how-to-commit-only-parts-of-a-file-hero-image.webp"
---

## Scenario

It probably happened to you that after making lots of changes to some code files, you see one of those changes is ready so you decide to commit only this single change (maybe because you want to share it with other colleagues or maybe the client is asking to have this change ready).

So you open your terminal and run the git diff command on the file where your change is and see the following output:
![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0jtid31t8atwk1kl03kw.png)

And there it is, the part of the file you want to commit is the where the line `font-weight: normal;` is added. But there is also another change in this file.

So using `git add style.css` isn't an option for this scenario, as it would be introducing two different changes into the commit I want to share.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/005a0x75yc2ywizzh5fg.png)

## The solution: the --patch argument

So the solution to this usual scenario (it has happened to me a million times) is adding the argument `--patch` to the `git add` command.

Doing this will prompt us for each change (or hunk) we have in our file if it should be part of our changes to be committed.

So after running `git restore style.css --staged` to restore our previously staged changes, we run `git add style.css --patch` and see the following output:

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/crt7r3tu1de867hodq63.png)

And as you can see, now git is asking us if the line that contains the change to the `background` should be included or not.

We can input `y` for yes or `n` for no, or simply input a `?` to get a description of each option.

So we decided to not include the first hunk and then include the second one as we can see in the image below:

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a5k24k9odqz6ymqh6rqg.png)

Then if we inspect our staged changes there's only the changes we want.

![image](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9wpc6rhqolayzsahz92c.png)

## Why would I ever want to commit one part of the file?

In git there is a concept called "atomic commits", which are commits that only contains a single logical change to a part of the code.

Using atomic commits makes our repositories easy to navigate and to inspect, and has a lot of benefits for teams that needs to review older parts of the code in order to fix a bug for example.
