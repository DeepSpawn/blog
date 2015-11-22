---
layout: post
title: Git Prepare Commit Message
---

If you have workflow that involves including Issue keys in your commit messages and branch names then you should be using a git hook to make your workflow more efficient.

[Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are scripts that get run when a particular action takes place in your git repository.
You can find them inside every git repository in `.git/hooks`. When a repository is intialized this directory will be prepopulated with a bunch of example scripts showing you what you can do with git hooks.
So dont be alarmed when you see a bunch files on your first time looking at it.

There are a number of different places we can hook into, in this case we want to hook into the creation of a commit message. The prepare-commit-msg hook does exactly that,
being run after the default commit message is created but before the commit message editor is opened. We simply need to create a new file called `prepare-commit-msg` and add our script to its body.

This hook takes a few parameters but we only need the first which is the path to the file that contains the commit message so far. 
All we need our script to do is to use git to grab the current branch name, then use a regex to find the issue key in the branch name and write the match of the regex to the start of the commit message file.
Here is an example implementation in Python, but you are free to use whatever scripting langauge you want when writing git hooks. 

<script src="https://gist.github.com/DeepSpawn/7334e5dd6588ca58d7b5.js"></script>

Now you may be thinking to yourself that this seem like a lot of effort to save a few characters per commit message, If so might I remind you of this wonderful is it worth it chart from [XKCD](https://xkcd.com/)
 ![XKCD 1205](http://imgs.xkcd.com/comics/is_it_worth_the_time.png)
This hook is going to shave a few seconds off every commit message so it well worth it.