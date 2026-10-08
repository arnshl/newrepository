# Session 01 · Exercises: Git as a Commit Graph

**To do after the session, on your own.** In class you built `audience-report/` step by step with
the guide. Here you redo the same skills **without** the guide, in a separate folder, so that you
know you can do it alone.

Self-contained. Work through these on your own machine, terminal only. Before each command, write
down what you expect; then run it and compare. Stop at `[BLOQUÉ?]` if you are stuck.

Starting material: copy `starter/audience_report.py` and `starter/data/audience_sample.csv` (also in
Appendix A of the course guide) into a new empty folder `audience-report-practice/` (keep the `data/`
subfolder). Do **not** reuse or nest it inside your class repository.

## Exercise 1 · Predict the status (paper first)

In a repository whose last commit contains `a.py` and `b.py`, the following commands are run in
order. After **each** line, write the exact output of `git status --short`.

```bash
echo "x = 1" >> a.py
git add a.py
echo "y = 2" >> a.py
echo "z = 3" > c.py
git add c.py
git restore --staged c.py
git restore a.py
```

Then reproduce it in a scratch folder and check every line.

## Exercise 2 · A clean repository in six atomic commits

In `audience-report-practice/`, initialise a repository on branch `main` and produce **exactly** these six
commits, in this order, each with a Conventional Commit message you choose:

1. A `.gitignore` that ignores at least `.env`, `.venv/`, `__pycache__/`.
2. The starter script and its sample data.
3. A `README.md` with one usage line.
4. A `.env.example` declaring `AUDIENCE_API_TOKEN=change-me`.
5. A change to the script: when `avg_read_seconds` is missing for a whole section, print `-`
   instead of `n/a`.
6. A change to the script: sort sections alphabetically instead of by views.

Then:

- Create a real `.env` with a fake token, run `git add -A`, and show with `git status` that it is not
  staged.
- Paste the output of `git log --oneline` in a file `answers.md`.

## Exercise 3 · Provoke and resolve a conflict

1. From the tip of `main`, create branch `feat/percent`. On it, add a column showing each section's
   share of total views in percent (one commit).
2. Switch back to `main`. Change the header line so the first column is titled `rubrique` instead of
   `section` (one commit).
3. Before merging, predict: fast-forward or three-way? Conflict or not? Write your prediction in
   `answers.md`.
4. Merge `feat/percent` into `main`. Resolve the conflict so the final report has **both** the
   `rubrique` title and the percent column. Run the script before committing.
5. Paste `git log --oneline --graph --all` in `answers.md`.

## Exercise 4 · Undo the right way

For each situation, name the command you would use (`restore`, `restore --staged`, `reset --soft`,
`reset --hard`, `revert`) and justify in one sentence.

a. You edited `audience_report.py` for ten minutes, the result is worse, nothing is staged.
b. You ran `git add .env` by mistake (your `.gitignore` was missing). Nothing committed yet.
c. Your last commit, not pushed, has the message `wip`.
d. Commit 6 from Exercise 2 (alphabetical sort) was pushed yesterday; the newsroom wants the old
   order back.
e. Same as c., but you also want to throw away everything that commit changed.

Then actually perform **d.** on your repository and paste the last three lines of
`git log --oneline` in `answers.md`.

## Exercise 5 · Lose a commit, find it again

1. Make a throwaway commit `docs: temporary note` that adds a line to the README.
2. Run `git reset --hard HEAD~1`. Check with `git log` that the commit is gone.
3. Use `git reflog` to find its hash and bring `main` back to it.
4. In `answers.md`, explain in two sentences why the commit was recoverable, and in which situation
   it would not have been.

## Exercise 6 (bonus) · Split a messy edit with `git add -p`

Make two unrelated edits to `audience_report.py` in one go (for example: fix a typo in the usage
message, and add a `--top N` option). Use `git add -p` to turn them into **two** commits with two
distinct Conventional Commit types.

## `[BLOQUÉ?]`

If you got stuck, write here which exercise, the exact command you ran, its full output (copy-paste,
not a summary), and what you expected instead. Then ask.
