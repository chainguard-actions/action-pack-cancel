<!-- markdownlint-disable -->

# Hardening Report: action-pack--cancel/v1.04

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **action-pack--cancel/v1.04** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The `run:` block in the 'Get previous tag' step directly interpolates `${{ github.ref_name }}` into a shell command. An attacker who controls the branch or tag name (e.g. via a crafted ref) can inject arbitrary shell commands. Offending line: `name=$(git --no-pager tag --sort=creatordate --merged ${{ github.ref_name }} | tail -1)`. Fix: move the value into an `env:` variable and double-quote it in the shell script.

Locations:

- `.github/workflows/build.yml:25`

### github-env-injection (severity: high)

The 'Get previous tag' step writes the variable `$name` — which is derived from the attacker-controllable `${{ github.ref_name }}` (via git tag output) — directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' "$name" | tr -d '\n\r'`). A newline embedded in a crafted ref name could inject arbitrary environment variables into subsequent steps. Offending line: `echo "previousTag=$name" >> $GITHUB_ENV`.

Locations:

- `.github/workflows/build.yml:27`

### unpinned-uses (severity: high)

Multiple `uses:` references in build.yml are pinned to mutable tags rather than immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if those tags are moved or compromised. Failing references: `actions/checkout@v6` (line 19), `requarks/changelog-action@v1` (line 31), `action-pack/github-release@v2` (line 38), `action-pack/github-release@v2` (line 47), `action-pack/bump@v2` (line 55), `action-pack/send-mail@v1` (line 59). All should be pinned to full SHA digests, e.g. `actions/checkout@<40-hex-char-sha> # v6`.

Locations:

- `.github/workflows/build.yml:19`
- `.github/workflows/build.yml:31`
- `.github/workflows/build.yml:38`
- `.github/workflows/build.yml:47`
- `.github/workflows/build.yml:55`
- `.github/workflows/build.yml:59`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/build.yml:
1. script-injection: Moved `${{ github.ref_name }}` into an `env:` block as `REF_NAME` and referenced it as `"$REF_NAME"` in the shell script.
2. github-env-injection: Added `safe=$(printf '%s' "$name" | tr -d '\n\r')` sanitization step before writing to GITHUB_ENV to prevent newline injection.
3. unpinned-uses: Pinned all 6 action references to full 40-character commit SHAs: actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 # v6, requarks/changelog-action@b78a3354a01f4a1affb484b9264b506a815c46b1 # v1, action-pack/github-release@2614fdae89655edf9e0661a4f933891f2ba8d3b5 # v2 (×2), action-pack/bump@f1ef267a89c9d1ca81c6f68ae05c9b2370fca939 # v2, action-pack/send-mail@33d560748d5c56be2a180c578ba4cf92b22a4723 # v1.

