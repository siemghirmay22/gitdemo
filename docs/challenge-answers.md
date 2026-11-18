# Challenge Questions

1. **Unstage a file without deleting it:** `git restore --staged <file>` (older syntax: `git reset HEAD <file>`). The change stays in the working directory.

2. **Discard uncommitted changes in a modified file:** `git restore <file>` (older syntax: `git checkout -- <file>`). This permanently loses the uncommitted edits.

3. **Compare two commits:** `git diff <commitA> <commitB>`. Add `--stat` for a summary, or `-- <file>` to limit it to one file.

4. **Delete a remote branch:** `git push origin --delete <branch>`

5. **Create and push tag v1.0:** `git tag -a v1.0 -m "Release 1.0"` then `git push origin v1.0`

6. **Revert a previously committed change:** `git revert <commit>`. This creates a new commit that undoes the old one without rewriting history, so it's safe on shared branches.

7. **git fetch vs git pull:** `git fetch` downloads new commits from the remote and updates the remote-tracking branches (e.g. `origin/main`) but does not change your local branch or files. `git pull` is `git fetch` plus a merge (or rebase) into your current branch.

8. **Two developers pushing to the same branch:** the first push succeeds. The second is rejected as non-fast-forward because the remote has commits that developer doesn't have. They must `git pull` (merge or rebase), resolve any conflicts, then push again. Using `git push --force` instead would overwrite and erase the first developer's commits.
