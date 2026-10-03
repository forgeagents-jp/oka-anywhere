# Installing OKA Anywhere in Cline

OKA Anywhere is a remote MCP server. There is nothing to download, build or run locally. You add one URL and one office token.

## What you need

- Server URL: `https://console.forgeagentsai.com/mcp`
- Transport: streamable HTTP
- An office token from an OKA Anywhere subscription (the setup card Forge Agents sends when an office is onboarded). Ask the user for it. Never invent one, and never write it into a file that gets committed.

## Authentication

The server accepts the office token in two ways:

1. Bearer header. Send `Authorization: Bearer <office token>` on every request. Use this in Cline.
2. Consent page. Clients that cannot send headers are sent to a Forge consent page, where the token is pasted once.

## Add the server

Pick the case that matches how Cline is running. In both cases the server name is up to you. `oka-anywhere` is a good one, and if the marketplace install rules give you a name, use that.

### Cline CLI

Run one command. Replace `YOUR_OFFICE_TOKEN` with the token the user gives you:

```bash
cline mcp add oka-anywhere https://console.forgeagentsai.com/mcp --transport http --header "Authorization: Bearer YOUR_OFFICE_TOKEN" --yes
```

This writes the entry to `~/.cline/data/settings/cline_mcp_settings.json`. That file does not exist until the first server is added, so do not spend time searching for it first. The CLI picks up new servers when a session starts, so the four tools appear in the next Cline session, not in the one that ran the command. To check the connection right away, use the curl check below, then tell the user to start a new session.

### Cline in VS Code

Open Cline, then MCP Servers, then the Remote Servers tab. Enter the name `oka-anywhere` and the URL above, choose streamable HTTP, and add the header `Authorization` with the value `Bearer <office token>`.

If you prefer to edit the settings file, open the MCP Servers icon, the Configure tab, then the Configure MCP Servers button. That opens `cline_mcp_settings.json`. Add this entry, replacing `YOUR_OFFICE_TOKEN`:

```json
{
  "mcpServers": {
    "oka-anywhere": {
      "url": "https://console.forgeagentsai.com/mcp",
      "type": "streamableHttp",
      "headers": {
        "Authorization": "Bearer YOUR_OFFICE_TOKEN"
      },
      "disabled": false
    }
  }
}
```

If the settings file already has other servers, keep them and only add the `oka-anywhere` entry.

Cline's remote server format has changed between versions. If your version rejects this entry, use the Remote Servers tab as described above, or the generic form with only the `url` and `headers` fields.

## Check the connection without Cline

This asks the server for its tool list. It needs both `Accept` types, or the server answers 406:

```bash
curl -s https://console.forgeagentsai.com/mcp \
  -H "Authorization: Bearer YOUR_OFFICE_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

The reply names four tools. A 401 means the token is missing, mistyped or expired.

## Tools

After connecting, Cline should list four tools. In the CLI each one shows with the server name in front, for example `oka-anywhere__ask_procedures`.

- `ask_procedures`: answers from the office's own documents with verified source quotes.
- `verify_claim`: checks an exact statement against the documents, word for word.
- `search_procedures`: finds the most relevant procedure passages.
- `list_boundaries`: lists what the documentation explicitly does not cover.

All four only read. None of them changes anything.

## Verify it works

1. Ask Cline to call `list_boundaries`. It should return a list of topics the documentation does not cover.
2. Ask a question about a procedure, for example "What is our policy on same-day cancellations?" A good answer names the source document and quotes it.
3. Ask about something outside the documents. A good answer says plainly that the documentation does not cover it. A refusal here is correct behavior, not a fault.

## Troubleshooting

- 401 or "token not recognized": the token is missing, mistyped or expired. Tokens rotate when re-issued, so use the newest one. Check that the header reads `Bearer ` followed by the token, with one space.
- "Couldn't reach the server": check the URL is exactly `https://console.forgeagentsai.com/mcp` and the transport is streamable HTTP, not SSE or stdio.
- The server connects but shows no tools: in the CLI, start a new session. Otherwise remove the entry, add it again, and restart Cline.
- The assistant declines a question: call `list_boundaries` first. Usually the topic is not in the office's documentation yet.
- A request fails with 406 when you call the server directly: send `Accept: application/json, text/event-stream`.

## Support

daldridge@forgeagentsio.com
