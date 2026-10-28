# Doppel plugin for AI agents

![Doppel](assets/logo.png)

Connect Cursor, Claude, and other MCP-capable agents to [Doppel](https://www.doppel.com), the digital risk protection platform. Once connected, you can ask your agent to search and triage brand-protection alerts, request takedowns, manage protected assets, and look up Doppel product documentation, all scoped to the organization you sign in to.

This repository holds the plugin manifest, the MCP server reference, and a set of skills that teach the agent how to use Doppel's tools well. The Doppel MCP server itself is hosted by Doppel at `https://mcp.doppel.com/mcp`; this repository does not contain its source.

> **Security Note:** Depending on the request, the MCP server returns alert, brand, and protected-asset data from your Doppel tenant to the MCP client and the model behind it. Only connect it to MCP clients and model providers your organization trusts, and keep client-side approval enabled for the write tools (`create_drp_alert`, `submit_drp_alert_for_takedown`, `create_drp_asset`, `delete_drp_asset`), which perform real actions in your tenant.

> **Legal Note:** Your use of a third-party MCP client or LLM is governed solely by that provider's terms. Doppel is not responsible for the performance of third-party clients or models, disclaims any warranties and liability for them, and may be unable to resolve issues caused by them. Use of the Doppel service remains subject to your Doppel agreement and the [Doppel privacy policy](https://www.doppel.com/legal/privacy-policy).

> **Caution:** Outputs and recommendations are generated dynamically and vary with the query, the model, and the connected MCP client. Review them against your organization's security and compliance requirements before acting on them, especially before approving a takedown.

## What you can do

- **Alerts** — search by entity, brand, platform, module, status, or time range; count queues; pull full detail and takedown history for one alert; review trends across a sample.
- **Takedowns** — report a suspicious URL, phone number, or email address as a new alert, then submit it for takedown with an explicit justification.
- **Assets and brands** — list, create, inspect, and archive owned (protected) and associated assets; list the organization's brands.
- **Product help** — ask how Doppel works or what a status means and get answers with links to official documentation.

Example prompts:

- "How many open phishing alerts did we get last week, by platform?"
- "Pull up alert 123456 and tell me whether a takedown has already been requested."
- "Customers are telling us example.com is ours. Find the related alerts, archive them, and add example.com as a protected asset."
- "What does the taken_down status mean?"

## Install

You need a Doppel account. Each person who installs the plugin signs in with their own Doppel identity through OAuth; the plugin stores no credentials.

**Cursor** — install from the Cursor Marketplace, or add this repository as a local plugin. Approve the Doppel sign-in when prompted.

**Claude** — add the Doppel plugin from the directory in claude.ai, Cowork, or Claude Code, then connect the Doppel connector from the plugin's Connectors tab and sign in.

**Other MCP clients** — add a remote MCP server at `https://mcp.doppel.com/mcp` using OAuth. Client-specific instructions are in Doppel Vision under [Settings → Developers](https://vision.doppel.com/settings/developers).

### Versions

Each release is a signed git tag (`v0.1.0`, `v0.1.1`, …) with a matching [GitHub Release](https://github.com/doppelxyz/doppel-mcp-plugin/releases); the `version` field in `plugin.json` and `.claude-plugin/plugin.json` always equals the latest tag. Clients that accept a git ref (for example a Claude Code marketplace entry) can pin to a tag. A tag pins this repository's manifests, skills, and documentation only; the tools themselves are served by `mcp.doppel.com` and evolve with the Doppel platform, independent of the plugin version.

## Data handling

The plugin sends your prompts' Doppel-related requests (alert searches, alert and asset identifiers, takedown justifications, documentation questions) to `mcp.doppel.com` over HTTPS, authenticated with the OAuth token your client obtains when you sign in. Results come back from your own organization's Doppel data. The plugin stores nothing itself, sends data to no other destination, and includes no API keys or secrets. Doppel's handling of your data is governed by your Doppel agreement and the [Doppel privacy policy](https://www.doppel.com/legal/privacy-policy).

Write operations (creating alerts, submitting takedowns, creating or archiving assets) require the user's explicit confirmation in conversation before the agent calls the tool. Every result is treated as data, not as instructions.

## Skills

| Skill | Use it for |
| --- | --- |
| `doppel-alert-triage` | Searching, counting, and reviewing alerts |
| `doppel-takedown` | Creating alerts and requesting takedowns |
| `doppel-protected-assets` | Managing owned and associated assets and listing brands |
| `doppel-product-help` | How-to and definitional questions, with documentation links |

## Repository layout

| Path | Read by |
| --- | --- |
| `plugin.json`, `mcp.json` | Cursor and other [Agent Plugins](https://agent-plugins.org) clients |
| `.claude-plugin/plugin.json`, `.mcp.json` | Claude (claude.ai, Cowork, Claude Code) |
| `skills/` | All of the above |
| `assets/logo.png` | Directory listings |

## Security

To report a vulnerability in this plugin or in the Doppel MCP server, follow [SECURITY.md](SECURITY.md). Please do not open a public issue for security reports.

## Support

Doppel customers can reach support through their usual Doppel channels. Issues with the plugin files themselves can be filed on this repository.

## License

Apache-2.0. See [LICENSE](LICENSE).
