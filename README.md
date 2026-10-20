# OKA Anywhere: your office's procedures inside Claude or ChatGPT

**OKA Anywhere** is a remote MCP connector by [Forge Agents LLC](https://forgeagentsio.com) for
small offices that run on documented procedures, such as property management companies,
home-service and trade shops, and insurance agencies. It connects Claude or ChatGPT to your
office's own procedures; other assistants that support custom MCP connectors may work, but we do
not promise them. Staff ask in the chat they already have open; answers come only from your
office's approved procedures, with verified citations, and if your documents don't cover it, it
says so.

| | |
| --- | --- |
| **Connector address** | `https://console.forgeagentsai.com/mcp` |
| **Transport** | Streamable HTTP, stateless |
| **Sign-in** | OAuth consent page (paste your office token when you connect), or `Authorization: Bearer <office token>` for scripts and Claude Code |
| **You need** | Your **office token**, shown on the welcome page right after you subscribe or start the 14-day free trial |
| **Price** | $149 a month per office, no per-question charges |
| **Free trial** | 14 days through Stripe checkout. Stripe collects a card, and you are not charged if you cancel before the 14 days end |
| **Website** | [forgeagentsio.com/oka.html](https://forgeagentsio.com/oka.html) |
| **Setup guide** | [forgeagentsio.com/oka-guide.html](https://forgeagentsio.com/oka-guide.html) |
| **Support** | daldridge@forgeagentsio.com |

## What it does

Every office has its own set of approved procedure documents. OKA Anywhere puts that set behind four
tools, so anyone on staff can ask their own assistant "how do we handle this?" and get an answer
that names the document it came from and quotes it. Each quote in its Sources list is checked word
for word against the source before the tool returns it. When the documentation does not cover a
question, OKA Anywhere says so plainly. A refusal is the correct answer in that case, not a fault.

The tools return only what your documents say, with every quote checked against the source. Your
assistant writes its own reply around them, so check the Sources it shows.

## Getting started

1. Go to [forgeagentsio.com/oka.html](https://forgeagentsio.com/oka.html) and click **Subscribe: $149/month**.
   Stripe checkout starts you on 14 days free, and you are not charged if you cancel before the 14 days end.
2. After checkout, you land on a welcome page with your connector address and office token right away.
   It shows the token for 30 minutes, and only in the browser where you first open it, so copy it
   somewhere safe. New offices start with a sample office's procedures; when you are ready, upload your own from the
   welcome page (Load your procedures), or later at
   [console.forgeagentsai.com/office/procedures](https://console.forgeagentsai.com/office/procedures),
   and the same token answers from them a few minutes later.
3. Connect it (below). When the connector opens the Forge consent page, paste the office token. The
   page asks each time someone connects, so on a plan where each person clicks Connect, each person
   pastes it once.

Upload office procedures and policies only, never patient, client or personnel records.

## Connect it

**Claude** (set up on the web or desktop, then it works on your phone too): Customize → Connectors →
\+ Add → Add custom connector. On a Team or Enterprise plan, an owner adds it first under
Organization settings → Connectors, then each person clicks Connect. Paste the address, click
Connect, and paste your office token on the Forge page that opens.

**ChatGPT** (Business, Enterprise or Edu on the web, added by an admin or owner; Pro works too):
Plugins → + → Add custom MCP server. Paste the address, choose OAuth, and paste your office token
on the Forge page that opens.

**Claude Code** (no browser needed):

```bash
claude mcp add --transport http oka-anywhere https://console.forgeagentsai.com/mcp \
  --header "Authorization: Bearer YOUR_OFFICE_TOKEN"
```

**Other MCP clients** may also work, but we do not promise them. Use the address above with
Streamable HTTP. A client that cannot send headers is sent to the consent page. A client that can
send headers can use the office token as a bearer token on the same address. Steps for Cline,
written so an agent can follow them, are in [llms-install.md](llms-install.md).

After connecting, the client lists four tools and three ready-made prompts.

## The four tools

All four tools are read-only. They never change your documents or anything else, and each one is
marked that way (`readOnlyHint: true`, `destructiveHint: false`, `openWorldHint: false`) so a client
can treat them as safe to run.

| Tool | What it does | Input | Uses a model? |
| --- | --- | --- | --- |
| `ask_procedures` | Answers a question from your documents, with a Sources list of verified quotes | `question` | Yes. Every citation is machine-verified against your documents |
| `verify_claim` | Checks whether an exact sentence appears word for word in your documents | `claim` | No |
| `search_procedures` | Finds up to three of the most relevant passages for a topic | `query` | No |
| `list_boundaries` | Lists what your documentation explicitly forbids or does not cover | none | No |

Each tool also takes an optional `corpus` field. Leave it out: it defaults to your office's own
document set, and an id that is not yours is refused.

## Prompts

Three prompts can show up in a client's prompt menu. They only tell the assistant which tool to use and
to keep the refusal rule. They carry no office text.

| Prompt | Input | What it asks for |
| --- | --- | --- |
| `ask-a-procedure-question` | `question` | An answer from `ask_procedures` with its Sources, and `list_boundaries` if the topic is not covered |
| `verify-a-statement` | `statement` | An exact check with `verify_claim`, then the closest passages from `search_procedures` if it is not found |
| `show-what-is-not-covered` | none | Everything the documentation says it does not cover, grouped by document |

## Examples

Questions to ask your assistant once it is connected:

- "What is our policy on same-day cancellations?" The assistant calls `ask_procedures` and answers
  with the source document and a quote.
- "Does our handbook say refunds are issued within 14 days?" It calls `verify_claim`. It reports
  VERIFIED with the document and surrounding text, or says the sentence is not in the documents
  word for word.
- "Show me what we have on opening procedures." It calls `search_procedures` and returns up to three
  excerpts with their document titles.
- "What do our procedures say they do not cover?" It calls `list_boundaries`.

Tool calls look like this:

```json
{ "name": "ask_procedures", "arguments": { "question": "What is our policy on same-day cancellations?" } }
{ "name": "verify_claim", "arguments": { "claim": "Refunds are issued within 14 days." } }
{ "name": "search_procedures", "arguments": { "query": "late cancellation fee" } }
{ "name": "list_boundaries", "arguments": {} }
```

## Troubleshooting

- **"Couldn't reach the MCP server"**: paste the address again, exactly: `https://console.forgeagentsai.com/mcp`
- **"That token was not recognized"**: tokens rotate when re-issued, so use the newest one.
  If you lost it, email support and we will rotate you a fresh one in minutes.
- **The assistant refuses to answer something**: call `list_boundaries` first. Usually the topic is
  not in your documentation yet. Add the missing document to your files and upload them again.
- **Rate limit message**: each office token allows 60 tool calls an hour. Try again next hour.

## Server card

Directories that cannot sign in can read the public server card at
[`/.well-known/mcp/server-card.json`](https://console.forgeagentsai.com/.well-known/mcp/server-card.json).
It is built from the live tool list, so it always matches what a signed-in client sees, and it never
contains an office token or any office document.

## Your data

Your documents serve only your office. They are held on our server so OKA can answer from them.
To generate cited answers, ask_procedures sends your procedures and the question to Google's
Gemini API, under its business API terms, which do not allow this data to be used to train its
models. The other three tools use no model. We never use your documents to train models, and we
delete them within 30 days after your subscription ends, or sooner if you ask. Backup copies are
deleted as the backup cycle turns over.

Calls are logged for support and metering: when, which office and which tool, how long it took and
whether it worked, with token counts, never the question text.

Your staff ask inside Claude or ChatGPT, under your office's own account with that assistant, and
that assistant keeps the chat history under its own terms.

---
**Support:** daldridge@forgeagentsio.com · **Website:** [forgeagentsio.com/oka.html](https://forgeagentsio.com/oka.html) ·
**Connector:** https://console.forgeagentsai.com/mcp ·
**Test results:** [forgeagentsio.com/proof.html](https://forgeagentsio.com/proof.html) ·
**Privacy:** https://forgeagentsio.com/privacy.html · $149 a month per office, no per-question charges.
