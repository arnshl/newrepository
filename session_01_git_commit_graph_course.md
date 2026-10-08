# Session 01 · Follow-Along Guide: Git as a Commit Graph

**Course:** Python Environments & Engineering Workflows (MSA-DATI07-01) · MSc 1 · 2h
**How to use this guide:** the teacher and the students run the **same commands, in the same order**,
on their own laptops. The teacher projects their terminal. Every step has the same four parts:

- **Predict**: say out loud what you expect before pressing Enter (the course rule).
- **Run**: the command(s).
- **You should see**: the real output, captured by running this guide end to end.
  Your commit hashes (`0f9164b`...) will be different: that is normal, everything else should match.
- **Why**: what just happened, in one or two sentences.

> **On the board** boxes say what to draw. **If it goes wrong** boxes cover the classic accidents.

**End of session deliverable:** a repository whose `git log --oneline --graph --all` shows about ten
atomic commits, one resolved merge conflict and one revert, and no `.env` anywhere in history.

| Part | Topic | Time |
|---|---|---|
| 0 | Setup | 0:00 - 0:15 |
| 1 | Three areas, first commits | 0:15 - 0:40 |
| 2 | Staging in detail, the two diffs | 0:40 - 0:55 |
| 3 | Branches and fast-forward | 0:55 - 1:05 |
| | Break | 1:05 - 1:15 |
| 4 | Diverging branches and a merge conflict | 1:15 - 1:40 |
| 5 | Undoing: restore, reset, revert, reflog | 1:40 - 1:55 |
| 6 | Check the deliverable | 1:55 - 2:00 |

Glossary and troubleshooting are at the end. The starter code is in Appendix A.

---

## Part 0 · Setup (15 min)

### 0.1 Check Git is installed

```bash
git --version
```

You should see `git version 2.x.x`. Any 2.30 or newer is fine.

> **If it goes wrong:** `command not found`. macOS: run `xcode-select --install`. Windows: install
> Git for Windows and use **Git Bash** for the whole course, not PowerShell.

### 0.2 Tell Git who you are (once per machine)

```bash
git config --global user.name "Firstname Lastname"
git config --global user.email "you@example.org"      # the email of your GitHub account
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.editor "code --wait"         # VS Code opens for commit messages
```

Check:

```bash
git config --global --list
```

**Why:** every commit records an author. Without `user.name` / `user.email`, Git refuses to commit.
`core.editor` avoids being trapped in Vim when Git needs a message (merge, revert).

> **If it goes wrong:** you are in Vim anyway (a screen full of `~`). Press `Esc`, type `:wq`, press
> Enter. That saves and quits.

### 0.3 Create the project folder

```bash
mkdir audience-report
cd audience-report
```

Put the two starter files inside (your teacher shares them, or copy them from Appendix A):

```
audience-report/
  audience_report.py
  data/audience_sample.csv
```

Check the script runs (on macOS, type `python3` wherever this guide says `python`):

```bash
python audience_report.py data/audience_sample.csv
```

You should see:

```
section       views  avg read (s)
sport         11200          37.2
politics       4800         101.9
economy        2750         156.5
culture        1500         126.7
```

> **If it goes wrong:** do **not** create this folder inside another Git repository (for instance
> inside the course materials you cloned). Check with `git rev-parse --show-toplevel`: before
> `git init`, it must answer `fatal: not a git repository`.

---

## Part 1 · The three areas and the first commits (25 min)

> **On the board:** draw three boxes from left to right and keep them all session.
>
> ```
>  WORKING DIRECTORY        STAGING AREA (index)         REPOSITORY
>  files on your disk  --add-->  next commit being   --commit-->  history (commits)
>                                prepared
> ```
>
> "Index" and "staging area" are two names for the same thing.

### 1.1 Turn the folder into a repository

**Predict:** what does `git init` create?

```bash
git init
git status
```

You should see:

```
Initialized empty Git repository in .../audience-report/.git/

On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	audience_report.py
	data/

nothing added to commit but untracked files present (use "git add" to track)
```

**Why:** `git init` creates a hidden `.git/` folder: that **is** the repository (all history lives
there). Your files exist on disk but Git does not track them yet: they are **untracked**.

The short form, which we use from now on:

```bash
git status --short
```

```
?? audience_report.py
?? data/
```

`??` = untracked.

### 1.2 First commit: the .gitignore, before anything else

Some files must **never** enter history. The `.env` file holds passwords and API tokens: once
committed and pushed, a secret stays in history forever, on every clone. So the very first commit
of a project is the list of files Git must ignore.

Create `.gitignore` with this content:

```
.env
.venv/
__pycache__/
.DS_Store
```

**Predict:** what does `git status --short` show now?

```bash
git status --short
```

```
?? .gitignore
?? audience_report.py
?? data/
```

Now stage only the `.gitignore`:

```bash
git add .gitignore
git status --short
```

```
A  .gitignore
?? audience_report.py
?? data/
```

**Why:** `A` in the **left** column = added to the staging area. `git add` copies the file's
current content into the index. Nothing is in history yet.

> **On the board:** move `.gitignore` from the first box to the second.

Commit:

```bash
git commit -m "chore: add .gitignore"
git log --oneline
```

```
[main (root-commit) 0f9164b] chore: add .gitignore
 1 file changed, 4 insertions(+)
 create mode 100644 .gitignore

0f9164b chore: add .gitignore
```

**Why:** `git commit` turns the staging area into a **commit**: a snapshot of the whole project,
with an author, a date, a message, and a pointer to its parent (none here: it is the root commit).
`0f9164b` is the start of its unique identifier (hash).

### 1.3 Commit messages: Conventional Commits

From now on, every message follows `<type>: <short summary in the imperative>`.

| Type | For |
|---|---|
| `feat` | a new capability |
| `fix` | a bug fix |
| `docs` | documentation only |
| `style` | formatting, naming, no behaviour change |
| `refactor` | restructuring, no behaviour change |
| `test` | tests |
| `chore` | config, tooling, dependencies |

`update`, `fix stuff`, `wip` are not acceptable messages: in six months nobody knows what they
contain.

### 1.4 Second commit: the script and its data

```bash
git add audience_report.py data/audience_sample.csv
git status --short
git commit -m "feat: add daily audience report script"
```

```
A  audience_report.py
A  data/audience_sample.csv

[main fe2c345] feat: add daily audience report script
 2 files changed, 64 insertions(+)
```

### 1.5 Prove that .env is ignored

Create a fake secret:

```bash
echo "AUDIENCE_API_TOKEN=s3cr3t" > .env
```

**Predict:** does `.env` appear in `git status --short`?

```bash
git status --short
git check-ignore -v .env
```

```

.gitignore:1:.env	.env
```

**Why:** `status` shows nothing: `.env` is invisible to Git. `check-ignore -v` tells you which line
of which file ignores it. `git add -A` would not pick it up either.

Teammates still need to know which variables exist, so we commit a **template** with fake values:

```bash
echo "AUDIENCE_API_TOKEN=change-me" > .env.example
git add .env.example
git commit -m "chore: add .env.example"
```

> **If it goes wrong:** `.env` was committed **before** being added to `.gitignore`. Adding it to
> `.gitignore` afterwards changes nothing: `.gitignore` only affects untracked files. Fix:
> `git rm --cached .env` then commit. And if the secret was real: revoke it, because it is still in
> history.

---

## Part 2 · The staging area in detail (15 min)

### 2.1 A file can have two versions at once

Create `README.md`:

```
# Audience report

Daily views and reading time per section.
```

```bash
git add README.md
git status --short
```

```
A  README.md
```

Now **add a line** at the end of `README.md`, without running `git add` again:

```

Usage: python audience_report.py data/audience_sample.csv
```

**Predict:** what does `git status --short` show?

```bash
git status --short
```

```
AM README.md
```

**Why:** two columns. Left `A`: the index contains the README (first version). Right `M`: the file
on disk was modified **after** being staged. If you committed now, only the first version would go
in. (For a file that already existed in history, the same situation shows `MM`.)

> **On the board:** draw README in **both** the working directory and the staging box, with two
> different contents.

### 2.2 The two diffs

```bash
git diff            # working directory vs staging area: what is NOT staged yet
git diff --staged   # staging area vs last commit: what WILL be committed
```

`git diff` shows only the usage line:

```diff
@@ -1,3 +1,5 @@
 # Audience report

 Daily views and reading time per section.
+
+Usage: python audience_report.py data/audience_sample.csv
```

`git diff --staged` shows the first three lines (the new file as staged).

**Why:** each `@@ ... @@` block is a **hunk**: a group of consecutive changed lines with a few
unchanged lines of context around them. `-1,3 +1,5` reads "from line 1, 3 lines before, 5 after".

Stage the new version and commit:

```bash
git add README.md
git commit -m "docs: add README with usage"
git log --oneline
```

```
882e7ec docs: add README with usage
230d366 chore: add .env.example
fe2c345 feat: add daily audience report script
0f9164b chore: add .gitignore
```

**Atomic commits:** each of these commits does **one** thing, describable in one line without
"and", and the project works at every one of them. That is what makes history readable and each
commit revertable on its own.

> Bonus for fast students: `git add -p <file>` stages a file **hunk by hunk** (answer `y` or `n` for
> each), to split one messy editing session into two clean commits.

---

## Part 3 · Branches and the fast-forward merge (10 min)

> **On the board:** redraw history as a chain of circles, newest on top, each pointing to its
> parent. Write `main` on a sticky label next to the top circle, and `HEAD` pointing at `main`.
>
> - A **commit** is a snapshot + a pointer to its parent.
> - A **branch** is just a **label** pointing at one commit. Creating one copies nothing.
> - **HEAD** is "where I am": normally it points at a branch.

### 3.1 Create a branch and commit on it

```bash
git branch
git switch -c docs/license-note
```

```
* main

Switched to a new branch 'docs/license-note'
```

Add this at the end of `README.md`:

```

License: internal use only.
```

```bash
git commit -am "docs: add license note"
git log --oneline --graph --all
```

```
* f1481db docs: add license note
* 882e7ec docs: add README with usage
* 230d366 chore: add .env.example
...
```

**Why:** `-a` stages every already-tracked modified file, then commits (it never adds untracked
files). The new commit was made on `docs/license-note`: that label moved forward, `main` did not.

### 3.2 Switch back and look at the files

```bash
git switch main
tail -2 README.md
```

```

Usage: python audience_report.py data/audience_sample.csv
```

**Why:** the license line is gone from the file on disk. Switching branch moves HEAD **and**
rewrites your working directory to match that commit. Nothing is lost: it lives on the other branch.

### 3.3 Merge: fast-forward

**Predict:** `main` has not moved since the branch was created. What will the merge do?

```bash
git merge docs/license-note
```

```
Updating 882e7ec..f1481db
Fast-forward
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

**Why:** **fast-forward**: there was nothing to combine, so Git just slid the `main` label forward
to the branch's commit. No new commit was created. The graph is still a straight line.

Delete the label we no longer need (the commit stays, `main` points at it):

```bash
git branch -d docs/license-note
```

---

## Break (10 min)

---

## Part 4 · Diverging branches and a merge conflict (25 min)

We now make **both** sides move, each changing the report's header line in a different way.

### 4.1 On a branch: show reading time in minutes

```bash
git switch -c feat/minutes
```

In `audience_report.py`, in `print_report`, change these three lines (the `for` line between them
does not change):

```python
# before
    print(f"{'section':<10} {'views':>8} {'avg read (s)':>13}")
        avg_txt = f"{avg:.1f}" if avg is not None else "n/a"
        print(f"{section:<10} {total:>8} {avg_txt:>13}")

# after
    print(f"{'section':<10} {'views':>8} {'avg read (min)':>15}")
        avg_txt = f"{avg / 60:.1f}" if avg is not None else "n/a"
        print(f"{section:<10} {total:>8} {avg_txt:>15}")
```

Check with `git diff`, then commit:

```bash
git diff
git commit -am "feat: show average read time in minutes"
```

### 4.2 Back on main: rename the views column

```bash
git switch main
```

In `print_report`, change two lines (the lines between them do not change):

```python
# before
    print(f"{'section':<10} {'views':>8} {'avg read (s)':>13}")
        print(f"{section:<10} {total:>8} {avg_txt:>13}")

# after
    print(f"{'section':<10} {'total views':>12} {'avg read (s)':>13}")
        print(f"{section:<10} {total:>12} {avg_txt:>13}")
```

```bash
git commit -am "style: rename views column to total views"
git log --oneline --graph --all
```

```
* 5afc440 style: rename views column to total views
| * 736de58 feat: show average read time in minutes
|/
* f1481db docs: add license note
* 882e7ec docs: add README with usage
...
```

> **On the board:** the chain now forks at `f1481db`: `main` on one side, `feat/minutes` on the
> other. `f1481db` is their **common ancestor**.

### 4.3 Merge, and meet the conflict

**Predict:** fast-forward or not? Conflict or not?

```bash
git merge feat/minutes
```

```
Auto-merging audience_report.py
CONFLICT (content): Merge conflict in audience_report.py
Automatic merge failed; fix conflicts and then commit the result.
```

**Why:** both sides moved, so Git does a **three-way merge**: it compares each side with the
common ancestor. Changes on different lines are combined automatically. Here both sides changed
the **same** lines, so Git cannot choose and stops in the middle of the merge.

```bash
git status
```

```
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
	both modified:   audience_report.py
```

### 4.4 Read the conflict

Open `audience_report.py`. You will find:

```
<<<<<<< HEAD
    print(f"{'section':<10} {'total views':>12} {'avg read (s)':>13}")
    for section, (total, avg) in sorted(report.items(), key=lambda kv: -kv[1][0]):
        avg_txt = f"{avg:.1f}" if avg is not None else "n/a"
        print(f"{section:<10} {total:>12} {avg_txt:>13}")
=======
    print(f"{'section':<10} {'views':>8} {'avg read (min)':>15}")
    for section, (total, avg) in sorted(report.items(), key=lambda kv: -kv[1][0]):
        avg_txt = f"{avg / 60:.1f}" if avg is not None else "n/a"
        print(f"{section:<10} {total:>8} {avg_txt:>15}")
>>>>>>> feat/minutes
```

- Between `<<<<<<< HEAD` and `=======`: **our** side (main, where we are).
- Between `=======` and `>>>>>>> feat/minutes`: **their** side (the branch we merge in).

**Why four lines and not just the header?** The changes on each side touch lines that are
**next to each other**, so Git groups them into a single hunk, and the conflict covers the whole
hunk, including the `for` line nobody changed. If you only fix the header line and delete the
rest, you delete the loop. Hence the rule below.

### 4.5 Resolve: keep both intentions

Main wanted "total views", the branch wanted minutes. We want **both**. Replace the whole block
(markers included) with:

```python
    print(f"{'section':<10} {'total views':>12} {'avg read (min)':>15}")
    for section, (total, avg) in sorted(report.items(), key=lambda kv: -kv[1][0]):
        avg_txt = f"{avg / 60:.1f}" if avg is not None else "n/a"
        print(f"{section:<10} {total:>12} {avg_txt:>15}")
```

**Rule: a resolved file that does not run is not resolved.** Check there is no marker left, then
run:

```bash
grep -n "<<<<<<<\|=======\|>>>>>>>" audience_report.py
python audience_report.py data/audience_sample.csv
```

The `grep` prints nothing, and the script prints:

```
section     total views  avg read (min)
sport             11200             0.6
politics           4800             1.7
economy            2750             2.6
culture            1500             2.1
```

Mark the conflict as resolved by staging the file, then finish the merge:

```bash
git add audience_report.py
git commit --no-edit
git log --oneline --graph --all
```

```
*   0b8f703 Merge branch 'feat/minutes'
|\
| * 736de58 feat: show average read time in minutes
* | 5afc440 style: rename views column to total views
|/
* f1481db docs: add license note
...
```

**Why:** a **merge commit** has **two parents**: the two lines joining in the graph. `--no-edit`
keeps the message Git prepared. The branch can now go:

```bash
git branch -d feat/minutes
```

> **If it goes wrong:** lost in the middle of a merge? `git merge --abort` puts everything back as
> it was before `git merge`, and you can start again.

---

## Part 5 · Undoing things (15 min)

One question decides which command to use: **has it been shared (pushed) yet?**

| Command | Acts on | Safe after a push? |
|---|---|---|
| `git restore <file>` | discards edits in the working directory | yes (nothing committed) |
| `git restore --staged <file>` | unstages, keeps the edit on disk | yes |
| `git reset --soft HEAD~1` | moves the branch back one commit, keeps the changes staged | **no** |
| `git reset --hard HEAD~1` | moves the branch back and **deletes** the changes | **no** |
| `git revert <commit>` | adds a **new** commit that undoes an old one | **yes** |

`HEAD~1` means "the parent of where I am". `HEAD~2`, its grandparent, and so on.

### 5.1 Throw away an edit: restore

```bash
echo "# TODO: rewrite everything" >> audience_report.py
git status --short
git restore audience_report.py
git status --short
```

```
 M audience_report.py

```

**Why:** the right-column `M` (modified, not staged) disappears. `git restore <file>` copies the
version **from the staging area** back onto the disk. The edit is gone for good: it was never
committed.

### 5.2 Unstage: restore --staged

```bash
echo "temporary" > notes.txt
git add notes.txt
git status --short
git restore --staged notes.txt
git status --short
rm notes.txt
```

```
A  notes.txt
?? notes.txt
```

**Why:** `--staged` targets the staging area instead of the disk. It is the exact opposite of
`git add`: the file is still there, just no longer prepared for the commit.

### 5.3 Fix your last local commit: reset --soft

Add a line to `README.md` (`Contact: data-team@example.org`), then commit with a bad message:

```bash
git commit -am "wip"
git log --oneline -2
git reset --soft HEAD~1
git status --short
git commit -m "docs: add data team contact"
git log --oneline -2
```

```
f8a27e3 wip
0b8f703 Merge branch 'feat/minutes'

M  README.md

94f083c docs: add data team contact
0b8f703 Merge branch 'feat/minutes'
```

**Why:** `reset --soft HEAD~1` moved the `main` label back one commit, and left the changes
**staged** (`M` in the left column). We recommitted properly. The `wip` commit no longer appears in
the log. Fine here because it was never pushed.

> Without an option, `git reset HEAD~1` uses `--mixed`: same move, but the changes end up
> **unstaged** (right column). Only `--hard` deletes them.

### 5.4 Undo a shared commit: revert

Add a grand-total line: right after the last `print` inside the loop of `print_report`, add (at
the same indentation as the `for`):

```python
    print(f"{'ALL':<10} {sum(t for t, _ in report.values()):>12}")
```

```bash
git commit -am "feat: add grand total line"
python audience_report.py data/audience_sample.csv | tail -2
```

```
culture            1500             2.1
ALL               20250
```

Imagine this commit was pushed yesterday and the newsroom does not want it.

```bash
git revert --no-edit HEAD
git log --oneline -3
python audience_report.py data/audience_sample.csv | tail -1
```

```
92684c1 Revert "feat: add grand total line"
5019bca feat: add grand total line
94f083c docs: add data team contact

culture            1500             2.1
```

**Why:** `revert` does not erase anything: it adds a **new** commit that does the opposite. History
stays consistent for everyone who already pulled the old commit. This is the only undo that is
safe on shared history.

### 5.5 "I lost my commits!": reset --hard and reflog

```bash
git reset --hard HEAD~2
git log --oneline -3
```

```
HEAD is now at 94f083c docs: add data team contact

94f083c docs: add data team contact
0b8f703 Merge branch 'feat/minutes'
5afc440 style: rename views column to total views
```

The last two commits have vanished from the log. They are not gone:

```bash
git reflog -4
```

```
94f083c HEAD@{0}: reset: moving to HEAD~2
92684c1 HEAD@{1}: revert: Revert "feat: add grand total line"
5019bca HEAD@{2}: commit: feat: add grand total line
94f083c HEAD@{3}: commit: docs: add data team contact
```

```bash
git reset --hard HEAD@{1}
git log --oneline -3
```

```
HEAD is now at 92684c1 Revert "feat: add grand total line"
```

**Why:** a reset only moves a label; the commits stay inside `.git`. The **reflog** records every
position HEAD has had, so you can put the label back. What `reset --hard` destroys for real is
**uncommitted** work: that never became a commit, so there is nothing to find.

### 5.6 (If time allows) Visit an old commit

```bash
git switch --detach HEAD~5
git status
git switch main
```

```
HEAD is now at f1481db docs: add license note
HEAD detached at f1481db
```

**Why:** "detached HEAD" means HEAD points directly at a commit instead of a branch. Fine for
looking around (run the old script, read old files). If you commit in that state, create a branch
right away with `git switch -c my-rescue`, otherwise your commit has no label.

---

## Part 6 · Check your deliverable (5 min)

```bash
git log --oneline --graph --all
git log --all --oneline -- .env
cat .git/HEAD
```

You should see:

```
* 92684c1 Revert "feat: add grand total line"
* 5019bca feat: add grand total line
* 94f083c docs: add data team contact
*   0b8f703 Merge branch 'feat/minutes'
|\
| * 736de58 feat: show average read time in minutes
* | 5afc440 style: rename views column to total views
|/
* f1481db docs: add license note
* 882e7ec docs: add README with usage
* 230d366 chore: add .env.example
* fe2c345 feat: add daily audience report script
* 0f9164b chore: add .gitignore

ref: refs/heads/main
```

Checklist:

- [ ] at least six atomic commits with Conventional Commit messages
- [ ] one merge commit (two parents) with the conflict resolved and the script running
- [ ] one revert commit
- [ ] `git log --all --oneline -- .env` prints **nothing** (`.env` never committed)
- [ ] `cat .git/HEAD` shows `ref: refs/heads/main` (HEAD points at the branch `main`)

Hand in the output of `git log --oneline --graph --all`.

**Next session:** we publish this repository on GitHub. Make sure you have a GitHub account
before Session 2.

---

## Glossary

| Term | Meaning |
|---|---|
| repository | the `.git/` folder: all commits, branches and settings |
| working directory | the files on your disk, as you edit them |
| staging area / index | the next commit being prepared; filled by `git add` |
| commit | a snapshot of the whole project + author, date, message, parent(s) |
| hash | the unique id of a commit (`0f9164b...`); the first 7 characters are enough |
| branch | a movable label pointing at one commit |
| HEAD | "where I am"; normally points at a branch |
| `HEAD~1`, `HEAD~2` | the parent, the grandparent of HEAD |
| fast-forward | merge where the label simply slides forward; no new commit |
| three-way merge | merge of two diverged sides using their common ancestor; creates a merge commit |
| hunk | a block of consecutive changed lines in a diff; conflicts are cut by hunk |
| reflog | the local log of every position HEAD has had |
| detached HEAD | HEAD points at a commit, not at a branch |

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Screen full of `~`, cannot quit | Vim opened for a message | `Esc`, then `:wq`, Enter. Then set `core.editor` (Part 0) |
| `Author identity unknown` | name / email not configured | `git config --global user.name ...` and `user.email ...` |
| Everything in your home folder shows as untracked | `git init` run in the wrong folder | delete that `.git` (`rm -rf ~/.git`), `cd` into the project, `git init` again |
| `fatal: not a git repository` | you are not inside the project folder | `cd audience-report` |
| `.env` shows up in `git status` | `.gitignore` missing or misspelled | check with `git check-ignore -v .env` |
| `.env` already committed | ignored too late | `git rm --cached .env`, commit, revoke the secret if real |
| `error: Your local changes would be overwritten by checkout` | uncommitted edits block `git switch` | commit them, or `git stash`, switch, then `git stash pop` |
| `HEAD detached at ...` | you switched to a commit, not a branch | `git switch main` (or `git switch -c rescue` to keep new commits) |
| Merge went badly | conflict resolution is a mess | `git merge --abort` and start again |
| "My commits disappeared" | a `reset` moved the label | `git reflog`, then `git reset --hard HEAD@{n}` |
| Windows: warnings about `LF will be replaced by CRLF` | line endings | harmless for this course; ignore |

---

## Appendix A · Starter files

`audience_report.py`:

```python
"""Daily audience report: views and read time by section.

Usage: python audience_report.py data/audience_sample.csv
"""

import csv
import sys
from collections import defaultdict


def load_rows(path):
    with open(path, newline="", encoding="utf-8") as f:
        return list(csv.DictReader(f))


def section_totals(rows):
    views = defaultdict(int)
    weighted_time = defaultdict(float)
    timed_views = defaultdict(int)
    for row in rows:
        section = row["section"]
        n = int(row["views"])
        views[section] += n
        if row["avg_read_seconds"]:
            weighted_time[section] += n * float(row["avg_read_seconds"])
            timed_views[section] += n
    report = {}
    for section, total in views.items():
        avg = weighted_time[section] / timed_views[section] if timed_views[section] else None
        report[section] = (total, avg)
    return report


def print_report(report):
    print(f"{'section':<10} {'views':>8} {'avg read (s)':>13}")
    for section, (total, avg) in sorted(report.items(), key=lambda kv: -kv[1][0]):
        avg_txt = f"{avg:.1f}" if avg is not None else "n/a"
        print(f"{section:<10} {total:>8} {avg_txt:>13}")


def main():
    if len(sys.argv) != 2:
        print("usage: python audience_report.py <csv>")
        return 1
    rows = load_rows(sys.argv[1])
    print_report(section_totals(rows))
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

`data/audience_sample.csv`:

```
date,article_id,section,views,avg_read_seconds
2026-09-28,A001,politics,1200,95
2026-09-28,A002,sport,3400,40
2026-09-28,A003,culture,800,130
2026-09-28,A004,politics,2100,110
2026-09-29,A001,politics,600,90
2026-09-29,A005,sport,5200,35
2026-09-29,A006,economy,950,150
2026-09-29,A003,culture,400,120
2026-09-30,A007,economy,1800,160
2026-09-30,A005,sport,2600,38
2026-09-30,A008,culture,300,
2026-09-30,A004,politics,900,100
```
