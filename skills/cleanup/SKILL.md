---
name: cleanup
description: Clean up local git state by switching to main, pulling latest, pruning remote-tracking branches, and deleting local branches whose remote has been deleted. Use when the user says "cleanup", "clean up branches", "sync main", or invokes /cleanup.
---

## Cleanup

### When to use

* "cleanup"
* "clean up branches"
* "sync main"
* "prune branches"
* Any time the user wants to reset local git state back to a clean main

---

## Steps

### 1. Check for uncommitted changes

Run `git status`. If there are uncommitted changes or staged files:

- **Warn the user** and list the dirty files.
- Ask whether to stash, discard, or abort before proceeding.
- **Do not continue** until the working tree is clean or the user has explicitly
  chosen what to do with the changes.

### 2. Switch to main

```bash
git checkout main
```

If the checkout fails (e.g. detached HEAD, merge conflict), diagnose and report
to the user before continuing.

### 3. Pull latest

```bash
git pull origin main
```

### 4. Prune remote-tracking branches

```bash
git fetch --prune
```

This removes local remote-tracking refs for branches that no longer exist on the
remote.

### 5. Delete stale local branches

Find local branches whose upstream has been deleted:

```bash
git branch -vv | grep ': gone]'
```

For each branch listed:

- Use `git branch -d {branch}` (safe delete).
- If `-d` fails because the branch is "not fully merged" (common after
  squash/rebase merges on GitHub), inform the user and list those branches.
  Only use `git branch -D` if the user explicitly confirms.

### 6. Report to user

Tell the user:

- Now on `main`, synced with origin
- Which branches were deleted
- Which branches (if any) could not be safely deleted and why

---

## Rules

- **Never delete branches without checking for uncommitted work first.**
- **Never force-delete (`-D`) without user confirmation.**
- **Never discard uncommitted changes silently.**
- **Keep it idempotent** — running cleanup twice in a row should be safe.
