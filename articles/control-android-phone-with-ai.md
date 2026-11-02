# Control a real Android phone with AI using DroidDesk MCP

An AI assistant can do more than explain where to tap. With **DroidDesk MCP**, it can inspect an authorized rented Android phone, open apps and websites, tap, type, scroll, and check the resulting screen. You describe the task in ordinary language while your chosen AI client uses DroidDesk’s tools to carry it out.

The phone runs on physical Android hardware, and the MCP server runs on DroidDesk’s side. **MCP access is included in your phone rental.** You pay your AI provider separately under its own terms.

This guide explains how to connect your agent, try a first task, and turn a working sequence into a reusable template. For device options and rental plans, see the [DroidDesk overview](../README.md).

![How DroidDesk MCP works: a user describes a task, an AI agent sends authorized actions through DroidDesk MCP, a physical Android phone responds, and the agent checks the updated screen.](../assets/droiddesk-mcp-workflow.png)

## How the DroidDesk Android MCP server works

MCP, or Model Context Protocol, connects an AI client to external tools. DroidDesk provides tools for phone access; the language model stays in the AI service you choose.

The interaction is a loop. The agent reads the current screen, chooses an action, sends it through DroidDesk MCP, and checks the updated screen before continuing. A request to tap a button is only one step: the next screen tells the agent whether the intended action worked.

Your key controls which rentals and actions the agent may access. Supported features depend on your account, key permissions, device and rental plan.

## What your AI agent can do

| Task | Example request |
| --- | --- |
| Inspect a device | “Show the phones available to this connection, then describe the selected phone’s screen.” |
| Navigate a website | “Open this URL, find the pricing section and capture the displayed plan.” |
| Work in an app | “Open the selected app and find its language setting. Show me the current value.” |
| Prepare content | “Use my video and draft an English caption. Show me the caption before publishing.” |
| Fill a form | “Enter the details I provide and stop before submission.” |
| Repeat a workflow | “Verify these steps, then save them as a private template with parameters.” |

These are example requests to try on a suitably configured phone. Apps may ask for verification or show screens the agent cannot complete without your help.

## Before you connect

You need a [DroidDesk account](https://rent.droiddesk.io/), an active phone rental, and an AI client that supports remote MCP over **Streamable HTTP** with **Bearer authentication**. Codex, Claude Code and Cursor have configuration options for this connection.

Create a separate DroidDesk access key for the assistant. This is different from your dashboard password or AI provider key. Add it to the client’s connection settings; do not send it as an ordinary chat message.

## Create a DroidDesk access key

1. Sign in to your dashboard and open the rented phone’s automation page.
2. Find **AI / MCP** and its connection wizard. You can also manage keys under **AI & API**.
3. Name the key so you can recognize the client using it.
4. Choose the phone or rentals that the key may access.
5. Give it viewing permission for inspection, or control permission for taps, typing and other actions.
6. Save the key when it is shown. The dashboard later displays only its prefix. Revoke and replace a lost or exposed key.

If you want the agent to purchase additional rentals, enable that permission separately and set a maximum price per order where the option is available. An existing rental can be controlled without granting permission to buy another one.

## Connect your AI client

![DroidDesk MCP connection wizard, step 1 of 5, showing Codex, Claude Code, Cursor and another MCP client as connection options.](../assets/droiddesk-mcp-connect-wizard.webp)

*Choose your AI client in the DroidDesk connection wizard to see its setup instructions.*

Use this connection information:

| Setting | Value |
| --- | --- |
| Server name | `droiddesk` |
| Transport | Streamable HTTP |
| URL | `https://api.mobirent.io/mcp` |
| Authentication header | `Authorization: Bearer YOUR_DROIDDESK_KEY` |

The server address uses the `api.mobirent.io` domain shown in the DroidDesk connection instructions. Opening it in a browser is not a connection test; use your MCP client.

### Codex

Make your key available to the process running Codex as `DROIDDESK_API_KEY`. Then add the remote server:

```bash
codex mcp add droiddesk \
  --url https://api.mobirent.io/mcp \
  --bearer-token-env-var DROIDDESK_API_KEY
```

Check that the entry exists with `codex mcp list`. Alternatively, merge this entry into `~/.codex/config.toml`:

```toml
[mcp_servers.droiddesk]
url = "https://api.mobirent.io/mcp"
bearer_token_env_var = "DROIDDESK_API_KEY"
```

A variable set in a terminal is available to that terminal’s child processes. A desktop app launched from the operating system’s menu may not inherit it. Restart the client with the required environment and start a new session. See the [official Codex MCP documentation](https://developers.openai.com/codex/mcp).

### Claude Code

For an installed Claude Code client, add the HTTP server at user scope:

```bash
claude mcp add --transport http --scope user droiddesk \
  https://api.mobirent.io/mcp \
  --header "Authorization: Bearer YOUR_DROIDDESK_KEY"
```

Replace the placeholder privately. This form stores the header in the client configuration and may leave the entered command in shell history. Use your client’s supported secret-management options for ongoing use.

Check `claude mcp list`, then start a new conversation. These instructions are for **Claude Code**; other Claude apps can have different connection requirements. See [Claude Code MCP configuration](https://code.claude.com/docs/en/mcp).

### Cursor

Make `DROIDDESK_API_KEY` available to the Cursor process. Merge this entry into your global `~/.cursor/mcp.json`, preserving any existing servers:

```json
{
  "mcpServers": {
    "droiddesk": {
      "url": "https://api.mobirent.io/mcp",
      "headers": {
        "Authorization": "Bearer ${env:DROIDDESK_API_KEY}"
      }
    }
  }
}
```

Restart Cursor and check the MCP connection in its settings. Environment-variable interpolation keeps the key out of this file. The variable must be available to the app itself. See [Cursor’s MCP documentation](https://cursor.com/docs/mcp).

### Other clients

Use the same URL and Bearer header in a client that supports remote HTTP MCP. A client that only accepts OAuth connections may require a different integration method.

## Try your first task

First, confirm the device scope:

> Show the DroidDesk phones available to this connection. Do not change anything on the devices.

Choose a phone from the returned list, then check its state:

> On the selected phone, read the current screen and describe it. Do not tap anything yet.

For a first navigation task, try:

> On this phone, open https://droiddesk.io, find the weekly rental plan and tell me the price displayed. Take a screenshot of the plan. Do not place an order.

This exercise has a clear result you can check against the screen. You can keep the remote phone view open while the agent works. More detailed instructions help when several similar buttons, accounts or devices are available.

## Use AI Android automation for content and forms

For a content task, specify the phone, app, source file and intended stopping point:

> On my selected phone, open the publishing app. Use my attached video and prepare an English caption. Show me the caption and preview before publishing.

File handling depends on the client. Where direct transfer is supported, the agent can prepare the upload. Otherwise, use the private upload page provided by DroidDesk. Keep that personal link private.

For forms and messages, state which information the agent should use and whether it should stop for review. If an operation has an unknown result, check its status and the app before repeating it.

## Save a working sequence as a private template

After a sequence works on the phone, ask the agent to save it as a private template, if templates are available to your account:

> Save the verified steps as a private template. Make the page URL and target text parameters. Do not store passwords or one-time codes in the template definition.

Open **My Templates** in the phone’s automation area, supply the parameters and run the saved steps. The template can run without AI involvement. It follows its saved sequence, so interface changes, verification prompts or missing inputs may still require attention. Review its result after execution.

See [Android automation with fewer AI tokens](android-automation-templates-sms-verification.md) for Auto-swipe, scenarios built from saved templates, and an example of registration with SMS verification through a connected provider.

## What does DroidDesk MCP cost

**There is no separate charge for MCP access: it is included in your DroidDesk phone rental.** Your rental plan still determines which apps and installation methods are available.

Your chosen AI client or model provider may charge for its service. Optional SMS orders use your connected provider account and its balance. Check the [current rental plans](https://droiddesk.io/plans) before choosing a device.

## SMS and additional rentals

Where supported, you can connect your own SMSPool or TextVerified account in DroidDesk. Ask the agent to show available offers before ordering and give a maximum price. SMS availability and billing follow the provider’s terms.

Additional phone rentals also need explicit permission and an order limit. Specify the device, duration and maximum cost. If the result of a paid operation is uncertain, inspect the existing order before attempting another purchase.

## Troubleshooting

| What you see | What to check |
| --- | --- |
| Server is missing | Confirm the URL, saved configuration and remote HTTP support. Restart the client. |
| Authorization error | Check that the key is active, the Bearer header is present and the client can access the environment variable. |
| No phones are listed | Confirm an active rental and that the key includes the intended device. |
| Phone is listed but actions fail | Check control permission, device readiness and plan access. |
| Screen changed during the task | Ask the agent to read the screen again before choosing its next action. |
| Upload is unavailable | Use the private manual upload page if your client cannot transfer the file. |
| An action has an unknown result | Check the operation and device state before repeating it. |

## Frequently asked questions

### Does the AI run on the rented phone

The model runs in your chosen AI service. DroidDesk MCP gives that agent tools to interact with the phone.

### Can one connection access several phones

Yes, if those active rentals are included in the key’s scope. Identify the intended phone in each task.

### Does this require a local Android emulator

No. The task runs on your rented physical phone, and DroidDesk hosts the MCP server.

### Can the agent solve every CAPTCHA

Automatic completion is not guaranteed. A site may require your participation or other verification.

### Are browser profiles a backup of all my Android apps

No. Browser profiles store their own browser data and settings. Other app data and files on the rented phone follow the rental cleanup process. Save what you need before the session ends. [Browser profiles](https://droiddesk.io/help/droiddesk-web-app) · [Rental cleanup](https://droiddesk.io/help/data-after-rental-ends).

## Connect your first phone

Open the [DroidDesk dashboard](https://rent.droiddesk.io/), choose a rental suited to your task, and create an AI access key. Start by asking your agent to list the available phones and describe the selected screen.

[Explore DroidDesk and rental plans](../README.md) · [DroidDesk Help Center](https://droiddesk.io/help)
