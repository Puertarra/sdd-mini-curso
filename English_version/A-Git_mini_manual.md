# Git Mini Manual

This annex gathers the Git commands the course uses, mostly from module 09 onward, with examples taken from ProductChat. It does not replace the Git documentation: it is the sheet you want to keep open while you work. If a command does not appear here, you probably do not need it for this course.

All commands require **Git 2.28 or later** (`git init -b` and `git switch` do not exist in older versions). Check with `git --version`.

## Git and GitHub

**Git** is a version control system: it records every change to a project on your computer and lets you go back to any point in its history. It works offline. **GitHub** is a web service that hosts Git repositories and adds collaboration tools: change review, *issues*, *pull requests*, *releases*. Git is the tool; GitHub is one of the places where what Git records is published.

| | Git | GitHub |
|---|---|---|
| What it is | A program | A web service |
| Where the repository lives | On your computer | On a remote server |
| Works offline | Yes | No |
| What we use it for | Recording ProductChat's history and separating the work by Spec | Publishing the project and sharing it |

```text
your computer (local repository)  ── git push ──▶  GitHub (remote repository)
                                  ◀── git pull ──
```

## Initial setup

You do this only once per computer. Git signs every commit with this information.

| Command | What for |
|---|---|
| `git --version` | Check that Git is installed and its version |
| `git config --global user.name "Ana Pérez"` | Name that appears on your commits |
| `git config --global user.email "ana.perez@example.com"` | Email associated with your commits |
| `git config --global init.defaultBranch main` | Make new repositories start on the `main` branch |

## Work areas and file states

Git works with three local areas and one remote. Each command moves changes from one area to another:

```text
 working folder ── git add ──▶ staging area ── git commit ──▶ local repository ── git push ──▶ GitHub
       ◀── git restore ──            ◀── git restore --staged ──                    ◀── git pull ──
```

The **working folder** is your files as you see them. The **staging area** is the selection of changes that will go into the next commit. The **local repository** stores the commit history. Separating staging from committing lets you review what is about to be recorded before you record it.

| State | What it means | How it moves to the next |
|---|---|---|
| Untracked | New file that Git does not track yet | `git add <file>` |
| Modified | Tracked file that changed since the last commit | `git add <file>` |
| Staged | Change selected for the next commit | `git commit` |
| Committed | Change saved in the history | Already recorded |

`git status` shows which state each file is in. For example, halfway through SPEC003:

```text
$ git status
On branch 003-catalog-filters
Changes not staged for commit:
        modified:   src/productchat/catalog.py
Untracked files:
        specs/003-catalog-filters/
```

## Create and save

| Command | What for | Example |
|---|---|---|
| `git init -b main` | Turn the current folder into a repository, with a `main` branch | `git init -b main` |
| `git clone <url>` | Copy a remote repository to your computer | `git clone https://github.com/user/productchat.git` |
| `git status` | See what changed and what is staged | `git status` |
| `git add <path>` | Stage a file or folder | `git add specs/003-catalog-filters/` |
| `git add .` | Stage everything that changed | `git add .` |
| `git diff` | See changes not yet staged | `git diff src/` |
| `git diff --cached` | See what is already staged, before the commit | `git diff --cached` |
| `git commit -m "<message>"` | Record what is staged in the history | `git commit -m "SPEC003: plan and tasks"` |

Review `git diff --cached` before every commit. It is the last moment you see exactly what you are about to record.

Avoid `git commit -am`: it stages and records the already-known files in one go, but **it skips new files**. In module 10 that would leave a patch file out of the history.

### What is not versioned

The `.gitignore` file lists what Git must ignore. In ProductChat:

```text
.venv/
__pycache__/
.pytest_cache/
~$*.xlsx
.specify/feature.json
```

`~$*.xlsx` are the temporary files Excel creates while the spreadsheet is open. `.specify/feature.json` stores which Spec is active in your directory, and it changes with every `/speckit-specify`: it is not project history (see module 11). The catalog **is** versioned: the acceptance criteria cite its SKUs.

## Look at the history

| Command | What for | Example |
|---|---|---|
| `git log --oneline` | Compact history, one commit per line | `git log --oneline` |
| `git log --oneline --graph --all` | History of all branches, as a graph | `git log --oneline --graph --all` |
| `git log -p <file>` | Every change a file went through | `git log -p specs/001-catalog-query/spec.md` |
| `git show <commit>` | What changed in a commit | `git show a1b2c3d` |
| `git show --stat <commit>` | Only the list of files in a commit | `git show --stat HEAD` |
| `git show <branch>:<path>` | Read a file from another branch without switching to it | `git show 005-csv-export:specs/005-csv-export/spec.md` |

```text
$ git log --oneline
e4f5a6b (HEAD -> 003-catalog-filters) SPEC003: implement and test filters
c3d4e5f SPEC003: plan and tasks
a1b2c3d SPEC003: specify and clarify filters
9f8e7d6 (main) ProductChat: base implementation with SDD artifacts
```

`HEAD` is the commit you are currently on. The first seven characters are the abbreviated commit identifier (*hash*), the one you use with `git show`.

## Branches

A branch is a separate line of work. In the course, each Spec is developed on its own branch and merged into `main` only once it is verified.

| Command | What for | Example |
|---|---|---|
| `git branch` | List the branches and see which one you are on | `git branch` |
| `git switch -c <branch>` | Create a branch and switch to it | `git switch -c 003-catalog-filters` |
| `git switch <branch>` | Switch to an existing branch | `git switch main` |
| `git merge <branch>` | Merge another branch into the current one | `git merge 003-catalog-filters` |
| `git branch -d <branch>` | Delete a branch that is already merged | `git branch -d 003-catalog-filters` |

```text
main     ──●──────────────────────●──   ← git merge 003-catalog-filters
            \                    /
003-…        ●──────●──────●────
           specify  plan  implement
```

To merge, first switch to the receiving branch (`git switch main`) and then merge the other one (`git merge …`). In older material you will see `git checkout` instead of `git switch`; they do the same thing, but `switch` is clearer.

### Conflicts

If two branches changed the same lines, Git does not choose: it stops the merge and marks the file like this:

```text
<<<<<<< HEAD
DEFAULT_SORT = "price"
=======
DEFAULT_SORT = "name"
>>>>>>> 005-csv-export
```

The top part is your branch; the bottom part, the one you are merging. To resolve it, edit the file until it contains the correct version, without the markers, and then run `git add <file>` and `git commit`. In SDD, you do not decide the correct version by looking at the code: the Spec decides it. Module 11 works through this case.

## Parallel work and deliveries

| Command | What for | Example |
|---|---|---|
| `git worktree add <folder> <branch>` | Open a branch in another folder, to work on two at once | `git worktree add ../ProductChat-csv 005-csv-export` |
| `git worktree list` | See the open folders | `git worktree list` |
| `git worktree remove <folder>` | Close a folder with no pending changes | `git worktree remove ../ProductChat-csv` |
| `git archive` | Export a copy of the files, without history | See module 12 |
| `git tag -a <version> -m "<message>"` | Mark a commit as a version | `git tag -a v1.0 -m "First release"` |

## Working with GitHub

| Command | What for | Example |
|---|---|---|
| `git remote add origin <url>` | Connect the local repository to one on GitHub | `git remote add origin https://github.com/user/productchat.git` |
| `git remote -v` | See which remote repository it is connected to | `git remote -v` |
| `git push -u origin main` | Push a branch for the first time | `git push -u origin main` |
| `git push` | Push the new commits | `git push` |
| `git push origin --tags` | Push the version tags | `git push origin --tags` |
| `git pull` | Fetch and apply the changes from the remote | `git pull` |
| `git fetch` | Fetch the changes from the remote without applying them | `git fetch origin` |

On the GitHub website you also work with *forks* (copies of a repository in your account), *pull requests* (change proposals that others review before merging them), *issues* (a record of problems and tasks) and *releases* (versions published from a tag, downloadable as a zip).

## Undo without fear

What to use depends on how far the change has gone:

| Situation | Command | What it does |
|---|---|---|
| You changed a file and want to go back to the last commit | `git restore <file>` | Discards your changes to that file |
| You staged something that should not go into the commit | `git restore --staged <file>` | Removes it from the staging area; your changes are still there |
| You committed, not pushed yet, and something is missing or the message is wrong | `git commit --amend` | Redoes the last commit |
| The commit is already on GitHub | `git revert <commit>` | Creates a new commit that undoes the previous one |

`git restore <file>` cannot be undone: what it discards is not kept anywhere.

Two commands you will see recommended online and that you should avoid while you are learning: `git reset --hard`, which deletes all unrecorded changes without warning, and `git push --force`, which rewrites the history on GitHub and can delete other people's work. `git revert` almost always solves the same problem without destroying anything.

## Course conventions

- **One branch per Spec**, with the same name as its folder in `specs/`: `003-catalog-filters`, `004-hide-out-of-stock`.
- **Commit messages in English**, with the Spec identifier as a prefix: `SPEC003: specify and clarify filters`. One commit per stage of the cycle, not one at the end.
- **Small, meaningful commits.** If the message needs the word *and* to join two unrelated things, they were probably two commits.
- **Nothing is merged into `main` without verification.** First the tests and the acceptance criteria; then the merge.

## Quick reference

| Command | What for | Module |
|---|---|---|
| `git init -b main` | Create the repository | 09, 12 |
| `git status` | See the state | 09 |
| `git add` | Stage changes | 09–12 |
| `git diff --cached` | Review what is staged | 09 |
| `git commit -m` | Record | 09–12 |
| `git switch -c` | Create a branch | 09–11 |
| `git switch` | Switch branches | 09–11 |
| `git merge` | Merge a branch | 09–11 |
| `git log --oneline` | See the history | 09 |
| `git log -p` | History of a file | 09 |
| `git show` | See a commit or a file from another branch | 09, 11 |
| `git worktree` | Work on two branches at once | 11 |
| `git archive` | Export without history | 12 |
| `git restore` | Undo local changes | Annex |
| `git revert` | Undo a published commit | Annex |
| `git push` / `git pull` | Sync with GitHub | Annex |

To go further: Chacon, S. and Straub, B. (2014). [*Pro Git*](https://git-scm.com/book/en/v2) (2nd ed.). Apress. Freely available online.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
