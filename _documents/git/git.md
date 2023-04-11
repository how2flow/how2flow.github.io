---
permalink: /documents/git/
title: Git
excerpt: "Git and Code Management Guide"
comments: false
toc: true
---

## Git

Git is a distributed version management system based on snapshot streams to track changes in computer files<br>
and coordinate the work of those files among multiple users.<br>

### basis

Git has several basis, it is important to know their concepts.<br>

**repository**:<br>
This represents the storage stored on a project-by-project basis.<br>
A repository is a "container" that tracks all changes to a project file.<br>

**commit**:<br>
This is a record of updates to the code.<br>
It is separated by the hash value.<br>

**branch**:<br>
A split it during shape management.<br>
If you are working temporarily for updates<br>
or tests with other purposes, create and apply new ones.<br>

### info

**Web**: https://github.com, https://about.gitlab.com<br>

**Documentation**: https://git-scm.com/docs/git<br>

**Author's**: https://github.com/how2flow

## Status

Git has four file status.<br>

```
- untracked
- staged
- commited
- modified
```

### Untracked

New files with no record in the repository.<br>
The git add command puts it in a staged state, and after the commit is registered,<br>
it becomes trakable.<br>

### staged

This is the state that is in the stage before it is reflected in the commit.<br>
The <span style="{{ site.code }}">git add</span> command puts it in the stage state.<br>
he <span style="{{ site.code }}">'git restore --staged'</span> command allows you to revert to the Untracked state.<br>

### commited

Staged files are in the state where commit was created with the <span style="{{ site.code }}">git commit</span> command.<br>

### modified

Status of files modified from the current commit.<br>
You must perform a stage to reflect the changes in the next commit.<br>

**git-add**<br>

It is a command to add untracked or modified files in a stage state that can be registered to the commit.
```
$ git add <target>
```

**git-restore**<br>

Restore the modified files to the criteria currently stored in the commit.
```
$ git restore <target>
```

## Functions

The Git supports several functions. It writes some of the most common uses.
```
- clone
- commit
- fetch
- merge
- push
- pull
- rebase
- rebase --interactive
- cherry-pick
```

### clone

Replicate the git repository on the server to the local environment.
```
$ git clone <url>
```
<br>

### commit

You can make files in a staged state into a commit.
```
$ git commit
```

### fetch

Download objects and refs from another repository
```
$ git fetch <remote-name|url>
```
<br>

It is often used for tasks such as merging or rebasing from other repositories.<br>
Only the information on the repository was downloaded,<br>
It will be in a traceable state.<br>

**All kinds of locally performed mergers must have information about the target**<br>
(that is, <span style="{{ site.code }}">fetch is required</span> )<br>

**case 1.** Merging between different branches of the same repository<br>
There is no problem if you clone all-things.<br>
But in other cases,<br>
For example, if you cloned with the options such as <span style="{{ site.code }}">--single-branch</span> option or the <span style="{{ site.code }}">--depth 1</span> option,<br>
If there is only a fraction of a clone that does not have a complete clone,<br>
you must fetch if the target is not currently in your local environment.<br>

### merge

Literally used to merge modifications with existing code.
```
$ git merge <target-branch|target-commit>
```
<br>

When you merge between branches, the merged commit is created separately.<br>
It is called <span style="{{ site.code }}">3-way-merge</span> .<br>

Merging in the same brenches or<br>
between different brenches but with no branched history is<br>
<span style="{{ site.code }}">fast-forward</span> merging.<br>

I don't prefer Merge because it usually messes up the history.<br>

### push

Push the locally updated repository to the server repository.
```
$ git push <remote-name> <branch>
```
<br>

The push can be done in the unit of the brenches.<br>

### pull

It's fetch + merge together.
```
$ git pull <remote-name> <branch>
```
<br>

Download the target repository object and refs stored on the server, and at the same time merge it.<br>
If the local history is linked to the server's history, it is <span style="{{ site.code }}">fast-forward merge</span> ,<br>
and if not, it is <span style="{{ site.code }}">3-way-merge</span> .<br>

### rebase

It does the same thing as merge, which is used to merge brenches or commits,<br>
but the method is different.<br>

Rebase aligns and merges the order between commitments.
```
$ git rebase <to-be-parents-branch|to-be-parents-commit>
```
<br>

The difference from merge is that **request is the commit to be one's own parent**, not the target to merge.<br>
Merging commits using the <span style="{{ site.code }}">rebase</span> cleans up the history.<br>

#### rebase --interactive

You can proceed with the rebase in multiple options at once.
```
$ git rebase -i <target-commit>
```
<br>

From the head commit to the child commit of the target commit, it is subject to rebase.<br>
The target commit may use a hash like other commands that receive the commit as a parameter,<br>
or may use any representation method that may specify the commit, such as a branch name, tag name, and location from HEAD.<br>

There are several options that are supported when performing interactive.<br>
<span style="{{ site.code }}">edit/e</span> : Modifying the contents of the commit operation<br>
<span style="{{ site.code }}">exec/x</span> : Commands that allow you to enter shell commands to run during the database<br>
<span style="{{ site.code }}">pick/p</span> : Use the commit as it is. You can also rearrange or delete the commit<br>
<span style="{{ site.code }}">reword/r</span> : Commands to modify commit messages<br>
<span style="{{ site.code }}">squash/s</span> : Commands that combine that commit with the previous commit<br>

### cherry-pick

take a commit you want and stack it over the current HEAD.<br>
Again, the local environment must have the corresponding commit information.
```
$ git cherry-pick <target-branch|target-commit>
```

## Others

Refer to [Documentation](https://git-scm.com/docs/git)<br>
