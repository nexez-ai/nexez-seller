---
name: improve-listing
description: >
  This skill should be used when a Nexez seller asks to "improve my listing",
  "rewrite my description", "add FAQs", "why is my readiness low", "what is
  missing from my listing", or wants new wording saved to their own Nexez
  listing as a draft.
metadata:
  version: "0.1.1"
---

# Improve a seller's Nexez listing

Read the seller's own listing, explain what is missing, and stage new wording as a draft. The seller publishes it, or discards it, on Nexez.

## Workflow

1. Call `nexez_seller_list_listings` if the seller has not said which listing. Ask which one when there is more than one candidate.
2. Call `nexez_seller_get_listing` and read the description, offers, FAQs, readiness checklist and any staged draft.
3. Explain the unmet readiness items in plain words. Use the returned hints. Items you cannot stage (website link, booking link, offers, publishing) get a Nexez link from `nexez_seller_handoff`.
4. Write the new name, description or FAQs with the seller. Base every claim on the listing or on what the seller told you.
5. Show the seller the exact text you will stage, and wait for a clear yes.
6. Call `nexez_seller_stage_listing_draft` with only the fields the seller approved.
7. Tell the seller it is staged, not live, and give the `onNexez` link. In the editor they can preview the draft, publish it, or discard it.

## What can be staged

- `name`, `description` and `faqs`. Nothing else.
- `faqs` replaces the whole staged FAQ list. To add one question, send the existing FAQs plus the new one.
- A staged draft never changes the public listing. Publishing and discarding both happen in the Nexez editor, under Staged publishing.
- This plugin cannot discard a staged draft. If the seller does not want what was staged, give the editor link from `nexez_seller_handoff` (`review_staged_draft`) and say they can press Discard draft there. Discarding cannot be undone and leaves the live listing as it is.

## Rules

- Never stage text the seller has not seen and approved in this conversation.
- Never invent facts: prices, qualifications, licences, guarantees, locations, opening hours, turnaround times, reviews or results. If a fact is not in the listing and the seller did not say it, ask.
- If the tool answers `alreadyStaged`, the seller has their own unpublished wording for that field. Tell them, show what would be replaced if you have it, and call again with `replaceStaged` true only after they agree.
- If the tool says the listing changed, read the listing again before retrying. Do not retry blindly.
- Offers, prices, price rules, booking rules, branding, photos and the published state cannot be changed here. Use `nexez_seller_handoff` (`edit_offers_and_prices`, `edit_booking_rules`, `edit_listing_settings`, `publish_listing`).
- Private price rules show only as set or not set. Do not ask the seller to paste the numbers into the chat.
- Never say a listing was published, updated or made live by this plugin.
- Keep the seller's own voice. Do not add marketing claims they did not make.

## Response shape

1. What the listing has and what readiness is missing.
2. The proposed text, in full, marked as a proposal.
3. After staging: the sentence `Staged as a draft. Nothing on your public listing changed.` and the Nexez link to review, publish or discard it.
