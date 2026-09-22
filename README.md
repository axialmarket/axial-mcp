# Axial MCP Server

The Axial MCP server lets an AI assistant work your Axial pipeline for you. Once it is connected,
the assistant can list your projects and the deals on them, read a deal and its message thread,
open a document you already have access to, and -- when you tell it to -- pursue a deal, pass on
one, request access, set a status, or send a message to the other side.

It acts as **you**. It sees exactly what you see when you sign in to Axial: your projects, your
account's deals, the documents you have been granted. It can do nothing your own login cannot do.

## Before you connect

1. **An Axial account.** Your assistant signs in as you, through the same Axial login you use in a
   browser.
2. **Accepted terms of service.** If you have never accepted them, tools will refuse with a message
   telling you to sign in at Axial and accept them. Do that once, then retry.
3. **An MCP client.** Claude (web, desktop, or Claude Code), ChatGPT, Cursor, or any other client
   that supports remote MCP servers over HTTP.

## The server address

``` text
https://api.axial.net/mcp
```

That single URL is all a client needs. Everything else -- discovery, sign-in, and token refresh --
the client and the server negotiate between themselves.

## Connecting

### Claude (web or desktop)

1. Open **Settings > Connectors**.
2. Choose **Add custom connector**.
3. Name it `Axial`, and paste `https://api.axial.net/mcp` as the URL.
4. Save, then click **Connect**. A browser window opens on the Axial sign-in page.
5. Sign in and approve the request. The window closes and the connector shows as connected.

The Axial tools are now offered to Claude in any conversation. Ask it something like *"what deals
are waiting on me?"* to confirm.

### Claude Code

```bash
claude mcp add --transport http axial https://api.axial.net/mcp
```

Then run `/mcp` inside Claude Code and authenticate when prompted. `claude mcp list` shows the
connection state.

### ChatGPT, Cursor, and other clients

Add a **remote** (HTTP / streamable-HTTP) MCP server pointing at `https://api.axial.net/mcp`. The
client will discover the sign-in endpoints on its own, register itself, and open a browser for you
to sign in. No client id, client secret, or API key is issued by Axial or needed by you.

## Signing in

Authentication is standard OAuth: the client sends you to Axial's login, you sign in (and approve
the connection the first time), and the client receives a short-lived token it refreshes on its
own. Axial never asks you to paste a token, and your password is never given to the client.

Access lasts as long as your Axial account does. To revoke it, remove the connector in your client.

## What the assistant can and cannot do

- **It reads only your own data.** Every request resolves to your Axial user, and every query is
  scoped to your account's projects and deals.
- **Actions that another person will see are yours to authorize.** Pursuing a deal, passing on one,
  requesting access, changing a status, and sending a message all notify a real counterparty. A
  well-behaved client will confirm with you before any of these; the tools are written to be asked
  for explicitly, not guessed at.
- **It cannot see documents you have not been granted.** Document access follows the same
  permissions as the web app, including NDA gates.

## When something does not work

| What you see | What it means |
|---|---|
| The client cannot connect at all | Check the URL is exactly `https://api.axial.net/mcp`, with no trailing slash. |
| Sign-in opens but the client stays disconnected | Let the browser window finish and close on its own; closing it early aborts the exchange. |
| "The Axial terms of service have not been accepted" | Sign in to Axial in a browser, accept the terms, then retry. |
| A tool answers that you have no projects | You are connected, but your Axial user is not assigned to a project. Ask your Axial contact to assign you. |
| Tools stop working after a while | Your session expired and the refresh failed. Disconnect and reconnect the connector. |

Anything else, or a tool that answers something that looks wrong: contact your Axial representative
with the name of the tool and what you asked for.

---

## Tool Reference

Every tool the server registers, with what it does. A tool's full instructions -- its arguments, its columns, and what it does when it cannot answer -- are carried in the tool definition the client reads at connect time; the summary here is the first paragraph of that text.

| Tool | Description |
| --- | --- |
| `buyside_deals_requiring_action` | The caller's BUYSIDE deals that are waiting on the CALLER - "what needs my attention", "what am I holding up", "what should I do next". Answers with the ranked deals, not a count. Covers every buyside project on the account unless `project_public_id` or `assigned_to_me` narrows it. |
| `buyside_get_deal` | Full detail for ONE deal a BUYSIDE project of the caller's received, by a "deal_id" from buyside_list_deals or buyside_deals_requiring_action. A SELLSIDE deal is refused: there is no per-deal sellside read. |
| `buyside_get_project` | Full detail for one of the caller's BUYSIDE projects: the buy box - what they are looking to buy - plus the roles they play, who they buy for, the terms sought and the deal counts. |
| `buyside_list_deals` | The buyer's own pipeline: every deal the caller's BUYSIDE projects have received that is still in play, as one ranked list - "what came in", "what is waiting on me". Buyside only; sellside_list_deals is the counterpart, for a SELLSIDE project id. Covers the whole account unless an argument narrows it. |
| `buyside_list_projects` | A page of the caller's BUYSIDE projects - what they are looking to buy - the live ones first, then those with the most deals waiting. Summaries only, which keeps the call cheap on an account holding many; call buyside_get_project for one project's headline, sectors, revenue bounds and the rest. Sellside projects are not here: sellside_list_projects answers those. |
| `buyside_pass_on_deal` | Pass on one or more deals received on a buyside project, by their "deal_id". IRREVERSIBLE through this server. |
| `buyside_pursue_deal` | Pursue ONE deal received on a buyside project, by its "deal_id". IRREVERSIBLE through this server. |
| `buyside_request_deal_access` | Send the caller's written pitch to ONE pitch-gated deal, asking the seller for access. IRREVERSIBLE: it is delivered and recorded on the deal permanently, with no edit and no recall. |
| `buyside_set_status` | Record the caller's OWN buyside status on one or more deals they received, by their "deal_id". |
| `get_deal_document` | Download ONE document attached to ONE of the caller's deals - a CIM, an NDA, a teaser, or whatever else the two sides have exchanged. Answers with the file itself, in whatever format it was uploaded in: PDF, DOCX, XLSX and so on. |
| `get_deal_thread_attachment` | Download ONE file attached to a message on one of the caller's deals - whatever the two sides have emailed each other: a redlined NDA, a diligence list, a spreadsheet. Answers with the file itself, in the format it was sent in. |
| `get_user` | Identify the authenticated caller: report which side(s) of the Axial marketplace their account works. |
| `read_deal_thread` | Read the conversation on ONE of the caller's deals, newest first. Either side's deal is accepted; "My Side" says which the caller sits on. READ-ONLY and marks nothing: reading does not mark a thread read, and the counterparty sees nothing. |
| `sellside_get_project` | Full detail for one of the caller's SELLSIDE projects: the company being sold and how its sale is set up - financials, industries, geographies, terms, buyer filters, counts, documents, timeline. |
| `sellside_list_deals` | SELLSIDE ONLY. The buyers on ONE of the caller's sellside projects: recommended, contacted or interested. A BUYSIDE project id is refused, not answered - buyside_list_deals is that tool. |
| `sellside_list_projects` | A page of the caller's SELLSIDE projects - each a company being sold - the ones still being worked first, newest first within that. Summaries only, which keeps the call cheap on an account holding many; call sellside_get_project for one project's full detail. Buyside projects are not here: buyside_list_projects answers those. |
| `send_deal_message` | Send ONE message on ONE of the caller's deals, to the other side of it. |
| `set_external_id` | Stamp ONE deal with the CALLER's own identifier for it - the id, code or reference that names this same deal in the caller's own systems, such as a CRM record or a pipeline tracker. Works on a deal the caller sits on EITHER side of. |
| `upload_deal_message_attachment` | Store ONE file so it can be attached to a message, and answer with the id that attaches it. |

---

## About This Document

The tool reference above is generated, not written. It is read out of the running Axial MCP server -- the same tool definitions your client is offered when it connects -- so a tool that appears here is a tool you can call, spelled the way the server spells it. New tools are added to Axial over time; ask your client to list the tools it has for the authoritative set at any given moment.

---

**Last Updated:** 2026-09-22 (19 tools)
