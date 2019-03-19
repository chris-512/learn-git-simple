# Learn Git in one page

## Just to know that:

- `HEAD`: A reference to the current commit. It is a commit that you have checked in the working directory. It is like a moving pointer. Sometimes it means the current branch, sometimes it doesn't. So HEAD is NOT a synonym for "current branch" everywhere already. HEAD means <b> "current" anywhere </b> in git, but it does not necessarily mean "current branch" (i.e. detached HEAD) But it almost always means the "current commit"
- `@`: A shortcut for `HEAD` since Git 1.8.5
- `ORIG_HEAD`: Previous state of `HEAD`, set by commands that have possibly dangerous behaviour, to be easy to revert them. It is less useful now that Git has reflog: HEAD@{1} is roughly equivalent to ORIG_HEAD (HEAD@{1} is always last value of HEAD, ORIGH_HEAD is last value of HEAD before dangerous operation)

## Some useful Git commands

Show the current HEAD

```
git show HEAD
git show @ # from git 1.8.4 (July 2013)
```

- Revert "Add new @ shortcut for HEAD"

  This reverts commit cdfd948, as it does not just apply to "@" (and forms with modifiers like @{u} applied to it), but also affects e.g. "refs/heads/@/foo", which it shouldn't.

  The basic idea of giving a short-hand might be good, and the topic can be retried later, but let's revert to avoid affecting existing use cases for now for the upcoming release.

Rollback what I've done. Reflog is a vehicle to go back in time and time machines have interesting interaction with the notion of "current".

```
git reflog
git reset --hard <hash>
git reset --hard HEAD@{5.minutes.ago}
```

Resetting hard to it brings your index file and the working tree back to that state, and resets the tip of the branch to that commit.

```
git reset --hard ORIG_HEAD
```

After inspecting the result of the merge, you may find that the change in the other branch is unsatisfactory. Running "git reset --hard ORIG_HEAD" will let you go back to where you were, but it will discard your local changes, which you do not want. "git reset --merge" keeps your local changes.

```
git reset --merge ORIG_HEAD
```

- pull or merge always leaves the original tip of the current branch in ORIG_HEAD. merge always sets '.git/ORIG_HEAD' to the original state of HEAD so a problematic merge can be removed by using 'git reset ORIG_HEAD'.

Bring commits with hash tags on top of your HEAD

```
git cherry-pick <hash id>
git cherry-pick <hash id-1> <hash id-2> ...
```

Move your HEAD to another branch

```
git checkout <branch>
```

Move your HEAD to commit with hash id

```
git checkout <hash>
```

Discard your change in a file

```
git checkout -- <file>
```

Unstage a file from your HEAD

```
git reset HEAD <file>
```

Amend/Rearrange/Omit your past commits using rebase interactive mode

```
git rebase -i HEAD~4
git rebase -i HEAD^^^^
git rebase -i @~4
git commit --amend
git rebase --continue
git rebase --abort
```

Rebase the first commit (normally you can't if you try)

```
git rebase -i --root
```

Reference: https://stackoverflow.com/questions/22992543/how-do-i-git-rebase-the-first-commit/22992544

Amend the author's name in HEAD

```
git commit --amend --author="baverzl <baverzl.doer@gmail.com>"
```

Undo a commit and redo

```
git commit -m "Something terribly misguided"
git reset HEAD~
git add <files>
git commit -c ORIG_HEAD
```

Reference: https://stackoverflow.com/questions/927358/how-do-i-undo-the-most-recent-commits-in-git

Amend your commit without any change in commit message

```
git commit --amend --no-edit
```

Using -f option to designate master branch to HEAD~3

```
git branch -f master HEAD~3
```

Tag your commit with the version number

```
git tag v1.0.0 <hash id>
```

Describe how much your provided commit has been proceeded from the most recent tag

```
git describe <hash id> # output: <tag>_<numCommits>_g<hash>
```

Locate your HEAD to the 1st parent, one who actually performed a merge to create you.

```
git checkout HEAD^
```

Locate your HEAD to the 2nd parent

```
git checkout HEAD^2
```

Locate your HEAD to the 1st parent

```
git checkout HEAD~
```

Locate your HEAD to the parent of the 1st parent

```
git checkout HEAD~2
```

- ^ is to select one of your parents, and ~ is to move your HEAD to parental commits<br>

Do previous things with a single command

```
git checkout HEAD~^2~
```

Examples with referencing

```
git branch bugWork HEAD~^2~ # Name it with a new branch bugWork
git checkout HEAD~^2~ -b bugWork # Name it with a new branch bugWork and move your HEAD to it
```

Reapply commits from branch2 on top of branch1

```
git rebase <branch1> <branch2>
```

git pull is equivalent to:

```
git fetch
git merge o/master
```

git pull --rebase is equivalent to:

```
git fetch
git rebase o/master
```

### Pros and cons of `merge`?

- `Pros`: You could notify that you have reflected changes from the remote repository and remote branches have become your parents. Plus, some developers prefer to merge because it perserves the commit history.
- `Cons`: Performing a merge on the local branch is usually followed by making an additional merge commit. This may look unnecessary to someone. Also, it makes you hard to track where your commit came from.

### Pros and cons of `rebase`?

- `Pros`: Your commit trees are nicely arranged in a single stream. If you want your history look tidy, use rebase.
- `Cons`: Performing rebase modifies the structure of commit history. For example, commit C1 can be rebased to past commit C3. After rebase, C1' is now ahead of C3. But, in fact, C1 is already completed before C3. rebase makes it hard for someone to fidn the order of commit history.

<br>

## Advanced remote git commands

### Your local branch tracks another remote branch

```
git checkout -b totallyNotMaster o/master
git branch -u o/master foo # Your local foo now tracks o/master
git branch -u o/master # If you are already in foo, just omit foo
```

## git push with arguments

When you want to push <source> to origin's <destination>

```
git push origin <source>:<destination>
git push origin HEAD^:master
```

If newBranch is not available remotely, it creates a new one.

```
git push origin master:newBranch
```

## git fetch with arguments

When you want to fetch origin's <source> to your local <destination>

```
git fetch origin <source>:<destination>
```

Fetch origin's foo and update your local o/foo

```
git fetch origin foo
```

Fetch origin's foo~1 to your local branch bar. If bar is not present, it will make a branch named bar and perfom a fetch.

```
git fetch origin foo~1:bar
```

git fetch without arguments will fetch all the remote changes to your local remote branches (branches starting with o/)

```
git fetch
```

### What's difference between git push and fetch?

- git push's `<source>` indicates your local commits
- git push's `<destination>` indicates (origin's) remote branch
- git pull's `<source>` indicates (origin's) remote commits
- git pull's `<destination>` indicates your local branch

### Create/Delete branches with push and fetch

Pushing a null to origin's foo means to delete the remote foo branch

```
git push origin :foo
```

Fetching "nothing" to local bar means to create one.

```
git fetch origin :bar
```

### git pull with arguments

git fetch origin foo; git merge o/foo

```
git pull origin foo
```

git fetch origin bar~1:bugFix; git merge bugFix

```
git pull origin bar~1:bugFix
```

Fetch origin's master branch to foo and merge it to which you are checked out. (HEAD)

```
git pull origin master:foo
```

<br><br>

## TODO Lists

- git bisect
