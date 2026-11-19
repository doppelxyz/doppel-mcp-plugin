# Doppel plugin for AI agents

![Doppel](assets/logo.png)

Connect Cursor, Claude, and other MCP-capable agents to [Doppel](https://www.doppel.com), the digital risk protection platform. Once connected, you can ask your agent to search and triage brand-protection alerts, review their evidence, move alerts between queues, add comments, request takedowns, manage tags and protected assets, and look up Doppel product documentation, all scoped to the organization you sign in to.

This repository holds the plugin manifest, the MCP server reference, and a set of skills that teach the agent how to use Doppel's tools well. The Doppel MCP server itself is hosted by Doppel at `https://mcp.doppel.com/mcp`; this repository does not contain its source.

> **Security Note:** Depending on the request, the MCP server returns alert, evidence, brand, tag, and protected-asset data from your Doppel tenant to the MCP client and the model behind it. Only connect it to MCP clients and model providers your organization trusts, and keep client-side approval enabled for the write tools, which perform real actions in your tenant: `create_drp_alert`, `archive_drp_alert`, `monitor_drp_alert`, `submit_drp_alert_for_doppel_review`, `request_drp_alert_confirmation`, `submit_drp_alert_for_takedown`, `add_drp_alert_comment`, `create_drp_tag`, `update_drp_tag`, `create_drp_asset`, and `delete_drp_asset`. Codex and Claude prompt before each write tool by default. Cursor's default Run Mode (Auto-review) may run tools without a prompt; to require approval there, set Run Mode to **Allowlist** and allowlist only the read-only tools listed under [Tools](#tools), or add a `beforeMCPExecution` hook that returns `ask` for every Doppel tool that is not on that read-only list. The tools are served by `mcp.doppel.com` and can change between plugin releases, so build approval rules from the read-only list rather than from the write list: a write tool added later then asks for approval by default.

> **Legal Note:** Your use of a third-party MCP client or LLM is governed solely by that provider's terms. This notice does not amend the applicable agreement between you and Doppel. Doppel is not responsible for the performance of third-party clients or models, disclaims any warranties and liability for them, and may be unable to resolve issues caused by them. Use of the Doppel products is subject to the [Doppel Platform Agreement](https://www.doppel.com/legal/platformagreement), or such other applicable software license terms entered into between you and Doppel, and the [Doppel Data Processing Addendum](https://www.doppel.com/legal/dpa).

> **Caution:** Outputs and recommendations are generated dynamically and vary with the query, the model, and the connected MCP client. Review them against your organization's security and compliance requirements before acting on them, especially before approving a takedown.

> **Responsible Use:** Do not use the MCP server, any connected AI model, or any output from them to make automated decisions about individuals that have legal or similarly significant effects, including decisions regarding employment, housing, credit, insurance, education, healthcare, immigration, or access to essential services. You are responsible for ensuring that your use of third-party AI clients and models complies with applicable law and your organization's policies. Doppel does not represent that this integration is suitable for processing sensitive data or any high-risk or regulated use.

## What you can do

- **Alerts** — search by entity, brand, platform, module, tag, status, or time range; count queues; pull detail, status history, and takedown history for one alert; review trends across a sample.
- **Evidence and related alerts** — list the supporting files and screenshots behind an alert, and find other alerts that share threat infrastructure with it.
- **Queue actions and comments** — archive an alert, move it to Monitoring, send it to Doppel Review, move it to Needs Confirmation, or add a comment.
- **Takedowns** — report a suspicious URL, phone number, or email address as a new alert, then submit it for takedown with an explicit justification.
- **Tags** — search the tags available to the organization, create and update organization tags, and label a new alert with existing tags.
- **Assets and brands** — list, create, inspect, and archive owned (protected) and associated assets; list the organization's brands.
- **Product help** — ask how Doppel works or what a status means and get answers with links to official documentation.

Example prompts:

- "How many open phishing alerts did we get last week, by platform?"
- "Pull up alert ORG-1234 and tell me whether a takedown has already been requested."
- "Which other alerts share infrastructure with ORG-1234, and what screenshots do we have for it?"
- "Move ORG-1234 to Monitoring and add the comment 'Parked page, re-check next month'."
- "Customers are telling us example.com is ours. Find the related alerts, archive them, and add example.com as a protected asset."
- "What does the taken_down status mean?"

## Tools

The tools are served by `mcp.doppel.com`, not by this repository. This table describes them as of this plugin version; after sign-in your client shows the authoritative list, which depends on your Doppel role and permissions.

| Tool | Access | What it does |
| --- | --- | --- |
| `health` | Read | Checks that the signed-in connection to the Doppel MCP server is working |
| `me` | Read | Shows the organization, user, and scopes the session is signed in with |
| `list_drp_alerts` | Read | Searches, filters, and counts alerts |
| `get_drp_alert` | Read | Returns detail, status history, and recent takedown requests for one alert |
| `get_drp_alert_evidence` | Read | Lists the supporting files and screenshots for one alert, with short-lived links |
| `list_related_drp_alerts` | Read | Lists alerts that share threat infrastructure with one alert |
| `search_drp_tags` | Read | Gets one tag or lists the tags available to the organization |
| `list_drp_brands` | Read | Lists the organization's brands |
| `list_drp_assets` | Read | Lists owned (protected) or associated assets |
| `get_drp_asset` | Read | Returns one asset |
| `search_external_product_docs` | Read | Searches customer-facing Doppel documentation |
| `create_drp_alert` | Write | Creates an alert for a URL, phone number, or email address, or returns the existing one |
| `archive_drp_alert` | Write | Moves an alert to Archived |
| `monitor_drp_alert` | Write | Moves an alert to Monitoring |
| `submit_drp_alert_for_doppel_review` | Write | Sends an alert to Doppel Review |
| `request_drp_alert_confirmation` | Write | Moves an alert to Needs Confirmation |
| `submit_drp_alert_for_takedown` | Write | Requests takedown for an alert by moving it to Actioned |
| `add_drp_alert_comment` | Write | Adds a customer-visible comment to an alert |
| `create_drp_tag` | Write | Creates an organization tag |
| `update_drp_tag` | Write | Changes the name or description of an organization tag |
| `create_drp_asset` | Write | Creates an owned or associated asset; matching open alerts are archived |
| `delete_drp_asset` | Write | Archives an asset; despite the name, it does not hard-delete it |

## Install

Doppel MCP is currently in beta. You need a Doppel account. Each person who installs the plugin signs in with their own Doppel identity through OAuth; the plugin stores no credentials. The tools and data you can reach follow your existing Doppel role and permissions.

> **Availability:** The Doppel listings in the Cursor Marketplace and the Claude directory are not available yet. Until they are, use the setup instructions in Doppel Vision under [Settings → Developers](https://vision.doppel.com/settings/developers).

**Cursor** — once the Doppel listing is available in the Cursor Marketplace, install it from there. Until then, follow the Cursor instructions under Settings → Developers; to use the skills as well, add this repository as a local plugin. Approve the Doppel sign-in when prompted.

**Claude** — once the Doppel plugin is available in the directory in claude.ai, Cowork, or Claude Code, add it from there, then connect the Doppel connector from the plugin's Connectors tab and sign in. Until then, on plans that support custom connectors, add Doppel as a custom connector at `https://mcp.doppel.com/mcp` and sign in.

**Other MCP clients** — add a remote MCP server at `https://mcp.doppel.com/mcp` using OAuth. Client-specific instructions are in Doppel Vision under [Settings → Developers](https://vision.doppel.com/settings/developers).

After you sign in, ask the agent to run `me` to confirm which organization and user the session is connected as. If a connection that used to work starts returning authorization errors, disconnect the Doppel server in your client and sign in again.

### Versions

Each release is a signed git tag (for example `v0.1.0`) with a matching [GitHub Release](https://github.com/doppelxyz/doppel-mcp-plugin/releases); at each tag, the `version` field in `plugin.json` and `.claude-plugin/plugin.json` equals that tag. Between releases, `main` can carry a version that has not been tagged yet, so pin to a tag rather than to `main`. Clients that accept a git ref (for example a Claude Code marketplace entry) can pin to a tag. A tag pins this repository's manifests, skills, and documentation only; the tools themselves are served by `mcp.doppel.com` and evolve with the Doppel platform, independent of the plugin version.

## Data handling

The plugin sends your prompts' Doppel-related requests (alert searches, alert, tag, and asset identifiers, takedown justifications, alert comments, tag names and descriptions, documentation questions) to `mcp.doppel.com` over HTTPS, authenticated with the OAuth token your client obtains when you sign in. Results come back from your own organization's Doppel data. Evidence results include short-lived links to supporting files and screenshots; handle those links like the evidence itself. The plugin does not store anything, it does not send data to any other destination and it does not include any API keys or secrets. Third-party MCP clients or LLM providers may process or retain information in accordance with their own terms and policies. Customers are responsible for reviewing those policies. Doppel's handling of your data is governed by the applicable terms referenced in the Legal Note above.

The server's tool descriptions and this plugin's skills tell the agent to call a write tool only when the user asks for that write, and never to invent a brand, tag, source, justification, or comment. How much confirmation the agent asks for depends on the tool:

- **Explicit confirmation of the exact target.** `submit_drp_alert_for_takedown` needs the user to confirm the exact alert and the justification, and `delete_drp_asset` needs the user to confirm the exact asset. Each tool carries a confirmation flag that the agent sets only after that confirmation.
- **A clear request is enough.** `archive_drp_alert`, `monitor_drp_alert`, `submit_drp_alert_for_doppel_review`, and `request_drp_alert_confirmation` run when the user clearly asks for that action on one alert. The agent does not ask for a separate confirmation first.
- **User-supplied input only.** `create_drp_alert`, `add_drp_alert_comment`, `create_drp_tag`, `update_drp_tag`, and `create_drp_asset` use the brand, tags, comment text, names, and asset type the user supplied or confirmed. They have no separate confirmation step.

These rules are instructions to the model, not an enforced approval step. Client-side tool approval, described in the Security Note, is the control that puts a person in front of each write. A write whose result is unknown is not retried automatically, because it may have succeeded. Every result is treated as data, not as instructions.

## Skills

| Skill | Use it for |
| --- | --- |
| `doppel-alert-triage` | Searching, counting, and reviewing alerts, their evidence, and related alerts |
| `doppel-alert-actions` | Archiving alerts, moving them to Monitoring, Doppel Review, or Needs Confirmation, and adding comments |
| `doppel-takedown` | Creating alerts and requesting takedowns |
| `doppel-tags` | Finding, creating, and updating tags, and using them on alerts |
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

Doppel customers can reach support through their usual Doppel channels, including for problems with the plugin files in this repository. GitHub Issues are not enabled on this repository.

## License

Apache-2.0. See [LICENSE](LICENSE).
