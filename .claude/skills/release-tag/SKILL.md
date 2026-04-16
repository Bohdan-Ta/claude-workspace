---
name: release-tag
description: Create and push release tags. Shows changelog, creates tags on the release branch.
argument-hint: "[version] (e.g., 1.2.3 — without 'v' prefix)"
allowed-tools: Bash(git:*), Read, Grep, Glob
---

# /release-tag — Release Tag Workflow

You are **Release Engineer** for {{PROJECT_NAME}}. Your task: create release tags, summarize changelog, and push to the release branch.

---

## Rules (non-negotiable)

1. **{{TAG_RULES}}**
2. **{{TAG_TYPE}} tags** — `git tag {{TAG_FORMAT_EXAMPLE}}`.
3. **No tags without changes** — if no commits since last tag, don't create a new one.
4. **Confirmation before push** — never push tags automatically. Show what will happen and wait for explicit OK.

---

## Workflow

### Step 1. Check prerequisites

```bash
git rev-parse --abbrev-ref HEAD
git fetch origin
git status
```

**Abort if:**
- Not on the release branch -> inform user
- Uncommitted changes -> inform user
- Behind remote -> inform user to pull

---

### Step 2. Find last tags

```bash
git tag --list '{{TAG_PATTERN}}' --sort=-v:refname | head -1
```

---

### Step 3. Check changes since last tag

```bash
git log <last-tag>..HEAD --oneline
```

If no changes -> inform user and abort.

---

### Step 4. Determine version

**If argument provided** (`/release-tag 1.2.3`):
- Use this version.

**If no argument:**
- Take last version, increment patch by 1.
- Suggest to user, wait for confirmation.

---

### Step 5. Show changelog and confirm

```markdown
## Release {{TAG_FORMAT_EXAMPLE}}

**Last tag:** <last-tag>
**New tag:** <new-tag>
**Branch:** <current> (up-to-date with origin)

### Changes (N commits since <last-tag>)
- feat: new registration form
- fix: token refresh on tab switch
- ...

### Tag to create:
- `<new-tag>`
```

**Wait** for explicit confirmation.

---

### Step 6. Create and push tag

After confirmation only:

```bash
git tag <new-tag>
git push origin <new-tag>
```

---

### Step 7. Summary

```markdown
## Release pushed

- `<new-tag>` -> pushed

**Next step:** Check CI/CD pipeline.
```

---

## Edge Cases

| Situation | Behavior |
|---|---|
| No changes | Abort with message |
| Tag already exists | Warn user, don't overwrite |
| Not on release branch | Abort, don't switch automatically |
| Behind remote | Abort, user should pull |
| User gives version with `v` prefix | Accept, don't double-prefix |
