---
layout: post
title: "The minimum basics of Git Version Control System"
excerpt: "Basic concepts and commands of Git VCS"
date: 2018-06-28
tags: [tech, git, vcs, scm, versioncontrol, dvcs]
categories: articles
image:
  feature: posts/git/gitflow_4.png
  credit: oskari.org
  creditlink: http://oskari.org/documentation/development/oskari-git-process
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

Git is an open source, distributed Version Control System. Originally created in 2006 to maintain Linux kernel.

Git gives every developer a full copy of the repository - not just the files, but all capabilities of the repository. So, effectively everyone has a full repository locally, with full history. Benefits

* Faster commands, as they run locally. It's also possible to work offline
* Stability. Many copies, no single point of failure
* Isolated environments - easier to experiment and test. Also, one can have team, feature or bug specific repository
* Effective merging (feature, not benefit) - because of this nature, it inherently has many branches. With a single local commit, local branch goes out of sync. With a push to remote master, all locals go out of sync. This requires, and Git provides, a strong merging system.

Each Git repository contains 4 components:

* The working directory
* The staging area
* Committed history
* Development branches

```bash
#see configured username. Provide within quotes to set. Also user.email
git config --global user.name
# --global flag does for all repositories, sets in ~/.gitconfig, omit for local repo
```

Setting aliases or shortcuts for commands, e.g. `st` for `status`

```bash
git config --global alias.st status #run as git st
git config --global alias.ch checkout #git ch
```

## Basic Git Commands

Git commands can be run in `Git Bash` that comes with default installation. Standard GUIs like `TortoiseGit`, `SourceTree` and others already have them built into the user interface.

```bash
# initialize git repo with .git directory at project root
git init <path>
#leave <path> as empty to run on current directory
# clone a remote repo (with https) example
git clone https://github.com/Readify/Neo4jClient.git
# clone with ssh (needs SSH configured)
git clone ssh://<user>@<host>/path/to/repo.git
```

Git maintains *snapshot* of the project for safe versioning. This is different from SVN's way of maintaining file changes. In Git, each commit has full files and complete version of each file that is part of it, i.e. a full snapshot.

#### Basic Git components & commit workflow

Working directory -> Staging area -> Commit to local repository -> Push to origin

* Working directory - is basically the directory where the files are locally, as a result of `init` or `clone`. This is just like a local directory where you can make changes to any files, additionally you can execute Git commands. Git stores the metadata and object database in this area, and it is also called _"working tree"_
* Staging area - is the intermediary between working directory & committed history. This lets you organize files, before actually committing the changes. Also called the _"index"_
* Commit to local repository - changes from staging area can be committed in single or multiple commits
* Push to origin - 

###### Staging

> The staging area is a file, generally contained in your Git directory, that stores information about what will go into your next commit. Its technical name in Git parlance is the “index”, but the phrase “staging area” works just as well. - from [Pro Git](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)

Files need to be added to the _staging area_ or _index_ to make them ready to commit. Files that are not staged (and not committed) are called `untracked` files.

```bash
# to add to staging, to track/index
git add <file> # use . for all files
# to remove from staging
git rm --cached <file>
# to remove from staging & working directory (delete)
git rm <file>
# check status of working directory files
git status #files come as to be committed, untracked etc.
```

![Image](/images/posts/git/git_status.png)

**Note:** Staging with `git add` actually stages changes in file, NOT the file as an entity. That means, if you stage a file, then make some changes in the file and then `git commit`, only the changes made before `git add` will be committed. To add the later changes, you have to `git add` again before commit (this can be forced though, with a `-a` switch in `git commit`. This adds unstaged changes, but it does not add untracked files).
{: .notice--info}

So, a file in Git basically has 4 stages:

1. Untracked - the file has not been made of the Git repository yet
2. Staged - ready to be committed in next commit, with current changes
3. Modified - there are changes made since last commit (staged or not)
4. Committed - safely added to local repository with history

In standard icon set, untracked/modified files (not added to stage) appears with a `?`. Staged ones appear with a `+` and the committed ones appear with a green tick.

Use following commands to see the changes in files.

```bash
git diff #unstaged changes, excluding untracked ones
git diff --cached #for staged changes to be committed
git log #to see committed history from current branch
```

![Image](/images/posts/git/git_diff.png)

Git also allows _partial/selective staging_, i.e. stage only part of file(s) by using the following `interactive` or `patch` commands. Git breaks file changes to chunks what it calls as **`hunks`** of file. Then you can selectively stage/commit changes. See more details [here](https://git-scm.com/docs/git-add).

```bash
git add --patch <filename> #or -p for short
#internally calls
git add --interactive
```

The `Git Bash` CLI and some GUI tools like [SourceTree](https://www.sourcetreeapp.com/) supports this feature.

###### Commit to local repository

Committing a staged changeset actually puts it into the local repository with commit history. The commit history has the following main components

* The snapshot of changed files, from the staging area
* A SHA-1 hash of all the contents, which also serves as the `Commit ID`
* The user information, name & email
* A timestamp
* The commit message

![Image](/images/posts/git/committed_history.png)

```java
[Commit #1] <-[Commit #2] <-[Commit #3] <-[Commit #4 | HEAD]
```

To commit all staged changes, simply run `git commit`. This will open an editor for specifying the commit message, you cannot commit without a message. To commit with a message, just use the `-m` flag. Use `git log` to see committed history from current branch. Git log also support [formatting options](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History).

```bash
git commit -m "cool & meaningful commit message"
git commit -a -m "include unstaged changes as well!"
```

###### Tagging

A commit can also be **tagged**. Tagging simply means marking a specific commit so that it can be identified easily, like for a specific release of the product. Tags are generally annotated with `-a` key, and a message is provided. Annotated tagged commits are saved with user details, timestamp, message and the actual commit.

```bash
git tag #show all tags in alphabetical order
git tag v1.2 -a -m "Release 1.2" #tag the last commit
git show v1.2 #show details of v1.2 tagged commit
```

## Git undo options

In any real project, it's frequently required to undo some changes that are already done. The changes might have been made to local (working tree) files, staged changes or the committed ones. There are many options in Git for all these possibilities.

Removing unwanted changes from working directory, and stage. The `git reset` undoes any staged changes, that are not committed. This effectively undoes the staging since last commit.

**Note:** Doing a `git reset` with `--hard HEAD` takes everything in the tracked files to the last committed version, effectively undoing all uncommitted changes. Be very **careful** while doing this, as this will remove all the local changes from the files, and those changes will be lost. Be sure you want to do this, before you actually run the command. For the same reason, be careful with `git clean -f` which forcefully removes all untracked files.
{: .notice--danger}

```bash
git reset #unstage changes
git reset --hard HEAD #reset to last commit
git clean –f #remove any untracked files
git checkout HEAD <file> #reset file to latest commit
git reset HEAD <file> #unstage a file
```

###### Undoing commits

Undoing changes done (committed) in local repository. This has 2 basic options. 

1. Remove the commit from commit history. It can be pretty risky if someone has already started working/branched on the commit. Fine for local only commits. The command for this is `git reset HEAD~n` where `n` is an integer that states, go back n-commits before the HEAD. (Conceptually it's like resetting the HEAD in a linked list, effectively removing last one or more heads).
2. Create a new commit that reverses all changes done in the target commit. Git handles the complexity of figuring out how to create a new change to undo the target changes, so it works fine and safe. For this, simply use `git revert <commit-id>`, where the commit-id is the full or part of SHA-1 ID of the target commit. In public commits, always to this over the other option.

###### Amending commit

The amend option `git commit -amend` simply replaces the last commit and amends that with new changes that you introduced to stage after the last commit. So, effectively it'll create a new commit and replace the last one, with the same commit message, with that commit changes and the new ones.

This is helpful if something was missed in the commit. But be careful, as it'll replace the commit history as well.

## Branches



branches are super light-weight. Git simply maintains the HEAD commit-id for the branch.

```bash
git branch #see all branches, current active one is starred*
git branch cool-feature #create a new branch 'cool-feature'
git checkout cool-feature #switch to branch
#now staging & commit will happen to cool-feature branch
#now switch to another branch before deleting
git branch -d cool-feature #delete merged branch, history preserved
git branch -D cool-feature #force delete un-merged branch
```



```java
[Commit #1] <-[Commit #2] <-[Commit #3] <-[Commit #5 | master HEAD]
                                ^
                                |
                            [Commit #4] <- [Commit #6 | cool-feature HEAD]
```

Git checkout can also be run with specific tag or commit-id. But this has some effects, it'll basically checkout specific commit, rather than a branch. This state is known as **detached-HEAD**, which can be difficult to merge back. The standard way to [fix](https://stackoverflow.com/questions/10228760/fix-a-git-detached-head) this is to create a new branch and commit the changes.

###### Merging branches

```bash
#commit changes to feature-branch
git checkout master
git merge feature-branch #merge feature to master
```

![Image](/images/posts/git/git_merge.png)

###### Merge conflicts

```bash
$ git merge feature2
Auto-merging feature2_2.txt
CONFLICT (content): Merge conflict in feature2_2.txt
Automatic merge failed; fix conflicts and then commit the result.
```

```
<<<<<<< HEAD
1st line in master branch
=======
1st line from feature2 branch
>>>>>>> feature2
```

```bash
git add feature2_2.txt
git commit -m "Resolving merge conflicts"
```

###### Rebasing

```bash
git checkout feature2
git rebase master
# OR for interactive options
git rebase –i master
```

<figure>
	<a href="/images/posts/git/rebase-succinctly.png">
        <img src="/images/posts/git/rebase-succinctly.png" alt="rebase" title="rebase">
    </a>
	<figcaption>Source <a href="https://www.syncfusion.com/ebooks/git">syncfusion.com</a></figcaption>
</figure>

_pick_ & _squash_ [rebase options](https://git-scm.com/docs/git-rebase)

will create new commits with same contents and effectively rewrite history.

General rule: _Never rebase public and/or shared branches_.

## Remote repositories

```bash
git remote add <name> <path-to-remote-repo> #syntax
#example witha github repo as remote, named as "origin"
git remote add origin https://github.com/chakrabar/GitTest.git
git remote #see all remote repositories
git remote -v #same, verbose
git remote rm origin #remove remote named "origin"
```

Git `fetch` is safe, it downloads the remote content but does NOT make any change to the local branch.

```bash
# omit branch name to fetch ALL remote branches
git fetch origin <remote-branch>
git checkout remote-branch #checkout remote branch
git branch -r #show all branches including romtes
```

```bash
git checkout local-branch
git fetch origin
git merge origin/remote-branch
git rebase origin/remote-branch
```

###### Pull & Push

Git `pull` is a combination of `git fetch` and `git merge`. It downloads the remote contents and then merges it into the local branch.

It is generally considered as NOT SAFE as it'll automatically change local branch after fetching the remote branch.

Git `push` sends the local branch to the remote repository. It uploads the local data to the remote, and creates a _local branch_ in the remote repository with the same name as the local-branch from where it was pushed (if a new remote-branch name is used).

```bash
# PULL
git pull origin/remote-branch
# is same as...
git fetch origin remote-branch #download remote branch
git merge origin/remote-branch #merge remote to local

# PUSH
# first, checkout the correct local branch
git push remote-repo remote-branch
# in it's simple form
git push origin master
```

## GitHub

43-75

at least 68-75

#### Further reading

* [Git reference docs](https://git-scm.com/docs)
* [Pro Git - online book](https://git-scm.com/book/en/v2)
* [Git Succinctly ebook](https://www.syncfusion.com/ebooks/git)
* [How to undo almost anything with Git](https://blog.github.com/2015-06-08-how-to-undo-almost-anything-with-git/)
* [Git workflows](https://www.atlassian.com/git/tutorials/comparing-workflows#!workflow-gitflow)
* [Git cheat sheet](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf)
* Git vs SVN - [Why is Git better than Subversion?](https://stackoverflow.com/questions/871/why-is-git-better-than-subversion), [Subversion vs. Git: Myths and Facts](https://svnvsgit.com/), [GIT VS SVN](https://walty8.com/comparison-of-git-and-svn/)
* [Forking vs. Branching in GitHub](https://stackoverflow.com/questions/3611256/forking-vs-branching-in-github)