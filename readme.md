<h1 align="center">Cancel Workflow<br />
<div align="center">
  
  [![Build](https://github.com/action-pack/cancel/workflows/Build/badge.svg)](https://github.com/action-pack/cancel/)
  [![Version](https://img.shields.io/github/v/tag/action-pack/cancel?label=version&sort=semver&color=066da5)](https://github.com/marketplace/actions/cancel-workflow)
  [![Size](https://img.shields.io/github/languages/code-size/action-pack/cancel?label=size&color=066da5)](https://github.com/action-pack/cancel/)
  
</div></h1>

This action cancels the currently running workflow.

## Usage

```yaml
permissions:
      actions: write

    steps:
      - name: Cancelling workflow
        uses: action-pack/cancel@v1
```

In combination with the ```if``` statement, this allows you to cancel the workflow conditionally.

## Privacy

This Action contacts Chainguard's licensing server to verify authorization. Connection metadata (IP address, GitHub repository identifier, timestamp, and any metadata encoded in the auth token) is transmitted to Chainguard, Inc. even if authorization is denied in accordance with our [Privacy Notice](https://www.chainguard.dev/legal/privacy-notice)
