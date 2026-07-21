<!-- markdownlint-disable -->

# Hardening Report: action-pack--cancel/v1.05

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **action-pack--cancel/v1.05** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Get previous tag' run: block directly interpolates `${{ github.ref_name }}` into a shell command string. An attacker who controls the ref name (e.g. via a crafted tag) can inject arbitrary shell commands. Offending line: `name=$(git --no-pager tag --sort=creatordate --merged ${{ github.ref_name }} | tail -1)`. Fix: move the value into an env var and double-quote it: `env: REF_NAME: ${{ github.ref_name }}` then use `"$REF_NAME"` in the script.

Locations:

- `.github/workflows/build.yml:22`

### github-env-injection (severity: high)

The 'Get previous tag' run: block writes `$name` — a value derived from `${{ github.ref_name }}` (attacker-controlled git ref) — directly to $GITHUB_ENV without the required sanitization step (`printf '%s' "$name" | tr -d '\n\r'`). A crafted ref name containing newlines could inject arbitrary environment variables into subsequent steps. Offending line: `echo "previousTag=$name" >> $GITHUB_ENV`.

Locations:

- `.github/workflows/build.yml:24`

### unpinned-uses (severity: high)

Multiple `uses:` references in build.yml use mutable version tags instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if any of those tags are moved or the referenced repositories are compromised. Unpinned references: `actions/checkout@v7`, `requarks/changelog-action@v1`, `action-pack/github-release@v2` (used twice), `action-pack/bump@v2`, `action-pack/send-mail@v1`. Each should be pinned to a full SHA, e.g. `actions/checkout@<40-char-sha> # v7`.

Locations:

- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:23`
- `.github/workflows/build.yml:31`
- `.github/workflows/build.yml:43`
- `.github/workflows/build.yml:55`
- `.github/workflows/build.yml:59`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/build.yml: (1) script-injection: moved `${{ github.ref_name }}` into an env var `REF_NAME` and used `"$REF_NAME"` in the git command; (2) github-env-injection: added `safe=$(printf '%s' "$name" | tr -d '\n\r')` sanitization before writing to $GITHUB_ENV; (3) unpinned-uses: pinned all 5 action references to full 40-char commit SHAs — actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1, requarks/changelog-action@b78a3354a01f4a1affb484b9264b506a815c46b1, action-pack/github-release@2614fdae89655edf9e0661a4f933891f2ba8d3b5 (×2), action-pack/bump@f1ef267a89c9d1ca81c6f68ae05c9b2370fca939, action-pack/send-mail@33d560748d5c56be2a180c578ba4cf92b22a4723.

