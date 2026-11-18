# Challenge Questions

1. git restore --staged <file>

2. git restore <file>

3. git diff <commitA> <commitB>`. Add `--stat` for a summary, or `-- <file>` to limit it to one file.

4. git push origin --delete <branch>`

5. git tag -a v1.0 -m "Release 1.0"` then `git push origin v1.0`

6. git revert <commit>`. This creates a new commit that undoes the old one without rewriting history, so it's safe on shared branches.

7. git fetch downloads new commits from the remote and updates the remote-tracking branches but does not change your local branch or files. `git pull` is `git fetch` plus a merge (or rebase) into your current branch.

8. the first push succeeds. The second is rejected as non-fast-forward because the remote has commits that developer doesn't have. They must `git pull` (merge or rebase), resolve any conflicts, then push again. Using `git push --force` instead would overwrite and erase the first developer's commits.
