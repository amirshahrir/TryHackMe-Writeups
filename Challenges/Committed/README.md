```
Room         : Comitted
Platform     : TryHackMe
Difficulty   : Easy
Date         : 16/06/2026
Author       : Amir Shahrir
Tools Used   : Terminal / Linux
Status       : Completed
```

---

### Executive Summary

A simulated insider scenario in which a developer accidentally committed a plaintext password to a Git repository. The task was to locate the exposed credential within the repository's history. Initial review of the default branch yielded no findings; enumeration of all branches revealed a non-default branch named dbint containing the sensitive commit. The plaintext password was recovered from the commit diff using standard Git inspection commands.

---

### Objectives

- Inspect the provided Git repository for accidentally committed sensitive data
- Enumerate all branches to ensure no findings were isolated to non-default branches
- Identify the specific commit responsible for the credential exposure
- Recover the flag from the commit diff

---

### Environment & Tools

| Component         | Details                                |
| ----------------- | -------------------------------------- |
| Platform          | TryHackMe in-browser VM (Ubuntu)       |
| Working Directory | `/home/ubuntu/commited`                |
| Tools             | Git, Linux Terminal                    |
---

### Methodology

#### Phase 1 — Repository Inspection

The provided archive was extracted and the directory contents inspected:

```bash
unzip committed.zip
ls -la
```

Running `ls -la` rather than a plain `ls` was deliberate — the `-a` flag surfaces hidden entries that a standard listing omits. The output revealed a hidden `.git` directory alongside the visible project files, confirming the archive contained a full Git repository. This is consequential: a `.git` directory holds the repository's entire object store, including the complete history of every file ever tracked. Removing a sensitive file from the working tree does not remove it from this history.

The commit history on the active branch was reviewed using the condensed format for easier reference:

```bash
git log --oneline
```

The short commit hashes returned by `--oneline` were used to inspect each commit individually:

```bash
git show <commit_id>
```

Each commit on master was examined in turn. None of the diffs contained credential-related content, and no commit message suggested a sensitive change. Master was a dead end.

---

#### Phase 2 — Branch Enumeration and Commit Analysis

The absence of any finding on master raised the possibility that the sensitive commit existed on a separate branch. All branches were enumerated:

```bash
git branch -a
```

Two branches were returned: `master` and `dbint`. The working tree was switched to the `dbint` branch for inspection:

```bash
git checkout dbint
git log --oneline
```

The commit list included a commit with the message `"ops"` — an informal shorthand that stood out immediately as a likely acknowledgement of a mistake. That commit was inspected:

```bash
git show <commit_id>
```

The diff confirmed it: a plaintext password had been hardcoded directly into the source code and committed to the `dbint` branch. The flag was recovered from this output: `flag{[REDACTED]}`


---

### Lessons Learned

- GIT files are by default hidden in desktop, in Terminal after extracting the zip file using `ls -la` reveals the git file.
- investigate the git files uses GIT commands: `git log`, `git log --oneline`, `git show <commit_id>`.
- While being on the master branch, turns out theres another branch, we use `git branch -a` to show available branches

---

_Write-up by Amir Shahrir | https://github.com/amirshahrir | Completed: [Date]_
_Note: This write-up is for educational purposes. All activities were conducted in an isolated, legal lab environment provided by TryHackMe._

```

```

