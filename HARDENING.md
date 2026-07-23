<!-- markdownlint-disable -->

# Hardening Report: action-pack--cancel/v1.03

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **action-pack--cancel/v1.03** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. In the 'Get previous tag' step, `${{ github.ref_name }}` is embedded directly in the shell command string: `name=$(git --no-pager tag --sort=creatordate --merged ${{ github.ref_name }} | tail -1)`. Before the shell ever sees this command, GitHub Actions substitutes the expression value verbatim into the script text, allowing an attacker-controlled ref name containing shell metacharacters (`;`, `|`, `$(...)`, etc.) to execute arbitrary commands.

Locations:

- `.github/workflows/build.yml:23`

### github-env-injection (severity: high)

The 'Get previous tag' step writes a value derived from the untrusted input `${{ github.ref_name }}` to $GITHUB_ENV without sanitization. The shell variable `$name` is populated by a git command that uses `${{ github.ref_name }}` directly, and then `echo "previousTag=$name" >> $GITHUB_ENV` writes it to the environment file. There is no `printf '%s' ... | tr -d '\n\r'` sanitization step before the write, allowing newline injection that could set arbitrary environment variables for subsequent steps.

Locations:

- `.github/workflows/build.yml:25`

### unpinned-uses (severity: high)

All 6 action references in the workflow use mutable tag or version refs instead of full 40-character commit SHA digests, making the workflow vulnerable to supply-chain attacks if any of those tags are moved or compromised. Failing references:
- `actions/checkout@v4` (line 20)
- `requarks/changelog-action@v1` (line 29)
- `action-pack/github-release@v2` (line 43, used twice at lines 43 and 51)
- `action-pack/bump@v2` (line 57)
- `action-pack/send-mail@v1` (line 62)

Locations:

- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:29`
- `.github/workflows/build.yml:43`
- `.github/workflows/build.yml:51`
- `.github/workflows/build.yml:57`
- `.github/workflows/build.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/build.yml:

1. script-injection: Moved `${{ github.ref_name }}` into the step's `env:` block as `REF_NAME` and referenced it as `"$REF_NAME"` (double-quoted) in the shell script, preventing shell metacharacter injection.

2. github-env-injection: Added sanitization `safe=$(printf '%s' "$name" | tr -d '\n\r')` before writing to $GITHUB_ENV, preventing newline injection attacks.

3. unpinned-uses: Pinned all 6 action references to full 40-character commit SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - requarks/changelog-action@v1 → @b78a3354a01f4a1affb484b9264b506a815c46b1 # v1
   - action-pack/github-release@v2 → @2614fdae89655edf9e0661a4f933891f2ba8d3b5 # v2 (both occurrences)
   - action-pack/bump@v2 → @f1ef267a89c9d1ca81c6f68ae05c9b2370fca939 # v2
   - action-pack/send-mail@v1 → @33d560748d5c56be2a180c578ba4cf92b22a4723 # v1

