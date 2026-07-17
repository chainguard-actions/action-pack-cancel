<!-- markdownlint-disable -->

# Hardening Report: action-pack--cancel/v1.02

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **action-pack--cancel/v1.02** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Get previous tag' run: block directly interpolates `${{ github.ref_name }}` into a shell command: `name=$(git --no-pager tag --sort=creatordate --merged ${{ github.ref_name }} | tail -1)`. An attacker who controls the ref name (e.g. via a crafted tag) can inject arbitrary shell commands. The expression must be passed via an env: variable and double-quoted in the shell instead.

Locations:

- `.github/workflows/build.yml:26`

### github-env-injection (severity: high)

The 'Get previous tag' run: block writes `$name` — a value derived from the untrusted `${{ github.ref_name }}` expression — directly to $GITHUB_ENV without the required sanitization step (`printf '%s' "$name" | tr -d '\n\r'`). This allows an attacker to inject arbitrary environment variables into subsequent steps by embedding newlines in the ref name.

Locations:

- `.github/workflows/build.yml:28`

### unpinned-uses (severity: high)

All five `uses:` references in build.yml use mutable tag refs instead of pinned 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if any of those tags are moved or compromised. Failing references: `actions/checkout@v4` (line 20), `requarks/changelog-action@v1` (line 32), `action-pack/github-release@v2` (line 40), `action-pack/github-release@v2` (line 49), `action-pack/bump@v2` (line 58).

Locations:

- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:32`
- `.github/workflows/build.yml:40`
- `.github/workflows/build.yml:49`
- `.github/workflows/build.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/build.yml: (1) script-injection: moved `${{ github.ref_name }}` into an env: block as REF_NAME and double-quoted it in the shell command; (2) github-env-injection: sanitized the derived $name value with `printf '%s' "$name" | tr -d '\n\r'` before writing to $GITHUB_ENV; (3) unpinned-uses: pinned all five uses references to full 40-char SHAs — actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5, requarks/changelog-action@b78a3354a01f4a1affb484b9264b506a815c46b1, action-pack/github-release@2614fdae89655edf9e0661a4f933891f2ba8d3b5 (×2), action-pack/bump@f1ef267a89c9d1ca81c6f68ae05c9b2370fca939.

