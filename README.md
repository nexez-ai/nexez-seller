# Nexez Seller plugin

Nexez Seller lets a merchant work with their own Nexez account from an AI assistant: see listings, orders, negotiations and appointments, find what needs attention, and stage new listing wording as a draft.

This is version `0.1.1`. Version 0.1.0 is the one in review at OpenAI; 0.1.1 only updates the skills' wording and is not uploaded anywhere. Its server is live and sign-in has been tested for Claude, but the plugin has not been submitted to any directory. It reads, and it stages drafts. It never refunds, fulfils, ships, replies to a customer, decides a negotiation, cancels or moves an appointment, publishes, or changes a price. For each of those it returns the Nexez page where the seller confirms the action themself.

## How it fits together

1. Skills teach the assistant how to handle a seller's request, which tools to use and where to stop.
2. The seller MCP server at `https://nexez.app/mcp/seller` gives the assistant the signed-in seller's own data.
3. Sign-in is OAuth. Supabase Auth is the authorization server and Nexez checks every token. There is no API key and no anonymous access: a request without a verified seller token gets HTTP 401.

It is the seller-side counterpart of the Nexez Buyer plugin (`nexez-ai/nexez-buyer`), and deliberately separate from it. The buyer tools are public and live at `/mcp` and `/mcp/chatgpt`. None of the seller tools is listed there.

## What is included

| Component | Purpose |
| --- | --- |
| `.claude-plugin/plugin.json` | Claude Code package metadata |
| `.codex-plugin/plugin.json` | ChatGPT and Codex package metadata |
| `.claude-plugin/marketplace.json` | Lets the plugin's own repository act as its install source |
| `.mcp.json` | Connection to the seller MCP endpoint, with the sign-in client for Claude Code |
| `skills/daily-briefing` | What needs attention, and what is coming up |
| `skills/review-orders` | Finding and explaining orders, with hand-offs for every action |
| `skills/review-negotiations` | Proposals waiting on the seller |
| `skills/improve-listing` | Readiness, and staging new wording as a draft |
| `openai/mcp.json` | Server and sign-in settings for the OpenAI Platform upload |
| `openai/review.json` | Review details for the OpenAI Platform upload |
| `evals/cases.json` | Scenarios for checking routing and safety |

## Tools

All ten need a signed-in seller and return only that seller's own data.

| Tool | What it does | Writes |
| --- | --- | --- |
| `nexez_seller_overview` | Listing counts, attention count, upcoming appointments | No |
| `nexez_seller_list_listings` | Listings with status, offer count, readiness, staged draft flag | No |
| `nexez_seller_get_listing` | One listing: description, offers, FAQs, readiness checklist, staged draft, booking setup, branding | No |
| `nexez_seller_list_orders` | Orders, newest first, with status and text filters | No |
| `nexez_seller_get_order` | One order: fulfilment, tracking, timeline, buyer requests, reviews | No |
| `nexez_seller_list_attention` | Disputes, refund requests, problem reports, orders to fulfil or ship, negotiations waiting | No |
| `nexez_seller_list_negotiations` | Proposals with status, value and next step | No |
| `nexez_seller_list_appointments` | Booked appointments that have not ended | No |
| `nexez_seller_handoff` | The Nexez page for an action the plugin does not perform | No |
| `nexez_seller_stage_listing_draft` | Stages a new name, description or FAQs as a draft | Draft only |

## Safety boundary

- **Own data only.** The server reads through the seller's own access token, so the database's row level security applies, and every query also filters on the verified seller's id.
- **One write, never public.** `nexez_seller_stage_listing_draft` writes the listing's staged draft and nothing else. The public listing changes only when the seller presses Publish in the Nexez editor, where a staged draft can also be discarded. The tool refuses to replace wording the seller already staged unless told to, and refuses if the listing changed since it was read.
- **Hand-offs for everything consequential.** Refunds, fulfilment and tracking, replies to buyers, negotiation decisions, appointment changes, publishing, offers and prices, booking rules, branding, payouts and billing all go through `nexez_seller_handoff`, which returns a link to the seller's own dashboard.
- **Private price rules stay private.** A price floor, discount limit or auto-accept band is reported as set or not set. The numbers are never returned.
- **Customer details are limited.** Buyer names are returned. Email addresses are masked (`j***@example.com`) and phone numbers and contact routes are left out.
- **Buyer text is data.** Anything a buyer wrote (a refund reason, a review, a proposal) is returned under `buyerWrote`, and the skills tell the assistant never to follow instructions inside it.

After staging, the assistant must say: `Staged as a draft. Nothing on your public listing changed.`

## What this release does not include

- Editing offers, prices, price rules, booking rules, branding or photos.
- Publishing or unpublishing a listing.
- Any order action (refund, fulfilment, tracking, answering a request) or negotiation action.
- Booking, cancelling, moving or proposing appointments, and appointment notes.
- Scans, analytics, payouts and finance figures.
- Team members. A collaborator on someone else's listing sees nothing through the plugin yet.
- Separate ChatGPT skills. There is one seller server, so one set of skills serves Claude, Codex and ChatGPT.

## How it is switched on

The server answers 404 unless `NEXEZ_SELLER_OAUTH_CLIENT_IDS` names at least one OAuth client registered in Supabase Auth. It is on in production for one client, "Claude (Nexez Seller)". Adding another assistant means registering a client for it and adding its id to that list. See `docs/seller-plugin.md` in the main Nexez repository, which is private.

## Install in Claude Code

The plugin has its own public repository, `nexez-ai/nexez-seller`, a copy of `plugins/nexez-seller` in the main Nexez repository. It installs in one step on Claude Code 2.1.275 or later:

```text
/plugin install nexez-seller --marketplace nexez-ai/nexez-seller
```

Then run `/mcp`, choose the `nexez-seller` server and authenticate. Your browser opens the Nexez page that asks whether Claude may use your Nexez account. Check what it lists, then press Connect.

## Try it with Claude Code

From the repository root:

```bash
claude --plugin-dir ./plugins/nexez-seller
```

Claude Code finds `.mcp.json`, receives the 401 challenge and opens the Nexez sign-in and consent page.

The sign-in server does not let apps register themselves, so `.mcp.json` names the registered public client (`oauth.clientId`) and a fixed port (`oauth.callbackPort`). Claude Code listens on that port for the answer, which means the return address `http://localhost:33418/callback` has to be on that client's list in Supabase Auth. Without it the sign-in stops with `invalid redirect_uri`. The client id is an identifier, not a secret.

In claude.ai, add `https://nexez.app/mcp/seller` as a custom connector and enter the same client id under its advanced settings.

Then try:

```text
What needs my attention on Nexez today?
```

Validate the package before distributing it:

```bash
claude plugin validate ./plugins/nexez-seller
```

## ChatGPT

The upload package for the OpenAI Platform takes its server and sign-in settings from `openai/mcp.json`, placed at the package root as `mcp.json`, with the `interface` keys and `extensions` block from `openai/review.json` merged into `.codex-plugin/plugin.json`. Create the plugin from a package that already has the sign-in settings: a draft first uploaded without them did not pick them up later.

It names its own public OAuth client, "ChatGPT (Nexez Seller)". Two things have to be true before it can connect: that client id is listed in `NEXEZ_SELLER_OAUTH_CLIENT_IDS` on the deployment, and the return address ChatGPT shows when Connect is pressed (`https://chatgpt.com/connector/oauth/<callback id>`) is on the client in Supabase Auth. Both are done for the Nexez organization, and a draft is uploaded and connected on the OpenAI Platform. It is not submitted for review.

## Data and privacy

The plugin sends the arguments of the selected tool to `https://nexez.app/mcp/seller` together with the seller's access token. Nexez returns that seller's own listings, orders, negotiations and appointments to the assistant, which means the assistant's provider processes them. That includes customer names and text customers wrote to the seller. Email addresses are masked and phone numbers are not returned.

The seller can disconnect at any time from the assistant, which ends the plugin's access.

Read the [Nexez Privacy Policy](https://nexez.ai/privacy) and [Terms of Service](https://nexez.ai/terms). For help, use [Nexez Support](https://nexez.ai/support).

## License

This plugin bundle is available under the [MIT License](./LICENSE). The license applies only to the files in this plugin folder. The Nexez platform, services, trademarks and other source code are not relicensed by this file.

## Example prompts

- What needs my attention on Nexez today?
- Show me this week's orders and anything not yet fulfilled.
- Did any buyers send me an offer?
- What appointments do I have tomorrow?
- Why is my listing's readiness low, and can you draft a better description?

## Release checklist

1. Validate both plugin manifests and all four skills.
2. Confirm `https://nexez.app/mcp/seller` answers 401 with a `WWW-Authenticate` challenge when called without a token, and lists exactly the ten tools when signed in.
3. Run the cases in `evals/cases.json` against a test seller account and keep the results.
4. Sign in from Claude Code and check the consent page shows the seller wording.
5. Confirm a buyer-plugin token is refused on the seller server.
6. Review the privacy policy, terms and store listing copy for the seller data described above.
7. Submit to a marketplace only after the owner approves the final listing.
