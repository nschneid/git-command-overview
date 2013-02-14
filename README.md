Git Command Overview
====================

*[Nathan Schneider](http://nathan.cl), 2012-02-13*

General notes on using the `git` version control system based primarily
on a newbie's reading of the [Pro Git book][].

[GitHub.com][] and [Bitbucket.org][] offer free Git repository hosting.

  [Pro Git book]: http://git-scm.com/book
  [Bitbucket.org]: https://bitbucket.org/
  [Github.com]: https://github.com/


References and tutorials
------------------------

*   [Pro Git Book][]: medium-length tutorial
*   [Git User's Manual][]: comprehensive documentation and [glossary][]
*   [Git Reference][]: nice summary of commands
*   [Git - SVN Crash Course][]: short synopsis of git for an SVN user
*   [Git Cheat Sheet][]

  [Git User's Manual]: http://schacon.github.com/git/user-manual.html
  [glossary]: http://schacon.github.com/git/user-manual.html#glossary
  [Git Reference]: http://gitref.org/index.html
  [Git - SVN Crash Course]: http://git.or.cz/course/svn.html
  [Git Cheat Sheet]: http://byte.kde.org/%7Ezrusin/git/git-cheat-sheet-medium.png

Initial configuration
---------------------

When configuring Git for the first time on a machine, I do the
following:

```bash
$ git config --global user.name "Nathan Schneider"
$ git config --global user.email "...my email address..."
$ git config --global core.editor emacs
$ git config --global color.ui true
```

Local repositories
------------------

One of the fundamental things to know about git is that every repository
(or copy thereof) contains the complete history of all its files. Each
version of every file is stored in full, not as a diff. All git has to
keep track of is which file versions were current after any given
commit. The overall repository “version” after a commit is represented
with a SHA-1 hash checksum.

#### Starting a new repository in the current directory
   `git init`
#### Viewing history
   `git log` shows the history of commit messages; add `-p` to see
    the changes in each commit

Files
-----

Files in the working directory can be in the following states:

#### untracked
&nbsp;&nbsp; 0\. not handled by Git

#### tracked

1. __modified or deleted:__ known to Git (in SVN terms, part of the
    “working copy”) but different from the committed file
2. __staged:__ among the batch of files (or more precisely, file
    versions) slated to be committed
    -   staging a file, then modifying it, then committing will
        commit the version of the file at the time of staging, not the
        latest version!
3. __committed:__ permanently part of the repository

#### Changing file states  
-   {0,1} → 2: `git add <file>`
-   2 → 3: `git commit [-m <message>]`
-   {1,2} → 3: `git commit -a [-m <message>]`
-   (if `-m` option is omitted, the configured text editor will open for
    a commit message to be entered)
-   2 → 0 (when a previously untracked file was unintentionally staged
    with `git add`): `git reset <file>`

#### Redoing the previous commit  
  `git commit --amend`

#### Checking status
  `git status`

#### Removing a file  
  `git rm <file>` (stages the removal to git, and if the file remains in
  working directory, removes it)

#### Moving a file  
  `git mv <file>`

#### Putting aside local modifications that have not yet been committed, reverting the working directory to the committed version  
  `git stash [save]`

#### Restoring local modifications that have been stashed  
  `git stash apply`
  
#### [Undoing][] changes in a previous commit  
  `git revert <revision>`, e.g. `git revert HEAD` to undo the most recent
  commit. Note that reverting an earlier commit will retain the changes
  made in any subsequent commits.
  - cf. `git commit --amend` (above) and `git rebase`, though these are
    not advised for commits that have been made public

  [Undoing]: http://book.git-scm.com/4_undoing_in_git_-_reset,_checkout_and_revert.html

Branching and merging
---------------------

As visualized [here][], a *branch* is simply a pointer to a commit
record, which in turn points to a snapshot of files (so branching is
cheap). One can locally create and switch among multiple branches. The
default branch for a new project is called `master`. As commit records
have backpointers to the record of the previous commit from the same
repository, all prior history is available with respect to a given
branch.

  [here]: http://progit.org/book/ch3-1.html

### Branch management

#### Create a branch pointing to the current snapshot of files  
  `git branch <branch-name>`

#### Delete a branch (deletes the pointer, not the files)  
  `git branch -d <branch-name>`

#### View branch descriptions  
  `git branch -v`

### Switching branches

A special pointer, **HEAD**, keeps track of the current branch. You can
switch to a newly created branch or back to an old one.

-   If changing to a branch that points to an older commit,
    `git checkout` effectively reverts the tracked files in the repository
    directory to an older snapshot. But the newer versions of these
    files can later be recovered by switching back to the other branch.
-   If the working directory or staging area has uncommitted changes,
    these must (normally) be dealt with prior to changing to another
    branch.

#### Change the current branch, and "swap" to the snapshot of files that it represents  
  `git checkout <branch-name>`

#### Create a new branch based on an existing one (by default, the HEAD branch) and change to it in one step  
  `git checkout -b <new-branch-name> [<source-branch-name>]`

### What a commit does, in terms of branches

Crucially, the commit operation creates a new commit record **whose
parent is the commit record pointed to by the current (HEAD) branch**,
and then **advances the current branch to point to this new record**.

### Merging

*Merging* is the process of incorporating one branch's changes into
another branch. Let H denote the current (HEAD) branch and B another
branch that is to be merged into it. There are three cases:

-   If B is an ancestor of H in the commit hierarchy, this has no
    effect—the current branch already incorporates the changes that were
    made in B. (TODO: CHECK)
-   If H is an ancestor of B in the commit hierarchy, this is called a
    **fast forward**: H will be advanced to point to the same snapshot
    as B.
-   Otherwise, a **three-way merge** takes place: git will attempt to
    create a new snapshot X automagically based on the snapshots of H,
    B, and their nearest common ancestor. X will have the snapshots of H
    and B as parents, and then H will be advanced to point to X.
    -   If a merge conflict occurs, it can be manually resolved by
        editing the conflicted files and then using the `git add`
        operation on them; or, `git mergetool` will provide a graphical
        alternative. Then commit to finalize the merge.

#### Merging a specified branch B into the current branch  
  `git merge B`

It is recommended to create branches frequently—including "topic
branches" representing specific changes to the codebase (such as
individual bug fixes). These can be worked on in parallel and then
merged back into the master branch when they are done.

#### "Moving" the sequence of commits in the current branch so they depend on the latest version of the base branch B ([details][])  
  `git rebase B`

  [details]: http://schacon.github.com/git/user-manual.html#using-git-rebase

