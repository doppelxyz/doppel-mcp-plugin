# Security policy

## Reporting a vulnerability

Please report security issues privately to **security@doppel.com**. Do not open a public GitHub issue or pull request for a security report.

Include what you found, how to reproduce it, and the impact you believe it has. We will acknowledge your report, keep you informed as we investigate, and credit you if you would like once a fix is released.

You can also use GitHub's private vulnerability reporting from the **Security** tab of this repository.

## Scope

This repository contains only plugin manifests, an MCP server reference, skills (markdown instructions), and a logo. It runs no code of its own and holds no credentials.

- Issues in these files (for example a manifest pointing at the wrong host, or a skill that instructs an agent unsafely) are in scope here.
- Issues in the hosted Doppel MCP server at `mcp.doppel.com`, the Doppel API, or Doppel Vision should be reported to the same address; they are handled by the Doppel security team, not through this repository.
- Issues in third-party MCP clients or models (Cursor, Claude, ChatGPT, and others) should go to those vendors.

## Supported versions

Only the latest release on the default branch is supported. Earlier versions receive no fixes.

## Rules for contributors

- Never commit credentials, API keys, tokens, or reviewer test-account details. Secret scanning with push protection is enabled on this repository.
- Every change to the default branch goes through a pull request reviewed by a code owner.
- The MCP endpoint in `mcp.json` and `.mcp.json` must stay `https://mcp.doppel.com/mcp`. Changing it is a security-relevant change and needs explicit review.
