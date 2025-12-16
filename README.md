# **Git Enterprise Guide**

## 🚨 **Real-World Git Scenarios for Professional Developers**

*"Git doesn't break your code. Lack of Git knowledge breaks your code."*

## **Real-World Git Scenario Questions**

### **1. Merge Strategy Scenarios**
**Q1:** Your feature branch is 3 months behind main with hundreds of commits. You need to merge without breaking everything. What's your step-by-step approach?

**Q2:** You accidentally committed sensitive data (API keys) to a branch you already pushed. How do you remove it completely from history?

**Q3:** You need to deploy a hotfix but your current branch has experimental code. How do you extract just the stable changes?

### **2. Rebase vs Merge Dilemmas**
**Q4:** Team A says "always rebase before PR," Team B says "never rebase, always merge." Who's right for a shared feature branch with 4 developers?

**Q5:** You're asked to clean up 20 messy commits before code review. Do you use interactive rebase or squash merge? When does each backfire?

**Q6:** You rebased your branch, then someone else pushed to the same branch. Now you have diverged histories. How do you recover?

### **3. Team Collaboration Issues**
**Q7:** Two developers modified the same file in different ways. Git shows no conflict but the feature is broken. What went wrong?

**Q8:** Your CI/CD pipeline fails after a merge, but `git status` is clean. How do you identify which specific commit introduced the bug?

**Q9:** A junior developer force-pushed to main, overwriting critical commits. How do you restore without losing recent valid work?

### **4. Branch Management Problems**
**Q10:** You created a branch called `featrue-login` (typo). It's been pushed to remote and others are using it. How do you rename it properly?

**Q11:** Your PR has 15 commits, but company policy requires one commit per PR. You can't squash because others have based work on your branch. Solutions?

**Q12:** You need to work on 3 urgent bugs simultaneously. Do you create 3 branches from main, or branch from each other? How do you manage dependencies?

### **5. Advanced Recovery Scenarios**
**Q13:** You ran `git reset --hard` and lost uncommitted work. No stash, no backup. Is the data truly gone? Any recovery options?

**Q14:** During a complex rebase, you get stuck with conflicts you can't resolve. How do you abort safely without losing hours of work?

**Q15:** Git says "your branch is ahead by X commits, behind by Y commits." What does this actually mean, and how do you synchronize?

### **6. Performance & Optimization**
**Q16:** Your repo is 5GB and cloning takes 30 minutes. Daily operations are slow. What investigation steps and fixes do you propose?

**Q17:** `git status` shows "Changes not staged for commit" but `git diff` shows nothing. What's happening and how do you fix it?

**Q18:** You need to find which commit introduced a specific line of code that's now causing a bug, but the file has been moved/renamed.

### **7. CI/CD & Automation Issues**
**Q19:** Your deployment script runs `git pull` but fails with "would be overwritten" errors. How do you make it resilient?

**Q20:** You need to verify that the code being deployed matches exactly what was tested in the PR. How do you guarantee this with Git?

### **8. Permission & Security**
**Q21:** A contractor pushed commits with their personal email instead of company email. How do you rewrite the author for already-pushed commits?

**Q22:** You need to give a developer access to only one specific branch in a repository. Is this possible with standard Git?

**Q23:** Your team uses both SSH and HTTPS to clone the same repo. Now you have authentication conflicts. Best practice solution?

### **9. Submodules & Complex Repos**
**Q24:** Your project uses submodules. A teammate says "I updated the submodule" but your local copy doesn't show changes. What's happening?

**Q25:** You need to move a folder from a monorepo to its own repository while preserving history. What's the safest approach?

### **10. Real Emergency Scenarios**
**Q26:** `git` commands suddenly return "not a git repository" in your project folder. The `.git` folder is missing. Recovery steps?

**Q27:** During a deploy, you realize the wrong branch was merged to production. How do you revert while minimizing downtime?

**Q28:** Your team uses "squash and merge" on PRs, but now you need to cherry-pick a fix that was squashed. How do you locate the actual change?

### **11. Git Hooks & Automation**
**Q29:** Your pre-commit hook runs tests that take 2 minutes, slowing development. How do you balance code quality vs developer velocity?

**Q30:** A commit-msg hook enforces ticket numbers, but you need to make an urgent fix without a ticket. How do you bypass safely?

---

## **Sample Answer to Q1 (The 3-month lag scenario):**

**Approach:**
1. **First, save your current state:** `git stash` or commit everything locally as "WIP backup"
2. **Fetch latest main:** `git fetch origin main`
3. **Rebase incrementally:** Don't rebase all 3 months at once. Rebase week-by-week:  
   `git rebase --onto main~12weeks main~11weeks your-branch`
4. **Test at each milestone:** Run basic tests after each weekly chunk
5. **Handle conflicts strategically:** When conflicts arise, create temporary fix commits, continue rebase, then fix properly later
6. **Final cleanup:** Interactive rebase to squash temporary fix commits
7. **Pre-merge testing:** Create a test merge branch: `git checkout -b test-merge main && git merge your-branch --no-ff`
8. **CI verification:** Push test branch to run full CI pipeline before merging to main

**Key Insight:** The real problem isn't the merge—it's that your tests/CI should have been running against main regularly. Setup regular merges from main to your feature branch (weekly) to avoid this.







# **Comprehensive Git Scenario Solutions for Enterprise Developers**

## **1. Merge Strategy Scenarios**

### **Q1: Your feature branch is 3 months behind main with hundreds of commits. You need to merge without breaking everything.**

**Step-by-Step Strategy:**

```bash
# 1. First, save your current work
git stash push -m "pre-merge-backup"
# Or commit it as a backup
git add .
git commit -m "WIP: Backup before major merge"

# 2. Fetch the absolute latest main
git fetch origin main --prune

# 3. Create a merge-base branch to understand the gap
git checkout -b merge-base $(git merge-base main feature-branch)
# This shows you where you diverged 3 months ago

# 4. Do a test merge in a new branch FIRST
git checkout -b test-merge main
git merge feature-branch --no-commit --no-ff
# The --no-commit flag lets you see what would happen without actually committing

# 5. If conflicts appear immediately, abort and strategize
# Check what files will conflict:
git diff --name-only --diff-filter=U

# 6. Instead of one massive rebase, do incremental merges
# Create a temporary branch from where you diverged
git checkout -b incremental-merge $(git merge-base main feature-branch)

# 7. Cherry-pick or merge in chunks (weekly/monthly)
# Get commits from your feature branch, grouped by time
git log --oneline --since="3 months ago" --until="2.5 months ago"
# Cherry-pick manageable chunks
git cherry-pick <commit-range>  # Use with careful review

# 8. OR use git rerere (reuse recorded resolution) to help
git config rerere.enabled true
# This remembers how you resolved conflicts and reuses them

# 9. Best approach: Merge main into your branch regularly going forward
git checkout feature-branch
git merge main --no-ff
# Yes, merge main INTO your feature branch, not the other way around
# This keeps your branch updated and conflicts manageable

# 10. Use a visual merge tool for complex conflicts
git mergetool  # Configure your preferred tool (vscode, meld, kdiff3)

# 11. After merging main into your branch, run tests
# Then you can safely merge your branch back to main
```

**Critical Insight:** The real solution starts **before** this problem. Setup automation:
```bash
# In your CI pipeline, add:
# "Daily: Merge main into all feature branches older than 3 days"
# This prevents the 3-month gap from happening
```

### **Q2: You accidentally committed sensitive data (API keys) to a branch you already pushed.**

**Immediate Action Plan:**

```bash
# 1. DO NOT PANIC. First, invalidate the exposed credentials
# Contact your security team/rotate API keys immediately

# 2. Local cleanup with BFG Repo-Cleaner (recommended) or git filter-repo
# Download BFG: https://rtyley.github.io/bfg-repo-cleaner/

# 3. Remove the sensitive file from ALL history
bfg --delete-files config/credentials.json
# Or remove specific strings:
bfg --replace-text passwords.txt  # passwords.txt contains: PASSWORD1==>***

# 4. Force push (this is the ONE time force push is acceptable)
git push origin --force --all

# 5. Notify EVERYONE on the team
# They must: git fetch origin && git reset --hard origin/feature-branch

# 6. For multiple branches, you need to clean each
git checkout main
bfg --delete-files credentials.json
git reflog expire --expire=now --all
git gc --prune=now --aggressive
git push origin --force

# 7. Add to .gitignore immediately
echo "config/credentials.json" >> .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add sensitive files to gitignore"

# 8. Use git-secret or git-crypt for future secrets
# Pre-commit hook to prevent secrets:
# https://github.com/awslabs/git-secrets
```

**Alternative if BFG isn't available:**
```bash
# Using git filter-branch (slower but built-in)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch config/credentials.json" \
  --prune-empty --tag-name-filter cat -- --all
```

### **Q3: You need to deploy a hotfix but your current branch has experimental code.**

**Strategic Extraction:**

```bash
# Method 1: Stash selectively
# 1. First, commit your experimental work as WIP
git add .
git commit -m "WIP: Experimental feature - DO NOT MERGE"

# 2. Create hotfix branch from main (clean state)
git checkout main
git pull origin main
git checkout -b hotfix/urgent-issue

# 3. Make and test your hotfix
# ... fix the bug ...
git add .
git commit -m "fix: critical production issue [PROD-123]"

# 4. Deploy from hotfix branch
# Merge to main and tag
git checkout main
git merge --no-ff hotfix/urgent-issue
git tag -a v1.2.3-hotfix -m "Hotfix for PROD-123"

# 5. Return to your feature branch and continue
git checkout feature-branch
# If you need the hotfix in your feature branch too:
git merge hotfix/urgent-issue

# Method 2: Cherry-pick specific commits
# If the fix is already in your experimental branch
git log --oneline --grep="fix"
# Copy the commit hash
git checkout main
git cherry-pick <commit-hash>  # Only the fix, not the experimental code

# Method 3: Checkout specific files
git checkout main -- path/to/file/needing/fix.js
# This brings just that file from main to your current branch
```

---

## **2. Rebase vs Merge Dilemmas**

### **Q4: Team A says "always rebase before PR," Team B says "never rebase, always merge." Who's right?**

**The Pragmatic Answer:** It depends on your workflow and team size.

**Use REBASE when:**
```bash
# Scenario 1: Solo developer or feature owner
# You're cleaning up your own messy commit history
git rebase -i main

# Scenario 2: Preparing a PR for review
# You want a clean, linear history for reviewers
git fetch origin main
git rebase origin/main
# This replays your commits on top of latest main

# Scenario 3: Fixing a typo in an earlier commit
git rebase -i HEAD~3
# Change "pick" to "edit" for the commit with typo
```

**Use MERGE when:**
```bash
# Scenario 1: Shared feature branch with 4 developers
# NEVER rebase a shared branch - you'll rewrite public history
git merge main  # Merge main into feature branch
# This preserves everyone's work and creates a merge commit

# Scenario 2: You want to preserve the true development history
# Merge commits show when features were integrated
git merge --no-ff feature-branch
# Creates explicit merge commit (good for audit trails)

# Scenario 3: Completing a feature/PR
# On main: git merge --no-ff feature-branch
# This marks the completion point clearly
```

**Hybrid Approach (Best Practice):**
```bash
# Individual developers: rebase locally
git rebase -i @{u}~3  # Interactive rebase last 3 commits

# Team branch: merge regularly
git checkout team-feature-branch
git merge main --no-ff  # Keep updated with main

# PR completion: squash or merge based on policy
# For clean history: git merge --squash feature-branch
# For full history: git merge --no-ff feature-branch
```

### **Q5: Clean up 20 messy commits before code review. Interactive rebase or squash merge?**

**When to use Interactive Rebase:**
```bash
# You want to:
# 1. Reorder commits logically
# 2. Split large commits into smaller ones
# 3. Rewrite commit messages
# 4. Remove debug commits entirely

git rebase -i HEAD~20
# Then:
# - pick: keep commit
# - reword: change message
# - edit: modify commit contents
# - squash: combine with previous
# - fixup: combine and discard message
# - drop: remove commit

# Example transformation:
# Before: 20 commits with "WIP", "fix", "oops", "tmp"
# After: 5 logical commits:
# 1. feat: add user authentication
# 2. test: auth unit tests  
# 3. refactor: auth middleware
# 4. docs: update API docs
# 5. chore: update dependencies
```

**When to use Squash Merge (in PR/GitHub/GitLab):**
```bash
# Use squash merge when:
# 1. The PR has many small commits that tell a story
# 2. You want to preserve the PR discussion history
# 3. Individual commit quality is poor but overall change is good

# GitHub/GitLab provides "Squash and merge" button
# This creates ONE clean commit on main while keeping
# the messy development history in the feature branch

# NEVER squash if:
# - Commits are already clean and logical
# - You need to cherry-pick individual fixes later
# - You're working with git bisect (squash breaks it)
```

**Danger of Interactive Rebase:**
```bash
# If others have already based work on your branch:
# DO NOT REBASE
# You'll force them to do painful resets

# Check if branch is shared:
git branch -r --contains feature-branch
# If others see it, use merge instead
```

### **Q6: You rebased your branch, then someone else pushed to the same branch.**

**Recovery Process:**
```bash
# 1. First, don't panic and DON'T force push
git fetch origin

# 2. See what happened:
git log --oneline --graph origin/feature-branch feature-branch

# 3. You have two options:

# Option A: Merge the diverged branches
git merge origin/feature-branch
# This creates a merge commit reconciling both histories

# Option B: Rebase again (if you MUST have linear history)
git pull --rebase origin feature-branch
# This replays your new commits on top of their new commits

# 4. If you already force-pushed (bad!):
# Roll back and fix properly
git reflog  # Find where you were before disaster
git reset --hard HEAD@{5}  # Go back 5 operations

# 5. Best practice: Use --force-with-lease
git push --force-with-lease origin feature-branch
# This fails if someone else pushed, preventing overwrites
```

---

## **3. Team Collaboration Issues**

### **Q7: Two developers modified the same file differently. Git shows no conflict but feature is broken.**

**The Silent Conflict Problem:**

```bash
# This happens with:
# 1. Non-conflicting but incompatible changes
# Example: 
# Dev A changed function signature: processUser(user)
# Dev B calls the function: processUser(user, true)

# 2. Changes to different parts that break runtime
# 3. Dependency updates that aren't compatible

# Detection:
# 1. Run integration tests (should catch this)
# 2. Use semantic conflict detection tools

# Solution:
# 1. Better communication - talk before changing shared interfaces
# 2. Feature flags for major changes
# 3. API versioning for breaking changes

# Git command to see combined changes:
git diff HEAD...origin/main -- path/to/file.js
# Shows total difference between your branch and main

# Use git blame to understand who changed what:
git blame -L 50,100 file.js  # Lines 50-100
# Then talk to that person!
```

### **Q8: CI/CD pipeline fails after a merge, but `git status` is clean.**

**Debugging Strategy:**

```bash
# 1. First, reproduce locally
git checkout main
git pull origin main
npm install  # Or your package manager
npm test     # Run the same tests as CI

# 2. If tests pass locally, check environment differences
# Common issues:
# - Node version mismatch
# - Missing environment variables
# - Different OS (Linux vs Windows line endings)

# 3. Use git bisect to find the culprit
git bisect start
git bisect bad  # Current commit fails
git bisect good v1.0.0  # Last known good version

# Git will binary search through commits
# For each commit, run your test script
git bisect run npm test

# 4. Check for hidden files
git clean -xdn  # Show what would be removed (dry run)
git clean -xdf  # Actually remove untracked files

# 5. Check submodules if used
git submodule status
git submodule update --init --recursive

# 6. Look at the actual merge commit
git show --name-only MERGE_HEAD
# Shows files changed in merge

# 7. Check for file permissions (common on Linux)
git diff --summary  # Shows mode changes (755 vs 644)
```

### **Q9: Junior developer force-pushed to main, overwriting critical commits.**

**Emergency Recovery:**

```bash
# 1. IMMEDIATELY lock the main branch
# GitHub/GitLab: Settings → Branches → Protect main branch
# Require pull requests, disable force push

# 2. Find the lost commits
# Every developer who pulled before the force push has them
git reflog show origin/main
# Look for commits before the bad push

# 3. Restore main to previous state
# Find the good commit hash from reflog
git checkout -b recovery-branch <good-commit-hash>

# 4. Force push the correct history BACK
git push origin recovery-branch:main --force
# Yes, you need to force push to fix a force push

# 5. Notify everyone to reset
# Team must run:
git fetch origin
git reset --hard origin/main

# 6. If commits are truly lost from all origins:
# Check CI system artifacts - they often clone repos
# Check backup systems
# Check other developers' machines

# 7. Implement prevention:
# Pre-receive hooks on server:
#!/bin/bash
# Prevent force push to main
if [ "$1" = "refs/heads/main" ]; then
    echo "Force push to main is prohibited"
    exit 1
fi

# 8. Train the junior developer
# Teach: git revert instead of git reset --hard
# Teach: git push --force-with-lease
```

---

## **4. Branch Management Problems**

### **Q10: Rename a branch with typo that's already pushed and used by others.**

**Safe Rename Process:**

```bash
# Scenario: Branch "featrue-login" (typo) instead of "feature-login"

# Step 1: Rename locally FIRST
git branch -m featrue-login feature-login

# Step 2: Push the new name to remote
git push origin feature-login

# Step 3: Delete the old name from remote
git push origin --delete featrue-login

# Step 4: Update tracking relationship
git branch --unset-upstream  # Remove old tracking
git branch -u origin/feature-login  # Set new tracking

# Step 5: Communicate to team members
# They need to:
git fetch origin
git checkout featrue-login
git branch -m featrue-login feature-login
git branch --unset-upstream
git branch -u origin/feature-login

# If someone has local commits not pushed:
git checkout featrue-login
git branch -m feature-login  # Rename
git fetch origin
git reset --hard origin/feature-login  # Align with remote

# Alternative if many people are using it:
# Keep both branches temporarily
git push origin feature-login
# Don't delete old branch yet
# Update documentation
# After 2 weeks, delete old branch
```

### **Q11: PR has 15 commits but company requires one commit per PR. Others have based work on your branch.**

**Solutions Without Breaking Others:**

```bash
# Option 1: Create a new squashed branch
git checkout main
git checkout -b feature-login-squashed
git merge --squash feature-login
git commit -m "feat: complete login implementation"
# Push this NEW branch for PR

# Keep original branch for others
git checkout feature-login
# Others continue working here

# Option 2: Use GitHub/GitLab squash merge feature
# Keep all 15 commits in feature branch
# Use UI "Squash and merge" button when approving PR
# This creates 1 commit on main but preserves branch history

# Option 3: Interactive rebase with backup branch
# First, create backup for others
git branch feature-login-backup feature-login

# Now squash locally
git checkout feature-login
git rebase -i main
# Change all but first "pick" to "squash"

# Force push with warning
git push origin feature-login --force-with-lease

# Tell team members to rebase:
git fetch origin
git rebase origin/feature-login

# Option 4: Change company policy
# Argue that:
# - Multiple commits help with bisect debugging
# - They preserve development context
# - Tools like git blame work better
```

### **Q12: Work on 3 urgent bugs simultaneously.**

**Branch Strategy:**

```bash
# Recommended: Branch from main for each bug
git checkout main
git pull origin main
git checkout -b hotfix/bug-1

git checkout main
git checkout -b hotfix/bug-2

git checkout main  
git checkout -b hotfix/bug-3

# Work on them in any order
# When fixing bug-1:
git add .
git commit -m "fix: resolve null pointer in user service [BUG-1]"

# If bug-2 depends on bug-1 fix:
git checkout hotfix/bug-2
git merge hotfix/bug-1 --no-ff
# Now bug-2 branch has bug-1 fix

# Merge strategy:
# 1. Merge smallest/independent fixes first
# 2. Test each fix individually before combining

# Bad approach: Branch from each other
# Creates dependency chain that's hard to untangle

# Use git worktree for parallel work:
git worktree add ../bug-1 hotfix/bug-1
git worktree add ../bug-2 hotfix/bug-2
# Now you have separate folders for each branch
# Can work on all simultaneously without stashing

# Stashing alternative:
git stash push -m "bug-1-wip"
git checkout hotfix/bug-2
# ... fix bug-2 ...
git stash pop  # Return to bug-1
```

---

## **5. Advanced Recovery Scenarios**

### **Q13: Ran `git reset --hard`, lost uncommitted work. No stash, no backup.**

**Data Recovery Options:**

```bash
# 1. Check if your IDE/editor has local history
# VSCode: Local History extension
# IntelliJ: Local History feature
# Most editors keep temporary backups

# 2. Check for file system recovery
# On Unix: check .swp, .swo files (vim recovery)
ls -la .*.sw*

# 3. Git might still have it (if staged even briefly)
git fsck --lost-found
# This finds dangling commits/blobs
cd .git/lost-found
# Check the files here

# 4. Use git reflog for committed work
git reflog
# Look for commits before reset
git reset --hard HEAD@{1}  # Go back one step in reflog

# 5. For truly uncommitted work:
# If you added to index (git add) even briefly:
git fsck --cache --unreachable | grep blob | cut -d' ' -f3 | xargs -I{} git show {}

# 6. Operating system recovery:
# Mac: Time Machine backups
# Windows: File History / Previous Versions
# Linux: check /tmp or backup systems

# 7. PREVENTION is key:
# Add to your .bashrc/.zshrc:
alias gah='git reset --hard'
alias gahs='echo "DANGER: Use git stash instead" && false'
# Override the dangerous command

# Use pre-commit hooks to auto-stash:
#!/bin/sh
# auto-stash.sh
if ! git diff --quiet || ! git diff --cached --quiet; then
    git stash push -m "auto-stash-$(date +%s)"
fi
```

### **Q14: Stuck in complex rebase with unresolvable conflicts.**

**Safe Abort Strategy:**

```bash
# 1. First, save your conflict resolution work
# Copy conflicted files somewhere safe
cp -r . /tmp/recovery-backup/

# 2. Standard abort
git rebase --abort
# This returns to pre-rebase state

# 3. If --abort fails (rare):
git reset --hard ORIG_HEAD
# ORIG_HEAD is where you were before rebase

# 4. If even that fails:
git reflog
# Find the commit before rebase started
git reset --hard HEAD@{5}  # Adjust number

# 5. Alternative: Skip the problematic commit
# During rebase, for the conflicted commit:
git add .  # Stage resolution (even if incomplete)
git rebase --skip  # Skip this commit
# WARNING: You lose that commit's changes

# 6. Better: Use merge instead of rebase
git merge main --no-ff
# Keep your branch history as is

# 7. For future: Rebase in smaller chunks
git rebase -i main~10  # Rebase last 10 commits only
# Less likely to get complex conflicts

# 8. Use rerere (reuse recorded resolution)
git config rerere.enabled true
git rerere  # Manually record current resolution
git rebase --continue
```

### **Q15: "Your branch is ahead by X commits, behind by Y commits."**

**Understanding & Synchronization:**

```bash
# What this means:
# Ahead: You have commits that remote doesn't have
# Behind: Remote has commits you don't have

# Visualize:
git log --oneline --graph --all

# Solution 1: Simple push (if only ahead)
git push origin feature-branch

# Solution 2: Pull first (if behind)
git pull origin feature-branch
# This fetches AND merges

# Solution 3: Pull with rebase (clean history)
git pull --rebase origin feature-branch
# Puts your commits on top of remote

# Solution 4: When both ahead AND behind:
# This happens when you and someone else both pushed
# You need to integrate changes

git fetch origin
git merge origin/feature-branch
# Creates merge commit reconciling both

# OR rebase your commits on top:
git rebase origin/feature-branch

# Solution 5: Reset if your local commits are bad
git fetch origin
git reset --hard origin/feature-branch
# WARNING: Discards your local commits

# Best practice workflow:
# 1. Always pull before starting work
git pull --rebase origin main
# 2. Commit frequently
# 3. Push when ready
# 4. If push rejected: pull --rebase first
```

---

## **6. Performance & Optimization**

### **Q16: Repo is 5GB, cloning takes 30 minutes, operations are slow.**

**Investigation & Fixes:**

```bash
# 1. Identify what's bloating the repo
git count-objects -vH  # Check pack size
du -sh .git            # Total .git size

# 2. Find large files in history
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  awk '/^blob/ {print substr($0,6)}' | \
  sort --numeric-sort --key=2 | \
  tail -10  # Top 10 largest files

# 3. Use BFG to clean history
bfg --strip-blobs-bigger-than 100M  # Remove files >100MB
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 4. Use shallow clone for CI/CD
git clone --depth 1 https://github.com/company/repo.git
# Only gets latest, not full history

# 5. Use sparse checkout if only need some files
git clone --filter=blob:none --no-checkout repo
cd repo
git sparse-checkout init --cone
git sparse-checkout set src/  # Only get src folder
git checkout main

# 6. Configure git for large repos
git config core.preloadindex true
git config core.fscache true
git config gc.auto 256  # More frequent GC

# 7. Split monorepo if possible
# Use git subtree or submodules

# 8. Add .gitignore rules
# Common culprits: node_modules, .DS_Store, *.log, build artifacts

# 9. Use git LFS for binaries
git lfs track "*.psd"
git lfs track "*.mp4"
```

### **Q17: `git status` shows "Changes not staged for commit" but `git diff` shows nothing.**

**Diagnosis & Fix:**

```bash
# This is usually a file mode/permissions issue
# Common on Linux/Mac when file permissions change

# 1. Check if it's file mode changes
git config core.filemode
# If true, Git tracks permission changes

# 2. See what actually changed
git diff --summary
# Shows mode changes like: mode change 100644 => 100755

# 3. Fix: Ignore permission changes
git config core.filemode false
# Then:
git add -A
git status  # Should be clean now

# 4. Alternative: Reset file modes
git diff --name-only | xargs chmod 644
# Or for executables:
git diff --name-only | xargs chmod 755

# 5. Could also be line ending issues (Windows/Linux)
git config --global core.autocrlf true  # Windows
git config --global core.autocrlf input # Linux/Mac

# 6. Check for case sensitivity issues (Mac/Windows)
# File was renamed with different case
git mv OldFile.txt oldfile.txt  # Fix case

# 7. Clean cache if corrupted
git rm -r --cached .
git reset HEAD -- .
git status
```

### **Q18: Find which commit introduced a bug in code that's been moved/renamed.**

**Tracking Through File Moves:**

```bash
# 1. Use git log with follow flag
git log --follow -p -- path/to/current/file.js
# --follow tracks file through renames

# 2. Use git blame with follow
git blame -C -C -C path/to/file.js
# -C finds copied code from other files
# More -C flags = more aggressive copy detection

# 3. Find when file was renamed
git log --name-status --oneline | grep -B2 -A2 "R.*file"
# R = rename

# 4. Use git pickaxe search for specific code
git log -S "functionName()" --all
# Find commits that added/removed this string

# 5. Combine with line numbers
git blame -L 50,100 path/to/file.js

# 6. Full investigation script:
#!/bin/bash
# trace-bug.sh
BUGGY_CODE="badFunctionCall()"

# Find all occurrences in history
git log --all -p | \
  grep -B5 -A5 "$BUGGY_CODE" | \
  grep "^commit " | \
  head -5

# 7. Visualize with gitk
gitk --all -- path/to/file.js

# 8. If file was split/merged:
# Use git log --find-renames
git log --find-renames=40%  # 40% similarity threshold
```

---

## **7. CI/CD & Automation Issues**

### **Q19: Deployment script runs `git pull` but fails with "would be overwritten" errors.**

**Resilient Deployment Script:**

```bash
#!/bin/bash
# deploy.sh - Safe git operations for CI/CD

set -e  # Exit on error

# 1. Use a lock file to prevent concurrent deployments
LOCK_FILE="/tmp/deploy.lock"
if [ -f "$LOCK_FILE" ]; then
    echo "Deployment already in progress"
    exit 1
fi
trap "rm -f $LOCK_FILE" EXIT
touch "$LOCK_FILE"

# 2. Always start from known state
cd /var/www/app
git reset --hard HEAD
git clean -f -d

# 3. Fetch with prune (remove stale branches)
git fetch origin --prune

# 4. Check if we can fast-forward
if git merge-base --is-ancestor HEAD origin/main; then
    # Safe to fast-forward
    git merge --ff-only origin/main
else
    # Need proper merge, backup first
    BACKUP_TAG="backup-$(date +%Y%m%d-%H%M%S)"
    git tag "$BACKUP_TAG"
    git merge origin/main --no-ff --strategy=recursive --strategy-option=theirs
fi

# 5. Alternative: Always use reset if you don't care about local changes
# git fetch origin main
# git reset --hard origin/main

# 6. Log the deployment
git log --oneline -5
echo "Deployed: $(git rev-parse --short HEAD)"

# 7. Clean up old backups (keep last 10)
git tag -l "backup-*" | sort -r | tail -n +11 | xargs git tag -d
```

### **Q20: Verify deployed code matches exactly what was tested in PR.**

**Deployment Integrity Check:**

```bash
# Method 1: SHA-based verification
# In CI pipeline:
PR_SHA=$(git rev-parse HEAD)
echo "PR_SHA=$PR_SHA" >> $GITHUB_ENV

# In deployment script:
EXPECTED_SHA="{{PR_SHA}}"
ACTUAL_SHA=$(git rev-parse HEAD)

if [ "$EXPECTED_SHA" != "$ACTUAL_SHA" ]; then
    echo "ERROR: SHA mismatch!"
    echo "Expected: $EXPECTED_SHA"
    echo "Actual: $ACTUAL_SHA"
    exit 1
fi

# Method 2: Tag-based deployment
# When PR is approved and tested:
git tag -a "release-$(date +%Y%m%d-%H%M%S)-${PR_NUMBER}" -m "PR $PR_NUMBER"
git push origin --tags

# Deployment script:
TAG=$(git describe --tags --abbrev=0)
git checkout "$TAG"

# Method 3: Checksum verification
# Generate checksum of critical files
find src/ -type f -name "*.js" -exec md5sum {} \; > /tmp/checksums.pr
# Deploy, then verify:
find src/ -type f -name "*.js" -exec md5sum {} \; > /tmp/checksums.deploy
diff /tmp/checksums.pr /tmp/checksums.deploy

# Method 4: Use Git Notes for metadata
git notes add -m "PR: $PR_NUMBER | Tested-by: CI-$RUN_ID"
git push origin refs/notes/*

# Verify on server:
git fetch origin refs/notes/*:refs/notes/*
git notes show
```

---

## **8. Permission & Security**

### **Q21: Contractor pushed with personal email instead of company email.**

**Rewriting Author Information:**

```bash
# WARNING: This rewrites history - force push required
# Only do on feature branches, not main

# 1. Clone fresh to avoid issues
git clone --bare https://github.com/company/repo.git
cd repo.git

# 2. Use git filter-branch
git filter-branch --env-filter '
OLD_EMAIL="contractor@gmail.com"
CORRECT_NAME="Jane Contractor"
CORRECT_EMAIL="jane@company.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]; then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]; then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --all

# 3. Clean up
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 4. Push back
git push --force --tags origin 'refs/heads/*'

# 5. For all developers:
git fetch origin
git reset --hard origin/feature-branch

# 6. Prevention: Use commit template
git config commit.template .gitmessage.txt
# In .gitmessage.txt:
# Author: company.email@domain.com

# 7. Use pre-commit hook to validate
#!/bin/sh
# .git/hooks/commit-msg
COMPANY_DOMAIN="@company.com"
if [[ "$(git config user.email)" != *"$COMPANY_DOMAIN" ]]; then
    echo "ERROR: Use your company email"
    exit 1
fi
```

### **Q22: Give developer access to only one specific branch.**

**Git Branch Permissions:**

```bash
# Git itself doesn't do permissions - use hosting platform

# GitHub:
# 1. Go to repo Settings → Branches
# 2. Add branch protection rule for main
# 3. Require PR, require reviews
# 4. Go to Settings → Collaborators
# 5. Add person with "Write" access
# 6. They can push to any branch but can't merge to protected branches

# GitLab:
# 1. Settings → Repository → Protected Branches
# 2. Protect main: Maintainers can merge
# 3. Create new branch: Developers can push
# 4. Use "Push Rules" for finer control

# Bitbucket:
# 1. Repository settings → Branch permissions
# 2. Add permission: branch name = "feature/*"
# 3. Allow specific users/groups

# Server-side hooks (advanced):
# In .git/hooks/pre-receive on server:
#!/bin/bash
while read oldrev newrev refname; do
    branch=$(git rev-parse --symbolic --abbrev-ref $refname)
    user=$(whoami)
    
    if [ "$branch" = "main" ] && [ "$user" != "allowed-user" ]; then
        echo "Access denied to main branch"
        exit 1
    fi
done

# Fork workflow alternative:
# Developer forks repo, works in their fork
# Submits PR from their branch to upstream main
```

### **Q23: Team uses both SSH and HTTPS, causing authentication conflicts.**

**Standardization Solution:**

```bash
# 1. Choose one protocol for the team
# Recommendation: Use SSH for developers, HTTPS for CI

# 2. Check current remote URL
git remote -v
# origin  https://github.com/company/repo.git (fetch)
# origin  https://github.com/company/repo.git (push)

# 3. Convert HTTPS to SSH
git remote set-url origin git@github.com:company/repo.git

# 4. Or SSH to HTTPS
git remote set-url origin https://github.com/company/repo.git

# 5. Use git config to set default
git config --global url."git@github.com:".insteadOf "https://github.com/"
# Now `git clone https://github.com/company/repo` uses SSH automatically

# 6. For CI/CD systems, use tokens:
# GitHub: Personal Access Token
git clone https://token@github.com/company/repo.git

# 7. Store credentials securely
# SSH: Use ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa

# HTTPS: Use credential helper
git config --global credential.helper cache
# Stores for 15 minutes

# 8. Team documentation should specify:
# - SSH key setup guide
# - Standard remote URL format
# - Troubleshooting steps
```

---

## **9. Submodules & Complex Repos**

### **Q24: Teammate says "I updated the submodule" but your local doesn't show changes.**

**Submodule Synchronization Issue:**

```bash
# Understanding: Submodules are pointers to specific commits
# Your repo stores which commit of submodule to use

# 1. Check current submodule state
git submodule status
# Shows commit hash and path
# - (dash) means not initialized
# + (plus) means different commit than recorded

# 2. Update submodules
git submodule update --init --recursive
# --init: initializes if needed
# --recursive: for nested submodules

# 3. If teammate updated submodule commit:
# First, get latest main repo
git pull origin main

# Then update submodules to new commits
git submodule update --recursive

# 4. If you need to update submodule yourself:
cd path/to/submodule
git checkout main
git pull origin main
cd ..
git add path/to/submodule
git commit -m "Update submodule to latest"
git push origin main

# 5. Common problem: Submodule pointer not updated
# Teammate updated submodule code but didn't commit the pointer change
git submodule foreach git pull origin main
# Updates all submodules to latest

# 6. Use --remote to track branch
git submodule update --remote
# Fetches latest from submodule's tracked branch

# 7. Prevention: Use subtrees instead of submodules
# Convert submodule to subtree:
git subtree add --prefix=lib/dependency https://github.com/vendor/dependency.git main --squash
# Easier to manage, no pointer commits
```

### **Q25: Move a folder from monorepo to its own repository while preserving history.**

**Splitting Repository with History:**

```bash
# Using git filter-repo (recommended):
# 1. Install filter-repo
pip install git-filter-repo

# 2. Clone the monorepo
git clone https://github.com/company/monorepo.git
cd monorepo

# 3. Extract folder with history
git filter-repo --path packages/my-module/ --subdirectory-filter packages/my-module
# This keeps only files from that folder, at repo root

# 4. Create new remote repo on GitHub/GitLab
# 5. Add new remote
git remote add origin https://github.com/company/my-module.git
git push -u origin main

# Using git subtree (alternative):
# 1. Prepare split in monorepo
git subtree split -P packages/my-module -b split-branch

# 2. Create new repo
mkdir my-module && cd my-module
git init
git pull ../monorepo split-branch

# 3. Clean up and push
git remote add origin https://github.com/company/my-module.git
git push -u origin main

# Keep connection with subtrees:
# In monorepo, after moving:
git remote add module-origin https://github.com/company/my-module.git
git subtree add --prefix=packages/my-module module-origin main --squash

# Now you can pull updates:
git subtree pull --prefix=packages/my-module module-origin main --squash

# Update the module from monorepo:
cd packages/my-module
# Make changes
git add . && git commit -m "Update module"
git push module-origin main
cd ../..
git commit -am "Update module subtree"
```

---

## **10. Real Emergency Scenarios**

### **Q26: `git` commands return "not a git repository" but you're in your project folder.**

**Recovery Steps:**

```bash
# 1. Check if .git folder exists
ls -la .git
# If missing, something deleted it

# 2. Look for backup .git folders
find . -name ".git*" -type d
# Maybe .git moved or renamed

# 3. Check if you're in subdirectory
pwd
git rev-parse --show-toplevel
# Should show repo root

# 4. If .git is truly gone:
# Option A: Re-clone (loses local uncommitted changes)
cd /tmp
git clone https://github.com/company/repo.git
cp -r /tmp/repo/.git /path/to/project/

# Option B: Recover from remote
cd /path/to/project
git init
git remote add origin https://github.com/company/repo.git
git fetch origin
git reset --hard origin/main
# WARNING: Loses all local commits not pushed

# Option C: Check for backups
# - Time Machine (Mac)
# - File History (Windows)  
# - Server backups
# - CI system artifacts

# 5. If .git is corrupted:
mv .git .git.corrupted
git init
git remote add origin https://github.com/company/repo.git
git fetch origin
# Try to recover specific branches:
git checkout -b main origin/main

# 6. Prevention:
# Regular backups of .git folder
# Push to remote frequently
# Use git bundle for local backups:
git bundle create repo-backup.bundle --all
```

### **Q27: Wrong branch merged to production during deploy.**

**Emergency Rollback:**

```bash
# Scenario: feature/experimental merged instead of hotfix/production

# 1. IMMEDIATE: Block new deployments
# Turn off CI/CD pipeline
# Notify team

# 2. Identify what was deployed
git log --oneline -5 main
# Find the bad merge commit

# 3. Revert the merge commit
git revert -m 1 <merge-commit-hash>
# -m 1 keeps main's side of merge (not feature branch)
# Creates a new commit that undoes the merge

# 4. Deploy the revert
git push origin main

# 5. If revert causes conflicts (complex merge):
# Use reset to go back before merge
git reset --hard <commit-before-merge>
git push --force origin main
# WARNING: This rewrites history - only if no one else has pulled

# 6. Alternative: Cherry-pick fixes
# If feature branch had some good changes
git log feature/experimental --oneline
git cherry-pick <good-commit-hash>

# 7. Communicate:
# - What happened
# - How it was fixed
# - Prevention steps

# 8. Prevention for future:
# - Deployment checklists
# - Pre-deployment approvals
# - Staging environment testing
# - Feature flags for risky changes
```

### **Q28: Team uses "squash and merge" on PRs, need to cherry-pick a fix that was squashed.**

**Finding the Actual Change:**

```bash
# Problem: Squash merge combines many commits into one
# The fix you need is buried in the squash

# 1. Find the squashed commit
git log --oneline --grep="PR #123" -5
# Find the squash merge commit

# 2. Look at what changed in that commit
git show <squash-commit-hash> --stat
# See all files changed

# 3. Search for the specific fix
git show <squash-commit-hash> | grep -B5 -A5 "fix-message"

# 4. If you know the original PR branch:
# Check if branch still exists locally/remotely
git branch -r | grep "PR-123"
# If yes, checkout and find the fix commit

# 5. Use git log with pickaxe on squash commit
git log -S "specificCodeChange" --all
# Might find other occurrences

# 6. Extract just the fix from squashed commit:
git checkout -b extract-fix <squash-commit-hash>~1
# Go to parent of squash commit
git cherry-pick -n <squash-commit-hash>
# Stage changes but don't commit
git reset  # Unstage all
git add path/to/fix.js  # Only stage the fix file
git commit -m "fix: extract specific fix from PR #123"

# 7. Prevention:
# Don't squash if you need to cherry-pick later
# Or, tag important individual commits before squash:
git tag "fix-bug-456" <commit-hash>
git push origin --tags
```

---

## **11. Git Hooks & Automation**

### **Q29: Pre-commit hook runs 2-minute tests, slowing development.**

**Balancing Speed vs Quality:**

```bash
# Solution: Staged testing - only test what changed

#!/bin/bash
# .git/hooks/pre-commit
# Only run tests on staged files

# 1. Get list of staged files
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM)

# 2. Only run tests if relevant files changed
if echo "$STAGED_FILES" | grep -q "\.js$"; then
    echo "Running JavaScript tests..."
    npm test -- --findRelatedTests $STAGED_FILES
    # Jest: --findRelatedTests runs tests for changed files only
fi

if echo "$STAGED_FILES" | grep -q "\.py$"; then
    echo "Running Python tests..."
    python -m pytest $(echo "$STAGED_FILES" | grep "\.py$")
fi

# 3. Use lint-staged for different file types
# package.json:
{
  "lint-staged": {
    "*.js": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}

# 4. Skip hook when needed (with warning)
git commit -m "WIP" --no-verify
# Or set environment variable:
BY_PASS_HOOKS=1 git commit -m "Emergency fix"

# 5. Fast pre-commit, full tests in CI
# Pre-commit: quick linting
# CI: full test suite

# 6. Progressive enhancement:
# First: Only syntax checking (fast)
# Second: Unit tests for changed files (medium)
# CI: Full integration tests (slow but thorough)
```

### **Q30: Commit-msg hook enforces ticket numbers, but you need urgent fix without ticket.**

**Safe Bypass Methods:**

```bash
# Hook code (should allow emergencies):
#!/bin/bash
# .git/hooks/commit-msg

COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# Allow emergency bypass
if [[ "$COMMIT_MSG" =~ ^EMERGENCY: ]]; then
    echo "WARNING: Emergency commit - skipping ticket validation"
    exit 0
fi

# Allow hotfix pattern
if [[ "$COMMIT_MSG" =~ ^(fix|hotfix):.*\[HOTFIX\] ]]; then
    echo "Hotfix detected - creating ticket automatically"
    # Auto-create ticket via API
    curl -X POST https://api.company.com/tickets \
      -d '{"title":"Emergency fix", "type":"hotfix"}'
    exit 0
fi

# Normal validation
if [[ ! "$COMMIT_MSG" =~ \[[A-Z]+-[0-9]+\] ]]; then
    echo "ERROR: Commit must include ticket number [PROJ-123]"
    echo "For emergencies, use: EMERGENCY: description"
    exit 1
fi

# Alternative bypass methods:

# 1. Use --no-verify flag
git commit -m "fix: critical production outage" --no-verify

# 2. Temporary disable hook
mv .git/hooks/commit-msg .git/hooks/commit-msg.disabled
git commit -m "Emergency fix"
mv .git/hooks/commit-msg.disabled .git/hooks/commit-msg

# 3. Environment variable bypass
export SKIP_COMMIT_HOOKS=1
git commit -m "Emergency fix"

# 4. Special branch for emergencies
git checkout -b emergency-fix
# This branch could have different hooks or no hooks

# 5. Post-commit cleanup
# Commit without ticket, then amend:
git commit -m "fix: database connection" --no-verify
# Get ticket created
git commit --amend -m "fix: database connection [PROD-456]"

# Important: All emergency commits should be reviewed
# and linked to tickets retroactively
```

---

## **Bonus: Git Configuration for Enterprise Teams**

```bash
# .gitconfig for all team members
[core]
    editor = code --wait  # Use VSCode as editor
    autocrlf = input      # Linux/Mac line endings
    filemode = false      # Ignore permission changes
[push]
    default = current     # Push current branch only
[pull]
    rebase = true         # Rebase by default (local branches)
[merge]
    ff = only             # Only fast-forward merges
[rerere]
    enabled = true        # Reuse conflict resolutions
[alias]
    # Safety aliases
    unstage = restore --staged
    discard = restore
    # Useful aliases
    lol = log --oneline --graph --all
    lola = log --oneline --graph --all --decorate
    # Undo aliases
    undo = reset HEAD~1
    amend = commit --amend --no-edit
[commit]
    template = ~/.gitmessage
[init]
    defaultBranch = main

# Team-specific hooks in repo
# These should be versioned in repo and installed via:
# git config core.hooksPath .githooks
```

## **Key Takeaways for Developers:**

1. **Think Before You Push:** Git is powerful but dangerous
2. **Communicate:** Most Git problems are team coordination problems
3. **Automate:** Use hooks, CI/CD, and scripts to prevent human error
4. **Backup:** Regular pushes to remote, use reflog, know recovery paths
5. **Simplify:** Choose workflows your whole team can understand
6. **Document:** Keep a team Git cheat sheet for common scenarios

This comprehensive guide covers real enterprise scenarios developers face daily. Master these, and you'll be the Git expert on any team.

---

## Final Note

Use this list to:

* Prepare for interviews
* Identify weak areas
* Teach others
* Build confidence

Contributions welcome. ⭐
