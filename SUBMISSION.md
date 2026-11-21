# Nexez Seller publication draft

Draft copy and reviewer notes for a future release. Nothing here has been submitted for review or approved for Claude. For ChatGPT, version 0.1.0 is in review on the OpenAI Platform and is not approved or published. The server is live for the registered Claude and ChatGPT clients.

## Public listing

- Name: Nexez Seller
- Short description: See and tend your own Nexez business from your assistant.
- Category: Business & Operations
- Website: https://nexez.ai/agents
- Support: https://nexez.ai/support
- Contact email: support@nexez.ai
- Privacy policy: https://nexez.ai/privacy
- Terms: https://nexez.ai/terms
- MCP server: https://nexez.app/mcp/seller
- Authentication: OAuth 2.1 with PKCE. Authorization server: Supabase Auth for the Nexez project. Protected resource metadata: https://nexez.app/.well-known/oauth-protected-resource/mcp/seller
- Transport: Streamable HTTP

### Long description

Nexez Seller connects a merchant's own Nexez account to their assistant. Ask what needs attention, look up an order, see which buyers sent a proposal, check tomorrow's appointments, or find out why a listing's readiness is low.

The plugin reads the signed-in seller's own data and can stage new listing wording as a draft. It never moves money, never contacts a customer and never changes what the public sees. Refunds, replies, negotiation decisions, appointment changes, prices and publishing stay on Nexez, where the seller confirms each one. The plugin gives the link.

## Starter prompts

- What needs my attention on Nexez today?
- Show me this week's orders and anything not yet fulfilled.
- Did any buyers send me an offer?
- What appointments do I have tomorrow?
- Help me improve the description on my listing.

## Tool annotations

Nine tools are read-only and not destructive. One writes a staged draft and is marked destructive, because with `replaceStaged` it can replace wording the seller already staged. None is open-world: each reads or stages the signed-in seller's own Nexez data.

| Tool | Annotation justification |
| --- | --- |
| `nexez_seller_overview` | Read-only counts from the seller's own account. |
| `nexez_seller_list_listings` | Read-only list of the seller's own listings. |
| `nexez_seller_get_listing` | Read-only. Private price rules are reported as set or not set, never as numbers. |
| `nexez_seller_list_orders` | Read-only. Buyer email addresses are masked. |
| `nexez_seller_get_order` | Read-only. Buyer-written text is returned under `buyerWrote`. |
| `nexez_seller_list_attention` | Read-only action queue with links to the seller's dashboard. |
| `nexez_seller_list_negotiations` | Read-only. Buyer contact details are left out. |
| `nexez_seller_list_appointments` | Read-only list of booked appointments. |
| `nexez_seller_handoff` | Read-only. Returns a first-party Nexez link and performs nothing. |
| `nexez_seller_stage_listing_draft` | Not read-only, marked destructive. Replacing already staged wording cannot be undone, so a client should ask first. Writes the listing's staged draft (name, description, FAQs). The public listing is unchanged until the seller publishes in the Nexez editor. Refuses to replace existing staged wording without `replaceStaged`. |

## Reviewer setup

A reviewer needs a Nexez seller test account with at least one listing, one order and one appointment. The owner supplies its sign-in details through the directory's reviewer-credentials field, never in this file.

Connect `https://nexez.app/mcp/seller`. The first request returns 401 with a `WWW-Authenticate` header pointing at the protected resource metadata. Sign in with the test account and approve the consent page, which lists what the plugin can and cannot do.

Run the twelve cases in `evals/cases.json`. The first six are positive cases and the last six are negative safety cases.

## Release notes

First release, version 0.1.0. Ten tools for a signed-in seller: nine reads and one staged-draft write. No tool moves money, contacts a customer, publishes or deletes.

## OpenAI upload package

The ZIP uploaded to the OpenAI Platform is assembled from this folder, not uploaded as is:

- `.codex-plugin/plugin.json`, with `mcpServers` pointed at `./mcp.json` and the `interface` keys and `extensions` block from `openai/review.json` merged in.
- `openai/mcp.json` placed at the package root as `mcp.json`.
- `skills/`, `assets/` and `LICENSE`.

One top-level folder inside the ZIP is accepted. Reviewer sign-in details are entered on the Platform's review form and never go in the package.

## Claude directory

Not submitted. What a submission needs, following the buyer plugin:

- A public repository, `nexez-ai/nexez-seller`, holding a copy of this folder. It was created on 2026-11-20 with the owner's go-ahead, and the manifests name it under `repository`. It is synced by hand from `plugins/nexez-seller`.
- `claude plugin validate` on the folder: passes, with no directory findings (checked with Claude Code 2.1.319). The directory also runs a safety scan and a policy review that the command does not.
- A reviewer account. The one prepared for the OpenAI review has the demo data the eval cases need. Its sign-in details go in the submission form only.

### The submission form

Submissions are made at `https://claude.ai/directory/manage` (Submit new). The form has six steps. These are the answers prepared for this plugin; the owner sends it.

1. **Source.** Repository `nexez-ai/nexez-seller`, default branch `main`, plugin at the repository root. The form validates the manifest and runs the directory's checks. A saved draft cannot change its repository, folder or branch.
2. **Content.** Read from the repository: 1 MCP server, 4 skills, no hooks, commands, agents or mods.
3. **Listing.** Read from `plugin.json` and the README: display name, description, author, keywords, privacy policy, support and homepage links. Surfaces to list on:
   - Claude Code: yes. Sign-in and all tools are proven there.
   - Cowork and the Claude apps (web, desktop, mobile): the form says a remote MCP server that needs sign-in works there only through its claude.ai connector. That path is not tested for this plugin, so tick these only after it is, or accept that the listing may not connect there.
4. **Data handling.**
   - Reads or stores personal data: **Reads only.** It reads the seller's own customer records (buyer names, masked email addresses, text buyers wrote). It stores no personal data: the one write saves listing wording.
   - Any skill sends data to a service other than the declared connectors: **No.**
   - How long the service retains data received from Claude: **Longer.** Wording staged with `nexez_seller_stage_listing_draft` stays on the seller's listing until they publish or discard it. Nothing else sent by Claude is kept beyond ordinary request logs.
   - Intended for users under 18: **No.**
5. **Compliance.** A contact email for scan results and review questions (not shown publicly), and four confirmations made on behalf of the organization: the Software Directory Terms and Directory Policy, that the privacy policy describes the data handled, that the plugin does not exfiltrate credentials or run code outside its declared MCP server, and that Anthropic may contact the account. These are the owner's to read and tick.
6. **Review and submit.** The owner's.

The data handling answers above are a recommendation. The second and third depend on what Nexez logs and keeps, which the owner knows better than these notes.

Notes for a reviewer:

- Every request needs sign-in. The first call returns 401 with a `WWW-Authenticate` challenge, and Claude Code opens the Nexez consent page.
- The sign-in server does not let clients register themselves, so `.mcp.json` names a pre-registered public client (`oauth.clientId`) and a fixed local port (`oauth.callbackPort`, 33418). A public client id is an identifier, not a secret.
- One tool writes: `nexez_seller_stage_listing_draft`, marked destructive, which stages listing wording as a draft that is not public. Everything else is read-only.

Proven in Claude Code: install from the folder, sign-in, and the read tools and the draft tool on production. Not proven: the plugin on claude.ai, Claude Desktop or Cowork, where the connection is made from Anthropic's side with the return address `https://claude.ai/api/mcp/auth_callback`. That address is registered on the client, but whether those surfaces pass the client id from `.mcp.json` has not been tested.

## What the review form holds

- Countries, the five test cases, the three negative cases and release notes come from the package (`openai/review.json`).
- Login URL `https://app.nexez.ai/login` and written sign-in steps.
- A dedicated reviewer account with demo data: one published listing ("Nexez Demo Studio", free intro call and a negotiable brand photo session), one free booking with its order, and one buyer proposal. Its username and password were typed into the form by the owner and are not recorded anywhere in this repository.
- A video walkthrough of the five test cases, hosted on the owner's Google Drive.

Test case 5 stages a draft on the demo listing, and it stays staged until someone publishes it or presses Discard draft in the listing editor (Staged publishing). A later run of the same prompt may answer that the wording is already staged.

## Versions

- 0.1.0: the package uploaded to the OpenAI Platform and in review.
- 0.1.1: skills wording only. The skills now say a staged draft can be discarded in the Nexez editor as well as published. No tool, server or manifest behaviour changed. Not uploaded anywhere.

## Publication state

- Claude: not submitted. Server live, sign-in and tools tested on production in Claude Code on 2026-11-19 (see `docs/seller-plugin.md`). The public repository the directory needs exists.
- ChatGPT and Codex: version 0.1.0 was submitted for review on the OpenAI Platform on 2026-11-20 by the owner. It is not published. Before submission the Platform's scan listed all ten tools and, after the draft tool's `destructiveHint` was changed to true, reported no findings.
