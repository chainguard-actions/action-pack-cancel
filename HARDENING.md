<!-- markdownlint-disable -->

# Hardening Report: action-pack--cancel--/v1.05

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **action-pack--cancel--/v1.05** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Get previous tag' step directly interpolates `${{ github.ref_name }}` inside a `run:` shell command string. This allows an attacker-controlled branch/tag name to inject arbitrary shell commands. Offending line: `name=$(git --no-pager tag --sort=creatordate --merged ${{ github.ref_name }} | tail -1)`

Locations:

- `.github/workflows/build.yml:24`

### github-env-injection (severity: high)

The 'Get previous tag' step writes `$name` (derived from the untrusted `${{ github.ref_name }}` value) to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline-containing ref name could inject arbitrary environment variables. Offending line: `echo "previousTag=$name" >> $GITHUB_ENV`

Locations:

- `.github/workflows/build.yml:26`

### unpinned-uses (severity: high)

All `uses:` references in build.yml use mutable tags or version strings instead of pinned full 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if those tags are moved or compromised. Failing references: `actions/checkout@v7` (line 19), `requarks/changelog-action@v1` (line 29), `action-pack/github-release@v2` (line 38), `action-pack/github-release@v2` (line 49), `action-pack/bump@v2` (line 57), `action-pack/send-mail@v1` (line 60).

Locations:

- `.github/workflows/build.yml:19`
- `.github/workflows/build.yml:29`
- `.github/workflows/build.yml:38`
- `.github/workflows/build.yml:49`
- `.github/workflows/build.yml:57`
- `.github/workflows/build.yml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings in .github/workflows/build.yml:
1. script-injection: Moved `${{ github.ref_name }}` into the step's `env:` block as REF_NAME and referenced it as "$REF_NAME" in the shell script.
2. github-env-injection: Added sanitization step `safe=$(printf '%s' "$name" | tr -d '\n\r')` before writing to $GITHUB_ENV, and quoted $GITHUB_ENV.
3. unpinned-uses: Pinned all 6 action references to full SHA hashes — actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 (v7), requarks/changelog-action@b78a3354a01f4a1affb484b9264b506a815c46b1 (v1), action-pack/github-release@2614fdae89655edf9e0661a4f933891f2ba8d3b5 (v2, used twice), action-pack/bump@f1ef267a89c9d1ca81c6f68ae05c9b2370fca939 (v2), action-pack/send-mail@33d560748d5c56be2a180c578ba4cf92b22a4723 (v1).

