# Git Practice Scenarios: ShopEasy

Completed by Siem Ghirmay, Nov 18, 2026. Each task ran in its own practice folder (`~/git-practice/taskN`).
Below each task are the key takeaways, then the full transcript of the commands and their output.

## Task 1: Setup and first commits

- `git config` without `--global` writes to `.git/config`, so the name and email apply to this repo only (`--local` is the default).
- Staging only `README.md` left `cart.txt` untracked (`??`) until the second commit.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -b main
Initialized empty Git repository in /home/siem/git-practice/task1/shopeasy/.git/
git config user.name "$NAME"
git config user.email "$EMAIL"
git config --local --list | grep user
user.name=Siem Ghirmay
user.email=siemable@gmail.com
echo "ShopEasy" > README.md
echo "add item" > cart.txt
git add README.md
git status --short
A  README.md
?? cart.txt
git commit -m "Add README"
[main (root-commit) 5a708f1] Add README
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
git status --short
?? cart.txt
git add cart.txt
git commit -m "Add cart module"
[main 5fd1862] Add cart module
 1 file changed, 1 insertion(+)
 create mode 100644 cart.txt
# Verify
git log --oneline
5fd1862 Add cart module
5a708f1 Add README
git status
On branch main
nothing to commit, working tree clean
```

</details>

## Task 2: Undoing mistakes (working directory and staging)

- `git restore <file>` discards working-directory changes; `git restore --staged <file>` unstages but keeps the edit.
- `.gitignore` with `*.log` and `secrets.env` hides both files from `git status`.
- `git commit --amend` rewrites the last commit. That's only safe before the commit is pushed and shared.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
echo "add item" > cart.txt && git add cart.txt && git commit -qm "Add cart module"
# 1. Edit then discard
echo "oops edit" >> cart.txt
git status --short
 M cart.txt
git restore cart.txt
cat cart.txt
add item
# 2. Ignore *.log and secrets.env
echo "API_KEY=fake" > secrets.env
echo "debug" > debug.log
git status --short
?? debug.log
?? secrets.env
printf "*.log\nsecrets.env\n" > .gitignore
git status --short
?? .gitignore
git add .gitignore && git commit -m "Add .gitignore"
[main 966d2e3] Add .gitignore
 1 file changed, 2 insertions(+)
 create mode 100644 .gitignore
# 3. Stage by accident, then unstage keeping the edit
echo "remove item" >> cart.txt
git add cart.txt
git status --short
M  cart.txt
git restore --staged cart.txt
git status --short
 M cart.txt
cat cart.txt
add item
remove item
git commit -qam "Add remove item to cart"
# 4. Fix a typo in the last commit message
echo "pay" > payment.txt && git add payment.txt
git commit -m "Ad payment"
[main a057b32] Ad payment
 1 file changed, 1 insertion(+)
 create mode 100644 payment.txt
git commit --amend -m "Add payment"
[main 286b2d5] Add payment
 Date: Wed Nov 18 12:25:50 2026 -0800
 1 file changed, 1 insertion(+)
 create mode 100644 payment.txt
# Verify
git status
On branch main
nothing to commit, working tree clean
git log -1 --oneline
286b2d5 Add payment
```

</details>

## Task 3: Branching and merging

- `feature/wishlist` fast-forwarded because `main` hadn't moved (it was an ancestor of the branch).
- Committing on both `main` and `feature/reviews` made them diverge, so `--no-ff` produced a real merge commit.
- `git branch -d` (safe form) deleted both because they were fully merged.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
echo "ShopEasy" > README.md && git add . && git commit -qm "Initial commit"
# 1. feature/wishlist with two commits
git switch -c feature/wishlist
Switched to a new branch 'feature/wishlist'
echo "wishlist v1" > wishlist.txt && git add . && git commit -m "Add wishlist"
[feature/wishlist ffbd9b7] Add wishlist
 1 file changed, 1 insertion(+)
 create mode 100644 wishlist.txt
echo "wishlist v2" >> wishlist.txt && git commit -am "Improve wishlist"
[feature/wishlist 5d6b207] Improve wishlist
 1 file changed, 1 insertion(+)
# 2. Fast-forward merge
git switch main
Switched to branch 'main'
git merge feature/wishlist
Updating 8328e56..5d6b207
Fast-forward
 wishlist.txt | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 wishlist.txt
# 3. Diverge
git switch -c feature/reviews
Switched to a new branch 'feature/reviews'
echo "reviews" > reviews.txt && git add . && git commit -m "Add reviews"
[feature/reviews 777cb08] Add reviews
 1 file changed, 1 insertion(+)
 create mode 100644 reviews.txt
git switch main
Switched to branch 'main'
echo "Home page" > home.txt && git add . && git commit -m "Add home page"
[main 163bdac] Add home page
 1 file changed, 1 insertion(+)
 create mode 100644 home.txt
git log --oneline --graph --all
* 777cb08 Add reviews
| * 163bdac Add home page
|/  
* 5d6b207 Improve wishlist
* ffbd9b7 Add wishlist
* 8328e56 Initial commit
# 4. Force a merge commit
git merge --no-ff feature/reviews -m "Merge feature/reviews"
Merge made by the 'ort' strategy.
 reviews.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 reviews.txt
# 5. Delete merged branches (safe form)
git branch -d feature/wishlist feature/reviews
Deleted branch feature/wishlist (was 5d6b207).
Deleted branch feature/reviews (was 777cb08).
# Verify
git log --oneline --graph --all
*   1aee724 Merge feature/reviews
|\  
| * 777cb08 Add reviews
* | 163bdac Add home page
|/  
* 5d6b207 Improve wishlist
* ffbd9b7 Add wishlist
* 8328e56 Initial commit
git branch
* main
```

</details>

## Task 4: Resolving merge conflicts

- The conflict markers split `<<<<<<< HEAD` (main: 110), `=======`, and `>>>>>>> feature/discount` (90).
- `git merge --abort` restored the exact pre-merge state.
- After editing to `Price: 95`, `git add` marks the conflict resolved, and `git commit` finishes the merge.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
# 1-3
echo "Price: 100" > price.txt && git add . && git commit -m "Set price 100"
[main (root-commit) e2f2d59] Set price 100
 1 file changed, 1 insertion(+)
 create mode 100644 price.txt
git switch -c feature/discount
Switched to a new branch 'feature/discount'
echo "Price: 90" > price.txt && git commit -am "Discount price 90"
[feature/discount 2b16b58] Discount price 90
 1 file changed, 1 insertion(+), 1 deletion(-)
git switch main
Switched to branch 'main'
echo "Price: 110" > price.txt && git commit -am "Raise price 110"
[main d1fdc6c] Raise price 110
 1 file changed, 1 insertion(+), 1 deletion(-)
# 4. Conflict
git merge feature/discount
Auto-merging price.txt
CONFLICT (content): Merge conflict in price.txt
Automatic merge failed; fix conflicts and then commit the result.
git status --short
UU price.txt
cat price.txt
<<<<<<< HEAD
Price: 110
=======
Price: 90
>>>>>>> feature/discount
# 5. Practice 1: abort
git merge --abort
git status
On branch main
nothing to commit, working tree clean
git log --oneline
d1fdc6c Raise price 110
e2f2d59 Set price 100
# 6. Practice 2: redo and resolve with Price: 95
git merge feature/discount
Auto-merging price.txt
CONFLICT (content): Merge conflict in price.txt
Automatic merge failed; fix conflicts and then commit the result.
echo "Price: 95" > price.txt
git add price.txt
git commit -m "Merge feature/discount: resolve price to 95"
[main 62ece1a] Merge feature/discount: resolve price to 95
# Verify
cat price.txt
Price: 95
grep -c '<<<<<<<' price.txt
0
git log --oneline --graph
*   62ece1a Merge feature/discount: resolve price to 95
|\  
| * 2b16b58 Discount price 90
* | d1fdc6c Raise price 110
|/  
* e2f2d59 Set price 100
```

</details>

## Task 5: Stash and context switching

- `git stash push -m` saves tracked changes only; `-u` would be needed for brand-new files.
- The hotfix branch was created from `main`, not from the feature branch.
- `git stash pop` applies and removes the entry (`apply` would keep it), so `git stash list` is empty.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
echo "add item" > cart.txt && echo "login v1" > login.txt && git add . && git commit -qm "Initial commit"
git switch -c feature/cart
Switched to a new branch 'feature/cart'
# 1-2. Edit and stash
echo "update quantity" >> cart.txt
git stash push -m "WIP cart"
Saved working directory and index state On feature/cart: WIP cart
git stash list
stash@{0}: On feature/cart: WIP cart
git status --short
# 3. Hotfix from main
git switch main
Switched to branch 'main'
git switch -c hotfix/login
Switched to a new branch 'hotfix/login'
echo "login v1 (fixed)" > login.txt && git commit -am "Fix login bug"
[hotfix/login 0672579] Fix login bug
 1 file changed, 1 insertion(+), 1 deletion(-)
git switch main
Switched to branch 'main'
git merge hotfix/login
Updating b11e660..0672579
Fast-forward
 login.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
# 4. Back to the branch, restore work
git switch feature/cart
Switched to branch 'feature/cart'
git stash pop
On branch feature/cart
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   cart.txt

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (44a2e4e7a50b3d3315b9aac9c39b47d1e3a07cf0)
# 5. Verify
git stash list
git status --short
 M cart.txt
cat cart.txt
add item
update quantity
```

</details>

## Task 6: Rewriting and undoing history

- `git revert` keeps commit C in history and adds a new commit that undoes it. That's safe on shared branches because nobody's history is rewritten.
- `reset --soft HEAD~1` moved the branch back but left E's changes staged.
- After `reset --hard HEAD~2`, the reflog still pointed at the lost commit, and `git reset --hard HEAD@{1}` brought it back.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
# 1. Commits A-D
for c in A B C D; do echo "$c" > "$c.txt"; git add .; git commit -qm "Commit $c"; done
git log --oneline
886cfb2 Commit D
4ae5c17 Commit C
0a92ca1 Commit B
db574a6 Commit A
# 2. Revert C (history kept, new commit added)
git revert --no-edit HEAD~1
[main d6a6981] Revert "Commit C"
 Date: Wed Nov 18 12:25:51 2026 -0800
 1 file changed, 1 deletion(-)
 delete mode 100644 C.txt
git log --oneline
d6a6981 Revert "Commit C"
886cfb2 Commit D
4ae5c17 Commit C
0a92ca1 Commit B
db574a6 Commit A
ls
A.txt
B.txt
D.txt
# 3. Commit E then soft reset
echo "E" > E.txt && git add . && git commit -qm "Commit E"
git reset --soft HEAD~1
git status --short
A  E.txt
git commit -qm "Commit E"
# 4. Accidental hard reset, then recover with reflog
git reset --hard HEAD~2
HEAD is now at 886cfb2 Commit D
git log --oneline
886cfb2 Commit D
4ae5c17 Commit C
0a92ca1 Commit B
db574a6 Commit A
git reflog -5
886cfb2 HEAD@{0}: reset: moving to HEAD~2
e1c7238 HEAD@{1}: commit: Commit E
d6a6981 HEAD@{2}: reset: moving to HEAD~1
e1c7238 HEAD@{3}: commit: Commit E
d6a6981 HEAD@{4}: revert: Revert "Commit C"
git reset --hard HEAD@{1}
HEAD is now at e1c7238 Commit E
# Verify
git log --oneline
e1c7238 Commit E
d6a6981 Revert "Commit C"
886cfb2 Commit D
4ae5c17 Commit C
0a92ca1 Commit B
db574a6 Commit A
```

</details>

## Task 7: Rebase and a clean history

- Rebasing replays commits on the new base, so they get new ids (different parent = different hash).
- The interactive rebase marked commits 2-4 as `squash` (scripted here with `GIT_SEQUENCE_EDITOR`; by hand you'd change `pick` to `squash` in the editor). `HEAD~4` means the last 4 commits.
- The branch sat directly on top of `main`, so `--ff-only` worked and the history is a straight line.
- Don't rebase commits others have already pulled, because it rewrites their ids.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
echo "ShopEasy" > README.md && git add . && git commit -qm "Initial commit"
# 1. Messy feature commits
git switch -c feature/checkout
Switched to a new branch 'feature/checkout'
echo "checkout step 1" > checkout.txt && git add . && git commit -qm "wip"
echo "chekout step 2" >> checkout.txt && git commit -qam "more wip"
sed -i 's/chekout/checkout/' checkout.txt && git commit -qam "fix typo"
echo "checkout step 3" >> checkout.txt && git commit -qam "fix again"
git log --oneline --graph --all
* b8a9992 fix again
* 2affd9a fix typo
* dfc7f7e more wip
* 401a352 wip
* 04d8a85 Initial commit
# 2. New commit on main
git switch main
Switched to branch 'main'
echo "Footer" > footer.txt && git add . && git commit -qm "Add footer"
# 3. Rebase onto updated main (ids change)
git switch feature/checkout
Switched to branch 'feature/checkout'
git rebase main
Rebasing (1/4)Rebasing (2/4)Rebasing (3/4)Rebasing (4/4)Successfully rebased and updated refs/heads/feature/checkout.
git log --oneline --graph --all
* 74511a0 fix again
* 37421de fix typo
* a782543 more wip
* 480250f wip
* f1db2bc Add footer
* 04d8a85 Initial commit
# 4. Squash last 4 into one (scripted rebase -i: pick, squash, squash, squash)
GIT_SEQUENCE_EDITOR="sed -i '2,\$s/^pick/squash/'" git rebase -i HEAD~4
Rebasing (2/4)Rebasing (3/4)Rebasing (4/4)[detached HEAD d18a8a8] wip
 Date: Wed Nov 18 12:25:51 2026 -0800
 1 file changed, 3 insertions(+)
 create mode 100644 checkout.txt
Successfully rebased and updated refs/heads/feature/checkout.
git commit --amend -m "Add checkout flow"
[feature/checkout 14fab1d] Add checkout flow
 Date: Wed Nov 18 12:25:51 2026 -0800
 1 file changed, 3 insertions(+)
 create mode 100644 checkout.txt
# 5. Fast-forward merge
git switch main
Switched to branch 'main'
git merge --ff-only feature/checkout
Updating f1db2bc..14fab1d
Fast-forward
 checkout.txt | 3 +++
 1 file changed, 3 insertions(+)
 create mode 100644 checkout.txt
# Verify
git log --oneline --graph
* 14fab1d Add checkout flow
* f1db2bc Add footer
* 04d8a85 Initial commit
```

</details>

## Task 8: Cherry-pick, tags and comparing

- `git cherry-pick dev~1` copied only the fix onto `main`; the copy gets a new id because its parent is different.
- The annotated tag stores a tagger, date and message (visible in `git show v1.0.0`); a lightweight tag is just a pointer.
- `git diff v1.0.0` compares the tag with the working tree.

<details><summary>Commands and output</summary>

```text
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
echo "ShopEasy" > README.md && git add . && git commit -qm "Initial commit"
# 1. Three commits on dev, the 2nd is the fix
git switch -c dev
Switched to a new branch 'dev'
echo "experimental search" > search.txt && git add . && git commit -qm "Add experimental search"
echo "total = price * qty" > total.txt && git add . && git commit -qm "Fix cart total bug"
echo "beta banner" > banner.txt && git add . && git commit -qm "Add beta banner"
git log --oneline
4616cf7 Add beta banner
9efecb2 Fix cart total bug
c8736c1 Add experimental search
04d8a85 Initial commit
# 2. Cherry-pick only the fix onto main
git switch main
Switched to branch 'main'
git cherry-pick dev~1
[main cb41b36] Fix cart total bug
 Date: Wed Nov 18 12:25:51 2026 -0800
 1 file changed, 1 insertion(+)
 create mode 100644 total.txt
git log --oneline
cb41b36 Fix cart total bug
04d8a85 Initial commit
# 3. Annotated tag
git tag -a v1.0.0 -m "Release 1.0.0"
git show v1.0.0 --stat
tag v1.0.0
Tagger: Siem Ghirmay <siemable@gmail.com>
Date:   Wed Nov 18 12:25:51 2026 -0800

Release 1.0.0

commit cb41b36af902ede583d1153cf0724da6bf542063
Author: Siem Ghirmay <siemable@gmail.com>
Date:   Wed Nov 18 12:25:51 2026 -0800

    Fix cart total bug

 total.txt | 1 +
 1 file changed, 1 insertion(+)
# 4. Change and diff vs tag
echo "Welcome to ShopEasy" >> README.md
git diff v1.0.0
diff --git a/README.md b/README.md
index 69cd2c5..6008ead 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,2 @@
 ShopEasy
+Welcome to ShopEasy
# 5. List tags
git tag
v1.0.0
```

</details>

## Task 9: Remotes and collaboration

- A bare repo has no working files, so nobody edits on the server and pushes can't clobber a checked-out tree.
- `push -u` set up tracking; `fetch` updated only `origin/main`, and `main..origin/main` showed the teammate's commit before `pull`.
- `git push origin --delete feature/temp` removed the remote branch.

<details><summary>Commands and output</summary>

```text
git init --bare -b main server.git
Initialized empty Git repository in /home/siem/git-practice/task9/server.git/
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
echo "ShopEasy" > README.md && git add . && git commit -qm "Initial commit"
# 1. Add remote, push with upstream
git remote add origin ../server.git
git push -u origin main
To ../server.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
# 2. Teammate clones, commits, pushes
cd ..
git clone server.git teammate
Cloning into 'teammate'...
done.
cd teammate
git config user.name "Teammate" && git config user.email "teammate@example.com"
echo "Shipping info" > shipping.txt && git add . && git commit -m "Add shipping info"
[main 0251620] Add shipping info
 1 file changed, 1 insertion(+)
 create mode 100644 shipping.txt
git push
To /home/siem/git-practice/task9/server.git
   04d8a85..0251620  main -> main
# 3. Fetch, compare, pull
cd ../shopeasy
git fetch
From ../server
   04d8a85..0251620  main       -> origin/main
git status
On branch main
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
git log --oneline main..origin/main
0251620 Add shipping info
git pull
Updating 04d8a85..0251620
Fast-forward
 shipping.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 shipping.txt
# 4. Push a branch, then delete it on the remote
git switch -c feature/temp
Switched to a new branch 'feature/temp'
git push -u origin feature/temp
To ../server.git
 * [new branch]      feature/temp -> feature/temp
branch 'feature/temp' set up to track 'origin/feature/temp'.
git branch -r
  origin/HEAD -> origin/main
  origin/feature/temp
  origin/main
git push origin --delete feature/temp
To ../server.git
 - [deleted]         feature/temp
git switch main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
# Verify
git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
git branch -r
  origin/HEAD -> origin/main
  origin/main
```

</details>

## Task 10: Final challenge (mixed scenario)

- The hotfix was merged and tagged `v1.0.1`, then the feature was rebased on top of it, and the conflict in `app.txt` was resolved before `git rebase --continue`.
- The work was squashed with `git reset --soft main` plus one commit, then merged with `--no-ff`.
- `--follow-tags` pushed the annotated tag; a plain push doesn't send tags.
- `git revert -m 1 HEAD` undid the merge, keeping parent 1 (`main`) as the mainline.

<details><summary>Commands and output</summary>

```text
git init -q --bare -b main server.git
mkdir shopeasy && cd shopeasy
git init -q -b main && git config user.name "$NAME" && git config user.email "$EMAIL"
printf "checkout: v1\npayment: none\n" > app.txt && git add . && git commit -qm "Initial app"
git remote add origin ../server.git && git push -qu origin main
git switch -c feature/payments
Switched to a new branch 'feature/payments'
sed -i 's/payment: none/payment: card/' app.txt && git commit -qam "Add card payment"
echo "paypal" >> app.txt
# 1. Stash, hotfix, merge, tag v1.0.1
git stash push -m "WIP payments"
Saved working directory and index state On feature/payments: WIP payments
git switch main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
git switch -c hotfix/critical
Switched to a new branch 'hotfix/critical'
sed -i 's/payment: none/payment: disabled/' app.txt && git commit -qam "Hotfix: disable broken payment path"
git switch main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
git merge hotfix/critical
Updating 00b0163..458aafd
Fast-forward
 app.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
git tag -a v1.0.1 -m "Hotfix release 1.0.1"
git branch -d hotfix/critical
Deleted branch hotfix/critical (was 458aafd).
# 2. Back to feature, restore stash, commit, rebase
git switch feature/payments
Switched to branch 'feature/payments'
git stash pop
On branch feature/payments
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   app.txt

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (56e737a06a9c2ad7f276d30647d85767a0c6a0a4)
git commit -qam "Add PayPal"
git rebase main
Rebasing (1/2)Auto-merging app.txt
CONFLICT (content): Merge conflict in app.txt
error: could not apply f968b0e... Add card payment
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
hint: Disable this message with "git config set advice.mergeConflict false"
Could not apply f968b0e... # Add card payment
# 3. Resolve the rebase conflict and continue
git status --short
UU app.txt
cat app.txt
checkout: v1
<<<<<<< HEAD
payment: disabled
=======
payment: card
>>>>>>> f968b0e (Add card payment)
printf "checkout: v1\npayment: card\n" > app.txt
git add app.txt
git rebase --continue
[detached HEAD 138e8e3] Add card payment
 1 file changed, 1 insertion(+), 1 deletion(-)
Rebasing (2/2)Successfully rebased and updated refs/heads/feature/payments.
git log --oneline --graph --all
* 04ca9ff Add PayPal
* 138e8e3 Add card payment
* 458aafd Hotfix: disable broken payment path
* 00b0163 Initial app
# 4. Squash, merge, push with tags
git reset --soft main
git commit -m "Add payments (card + PayPal)"
[feature/payments 46cf468] Add payments (card + PayPal)
 1 file changed, 2 insertions(+), 1 deletion(-)
git switch main
Switched to branch 'main'
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
git merge --no-ff feature/payments -m "Merge feature/payments"
Merge made by the 'ort' strategy.
 app.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
git push origin main --follow-tags
To ../server.git
   00b0163..3f5a044  main -> main
 * [new tag]         v1.0.1 -> v1.0.1
git ls-remote --tags origin
9f610938d04c7402e643074084a8f3351ed0789b	refs/tags/v1.0.1
458aafd076386176239a0a019ea2fa3acf6f0b68	refs/tags/v1.0.1^{}
# 5. Production broke: revert the merge commit, keeping main (parent 1)
git revert -m 1 --no-edit HEAD
[main b8180c0] Revert "Merge feature/payments"
 Date: Wed Nov 18 12:25:51 2026 -0800
 1 file changed, 1 insertion(+), 2 deletions(-)
cat app.txt
checkout: v1
payment: disabled
git push origin main
To ../server.git
   3f5a044..b8180c0  main -> main
# Verify
git log --oneline --graph
* b8180c0 Revert "Merge feature/payments"
*   3f5a044 Merge feature/payments
|\  
| * 46cf468 Add payments (card + PayPal)
|/  
* 458aafd Hotfix: disable broken payment path
* 00b0163 Initial app
git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

</details>

## Bonus questions

1. **git fetch vs git pull:** `fetch` downloads commits and updates remote-tracking branches like `origin/main` without touching your branch or files. `pull` = `fetch` + merge (or rebase) into the current branch.
2. **Revert vs reset:** use `revert` on anything already pushed or shared, because it adds an undo commit and leaves history intact. `reset` rewrites history, so keep it for local, unshared commits.
3. **Why rebase is dangerous on shared branches:** it replaces commits with new ones (new ids). Teammates who built on the old commits end up with duplicated or diverged history and painful merges.
4. **git reset --hard:** moves the branch and overwrites the staging area and working directory, throwing away uncommitted changes. Committed work is still in the reflog (`git reflog`), so `git reset --hard HEAD@{n}` can bring it back. Uncommitted changes are gone for good.
5. **Finding the commit that introduced a bug:** `git bisect start`, `git bisect bad` (current), `git bisect good <known-good>`, then test each commit Git checks out and mark it good/bad until it names the first bad commit, then `git bisect reset`. `git bisect run <test-script>` automates it.
