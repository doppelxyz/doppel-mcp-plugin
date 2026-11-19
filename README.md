# Doppel plugin for AI agents

![Doppel](assets/logo.png)

Connect Cursor, Claude, and other MCP-capable agents to [Doppel](https://www.doppel.com), the digital risk protection platform. Once connected, you can ask your agent to search and triage brand-protection alerts, review their evidence, move alerts between queues, add comments, request takedowns, manage tags and protected assets, and look up Doppel product documentation, all scoped to the organization you sign in to.

This repository holds the plugin manifests and the MCP server reference. Installing the plugin connects your client to the Doppel MCP server and adds nothing else: the tools, and the instructions that tell the agent how to use them, come from the server. The Doppel MCP server itself is hosted by Doppel at `https://mcp.doppel.com/mcp`; this repository does not contain its source.

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

The tools are served by `mcp.doppel.com`, not by this repository. This table describes them as of this plugin version; after sign-in your client shows the authoritative list. Every customer account is shown the same tools. Your Doppel role and permissions decide what a call can do, so a tool that is listed can still be refused; see [Limits and side effects](#limits-and-side-effects).

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

### Limits and side effects

- **Sign-in.** Every tool needs a signed-in session, including `health`. `health` fails until you have signed in, needs no particular permission, and checks only the connection, not the health of a Doppel product.
- **Organization.** Each sign-in is bound to one organization, which every call uses; no tool takes an organization argument. If your account belongs to more than one organization, sign-in asks you to choose one from an organization picker. If you are not offered a choice, the sign-in uses your active organization in Doppel. To work in another organization, sign out of the Doppel connection, sign in again, and choose it; if no picker appears, switch your active organization in Doppel before you sign in again. Refreshing an existing connection does not change its organization. `me` shows which organization a session is using.
- **Consent and availability.** A listed tool does not always run. When a call needs a permission that the connection has not been granted yet, the server asks your client for additional consent before anything runs; giving consent does not change your Doppel role. Doppel can also switch off data access for the server. While it is off, `health` and the tool list keep working and every other tool, including `me`, is refused.
- **Alert search.** By default `list_drp_alerts` searches what the Brand Protection tab in Doppel Vision shows, newest first; Executive Protection and Dark Web alerts are separate filters. A page holds up to 20 alerts with summary fields (10 by default), or up to 5 with full detail, and each response carries the total number of matches. A search can be paged through 101 pages at most, so it returns no more than 2,020 alerts with summary fields or 505 with full detail, however many match; narrow the filters or the time range to reach the rest. The server can also refuse a very broad search and ask for the time range to be split. The status `taken_down` means Resolved. Alert results carry no reliable false-positive label.
- **Alert detail.** `get_drp_alert` returns up to 20 recent status-history entries and the 25 most recent takedown requests. It does not return screenshot links; `get_drp_alert_evidence` does. `list_related_drp_alerts` returns at most 100 alert summaries, newest first.
- **New alerts.** `create_drp_alert` needs the exact brand. If an alert already exists for the same entity and brand, the tool returns that alert unchanged and does not apply the tags or source you asked for; an existing alert under another brand is rejected. Tag names that do not exist are ignored without notice, and an unknown source is rejected. The tool cannot attach files or screenshots, or set severity or queue.
- **Queue actions.** `archive_drp_alert`, `monitor_drp_alert`, `submit_drp_alert_for_doppel_review`, and `request_drp_alert_confirmation` each move one alert per call, with an optional comment, and change nothing else on it. Doppel's queue policy can reject a move.
- **Takedowns.** `submit_drp_alert_for_takedown` works on an existing alert and moves it to Actioned. An accepted request means Doppel accepted it, not that a platform has received it or removed the content. Dark-web content cannot be taken down. A takedown is also refused when the alert has to go through Needs Confirmation first, when a trial organization has no takedown credits left, when the alert type does not support takedown, and when the alert's current queue does not allow the move.
- **Comments.** A comment on an alert that customers cannot see yet moves that alert into Doppel Review.
- **Tags.** Organization tags can be created and updated; global tags cannot be changed. A tag name is 1 to 15 characters with no comma, and a description is required. No tool deletes a tag, or adds a tag to or removes a tag from an existing alert.
- **Assets.** Owned assets, which Doppel Vision calls protected assets, add detection coverage. Associated assets suppress matching alerts without adding coverage. Creating either type needs a brand and archives the matching open alerts. An asset archived with `delete_drp_asset` leaves the active lists and can still be read with `get_drp_asset`.
- **Documentation search.** `search_external_product_docs` returns up to five ranked excerpts from customer-facing documentation for the products available to your organization, not a complete specification.

## Install

Doppel MCP is currently in beta, and access is given per organization. You need a Doppel account in an organization that has access. If your organization does not have it, Settings → Developers does not show the MCP setup and, after sign-in, the tools that read or change your organization's data are refused; ask through your usual Doppel channels to have it turned on. Each person who installs the plugin signs in with their own Doppel identity through OAuth; the plugin stores no credentials. What a tool can read or change follows your existing Doppel role and permissions.

> **Availability:** The Doppel listings in the Cursor Marketplace and the Claude directory are not available yet. Until they are, use the setup instructions in Doppel Vision under [Settings → Developers](https://vision.doppel.com/settings/developers).

**Cursor** — follow the Cursor instructions under Settings → Developers, and approve the Doppel sign-in when prompted. Cursor signs in with connection details that Doppel gives you there. The `mcp.json` in this repository carries only the server address, so installing this plugin on its own, including from the Cursor Marketplace once a Doppel listing is available, may not be enough for Cursor to sign in.

**Claude** — once the Doppel plugin is available in the directory in claude.ai, Cowork, or Claude Code, add it from there, then connect the Doppel connector from the plugin's Connectors tab and sign in. Until then, on plans that support custom connectors, add Doppel as a custom connector at `https://mcp.doppel.com/mcp` and sign in.

**Other MCP clients** — only MCP clients that Doppel supports can sign in. The server has no dynamic client registration, so a client cannot register itself, and adding `https://mcp.doppel.com/mcp` to a client that Doppel does not support fails at sign-in. Setup instructions for the supported clients are in Doppel Vision under [Settings → Developers](https://vision.doppel.com/settings/developers).

After you sign in, ask the agent to run `me` to confirm which organization and user the session is connected as. A sign-in is time-limited. Expect to sign in again roughly once a day, or sooner when the connection has sat idle for several hours; Doppel can change these limits. If a connection that used to work starts returning authorization errors, disconnect the Doppel server in your client and sign in again.

### Versions

Each release is a signed git tag (for example `v0.1.0`) with a matching [GitHub Release](https://github.com/doppelxyz/doppel-mcp-plugin/releases); at each tag, the `version` field in `plugin.json` and `.claude-plugin/plugin.json` equals that tag. Between releases, `main` can carry a version that has not been tagged yet, so pin to a tag rather than to `main`. Clients that accept a git ref (for example a Claude Code marketplace entry) can pin to a tag. A tag pins this repository's manifests and documentation only; the tools themselves are served by `mcp.doppel.com` and evolve with the Doppel platform, independent of the plugin version.

## Data handling

The plugin sends your prompts' Doppel-related requests (alert searches, alert, tag, and asset identifiers, new alert entities, brand names, asset values, takedown justifications, alert comments, tag names and descriptions, documentation questions) to `mcp.doppel.com` over HTTPS, authenticated with the OAuth token your client obtains when you sign in. Results come back from your own organization's Doppel data. Evidence results include short-lived links to supporting files and screenshots; handle those links like the evidence itself. The plugin does not store anything, it does not send data to any other destination and it does not include any API keys or secrets. Third-party MCP clients or LLM providers may process or retain information in accordance with their own terms and policies. Customers are responsible for reviewing those policies. Doppel's handling of your data is governed by the applicable terms referenced in the Legal Note above.

The server's instructions and tool descriptions tell the agent to call a write tool only when the user asks for that write, and never to invent a brand, tag, source, justification, or comment. How much confirmation the agent asks for depends on the tool:

- **Explicit confirmation of the exact target.** `submit_drp_alert_for_takedown` needs the user to confirm the exact alert and the justification, and `delete_drp_asset` needs the user to confirm the exact asset. Each tool carries a confirmation flag that the agent sets only after that confirmation.
- **A clear request is enough.** `archive_drp_alert`, `monitor_drp_alert`, `submit_drp_alert_for_doppel_review`, and `request_drp_alert_confirmation` run when the user clearly asks for that action on one alert. The agent does not ask for a separate confirmation first.
- **User-supplied input only.** `create_drp_alert`, `add_drp_alert_comment`, `create_drp_tag`, `update_drp_tag`, and `create_drp_asset` use the brand, tags, comment text, names, and asset type the user supplied or confirmed. They have no separate confirmation step.

These rules are instructions to the model, not an enforced approval step. Client-side tool approval, described in the Security Note, is the control that puts a person in front of each write. A write whose result is unknown is not retried automatically, because it may have succeeded. Every result is treated as data, not as instructions.

## Repository layout

| Path | Read by |
| --- | --- |
| `plugin.json`, `mcp.json` | Cursor and other [Agent Plugins](https://agent-plugins.org) clients |
| `.claude-plugin/plugin.json`, `.mcp.json` | Claude (claude.ai, Cowork, Claude Code) |
| `assets/logo.png` | Directory listings |

## Security

To report a vulnerability in this plugin or in the Doppel MCP server, follow [SECURITY.md](SECURITY.md). Please do not open a public issue for security reports.

## Support

Doppel customers can reach support through their usual Doppel channels, including for problems with the plugin files in this repository. GitHub Issues are not enabled on this repository.

## License

Apache-2.0. See [LICENSE](LICENSE).
