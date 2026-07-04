<!-- markdownlint-disable -->

# Hardening Report: action-pack--cancel--/v1.06

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **action-pack--cancel--/v1.06** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in .github/workflows/build.yml use mutable version tags instead of pinned 40-character SHA commits, making the workflow vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references: `actions/checkout@v7` (line 20), `requarks/changelog-action@v1` (line 32), `action-pack/github-release@v2` (line 40), `action-pack/github-release@v2` (line 50), `action-pack/bump@v2` (line 58), `action-pack/send-mail@v1` (line 61).

Locations:

- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:32`
- `.github/workflows/build.yml:40`
- `.github/workflows/build.yml:50`
- `.github/workflows/build.yml:58`
- `.github/workflows/build.yml:61`

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Get previous tag' step directly interpolates `${{ github.ref_name }}` inside a shell command string: `name=$(git --no-pager tag --sort=creatordate --merged ${{ github.ref_name }} | tail -1)`. The `github.ref_name` value is substituted into the shell command before the shell parses it, allowing an attacker to inject arbitrary shell commands via a crafted branch or tag name.

Locations:

- `.github/workflows/build.yml:25`

### github-env-injection (severity: high)

The 'Get previous tag' step writes the variable `$name` — derived from `${{ github.ref_name }}` via command substitution — to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker controlling the branch/tag name could inject newlines to set arbitrary environment variables for subsequent steps. Offending line: `echo "previousTag=$name" >> $GITHUB_ENV`.

Locations:

- `.github/workflows/build.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in .github/workflows/build.yml:
1. unpinned-uses: Pinned all 6 uses: references to full commit SHAs — actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 (v7), requarks/changelog-action@b78a3354a01f4a1affb484b9264b506a815c46b1 (v1), action-pack/github-release@2614fdae89655edf9e0661a4f933891f2ba8d3b5 (v2, used twice), action-pack/bump@f1ef267a89c9d1ca81c6f68ae05c9b2370fca939 (v2), action-pack/send-mail@33d560748d5c56be2a180c578ba4cf92b22a4723 (v1).
2. script-injection: Moved ${{ github.ref_name }} out of the run: shell string into an env: block as REF_NAME, then referenced as "$REF_NAME" in the git command.
3. github-env-injection: Added sanitization step using `safe=$(printf '%s' "$name" | tr -d '\n\r')` before writing to $GITHUB_ENV, and quoted $GITHUB_ENV as "$GITHUB_ENV".

