---
layout: post
title: Easier git commands using git-number
date: 2022-01-07 23:16 +0000
description: Easier git commands using git-number, which can be used to number each file listed in git commands for a quicker way to choose files
---
_This post describes `git-number` which can be used to number each file listed in git commands for a quicker way to choose files._

If you work with git on the command line, you'll be used to seeing the output of `git status`. e.g.,

```shell
cdrussell.github.io % git status                                                                                                    main (cdrussell.github.io)
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   _posts/2016-08-10-SSH-Keys-with-Multiple-GitHub-Accounts-c67db56f191e.md
        modified:   _posts/2018-03-21-Recycler View and ListAdapter, More Animations and Less Code.md
        modified:   _posts/2022-01-07-easier-git-commands-using-git-number.md
```
 
If you want to add these files you need to run `git add`, and for each you have to provide the filename. Sometimes you can regex to make this easier, and sometimes you might be adding everything, but other times you need to choose individual files and it's a pain to have to provide this full name each time. e.g., 

```shell
git add _posts/2016-08-10-SSH-Keys-with-Multiple-GitHub-Accounts-c67db56f191e.md  
git add _posts/2018-03-21-Recycler View and ListAdapter, More Animations and Less Code.md
git add _posts/2022-01-07-easier-git-commands-using-git-number.md
```

_Tedious_. Enter `git-number`.

## What git-number does
`git-number` will number each file and allow you to use just that number in future git commands instead of having to type out the full filename.

Let's run `git-number status`.

```shell
cdrussell.github.io % git-number                                                                                                            main (cdrussell.github.io)
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
1       modified:   _posts/2016-08-10-SSH-Keys-with-Multiple-GitHub-Accounts-c67db56f191e.md
2       modified:   _posts/2018-03-21-Recycler View and ListAdapter, More Animations and Less Code.md
3       modified:   _posts/2022-01-07-easier-git-commands-using-git-number.md
```

It's a subtle difference, but you'll see each file is numbered on the left-hand side.

To add the files now, we can provide the number instead of the full filename. e.g., 

```shell
git-number add 1
git add _posts/2016-08-10-SSH-Keys-with-Multiple-GitHub-Accounts-c67db56f191e.md 
```

☝️ Both of these commands do the same thing. Which would you rather type? You can also alias `git-number` to something quicker to type.

## Installing git-number
`brew install git-number`

## Further reading
- See [Official GitHub page](https://github.com/holygeek/git-number) for more info.
- [git-number to the command line rescue](https://jkl.gg/b/git-number/) by [Kaushik Gopal](https://twitter.com/kaushikgopal), which has more details and is where I discovered that this tool exists.