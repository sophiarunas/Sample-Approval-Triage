---
name: sample-request-triage
description: "Triage pending TikTok Shop creator sample requests for a Reacher shop into Auto-approve, Auto-reject, or Needs manual review against brand-specific criteria, with reasoning, and deliver the results as a Google Sheet."
---

# Sample Request Triage

Automates the first layer of review for TikTok Shop affiliate sample requests on Reacher: instead of the user opening every creator profile from scratch, this skill pulls each pending request's signals and recommends a decision with reasoning, leaving only genuinely borderline cases for manual judgment. The output is delivered as a Google Sheet so the user (and teammates) can filter, sort, and mark decisions collaboratively.
## When to use

Use when the user asks to review, triage, or auto-screen pending sample requests for a shop/brand — e.g. "go through the pending samples for [shop]" or "triage this week's sample requests."

## Step 1: Get the shop and criteria

1. Call `list_shops` to find the shop_id(s) available. If the user named a brand/shop, match it; if ambiguous, ask which shop.
2. Ask the user (if not already given in this request) for the brand's approval criteria for this run — which factors matter and roughly how they should be weighted or what the bar is. Typical factors seen in practice (use only what the user actually specifies, don't assume all apply):
 - Niche relevance
 - Content quality
 - Posting consistency
 - Engagement
 - GMV or sales performance
 - Previous affiliate activity
 - Audience fit
 - Whether the creator has received samples before
 - Whether they posted after previous samples

 Do not bake in a fixed default checklist — always work from what the user states for that run.

## Step 2: Pull pending sample requests

Use `samples_list_samples_list_post` (filtered to pending/awaiting-review status if the endpoint supports it) for the shop to get the list of creators with open sample requests. If it's more useful to scope by product, `samples_by_product_endpoint_samples_by_product_post` can help.
## Step 3: Gather signals per creator

For each pending creator, pull enough signal to evaluate the user's criteria:

- `creator_detail_creators__creator_handle__get` — profile-level info (niche, audience fit signals, bio/category)
- `creator_videos_creators__creator_handle__videos_post` — recent content: posting cadence/consistency, content quality/topics, engagement
- `creators_performance_creators_performance_post` — TikTok Shop performance: GMV, sales, conversion where available
- Check sample history: has this creator received samples before, and did they post after receiving them (cross-reference with prior sample records from `samples_list_samples_list_post` filtered to this creator, or `samples_profiles_list` / `samples_product_usage_samples_product_usage_get` if more relevant)
- `crm_roster_creator_history` or `crm_roster_creator_detail` if the shop has CRM roster data and it adds useful history

Don't over-fetch — pull only the endpoints needed to evaluate the criteria the user gave you for this run.

## Step 4: Categorize

For each creator, apply the user's stated criteria and bucket into exactly one of:

- **Auto-approve** — clearly meets the criteria
- **Auto-reject** — clearly fails or has weak/no relevant activity
- **Needs manual review** — in-between, or signal is missing/ambiguous enough that it needs human judgment

Write a one-line reasoning for each, in the style of: "Recommended for approval because this creator has strong pet-related content, consistent posting, good engagement, and has generated sales from similar products." Be specific — cite the actual signal (numbers, posting frequency, past sample follow-through), not generic language.
## Step 5: Deliver as a Google Sheet

Build the results into a Google Sheet using the Google Drive connector (`mcp__Google_Drive__create_file`), rather than only a table in chat:

- One row per creator, with columns: Creator handle, Recommendation (Auto-approve / Auto-reject / Needs manual review), Reasoning, and any key signal columns worth surfacing at a glance (e.g. Niche, Posting consistency, Engagement, GMV, Prior samples).
- Add a header row with bold formatting if the create/update tool supports it; otherwise plain header text is fine.
- Sort or group so Needs manual review rows are easy to find (e.g. sort recommendation column, or put Needs manual review at the top) since that's the set requiring the user's attention.
- Add an empty "Decision" or "Reviewed by" column at the end for the human reviewer to fill in as they act on each row — this is what makes the sheet useful for collaborative review rather than a static report.
- Name the file clearly, e.g. "Sample Request Triage – [Shop/Brand] – [date]".
- If the user has a specific Drive folder they use for this brand/shop, ask once (or reuse one established earlier in the conversation) rather than defaulting to My Drive root every time.
- After creating the file, share the link back to the user with `mcp__Google_Drive__share_file` if needed for their teammates, and tell them where it landed.

If Google Drive isn't connected or the create call fails, fall back to presenting the table directly in chat (or another available spreadsheet output) and tell the user why the sheet wasn't created.

This skill only produces the report/sheet — it does not call any approve/reject action on Reacher. Tell the user the recommendations are ready for them to action manually, and ask if they want anything re-triaged with adjusted criteria.
## Notes

- Always call `list_shops` first per the Reacher connector's own instructions, and pass the resolved shop_id on every subsequent call.
- If sample volume is large, process in batches and give the user a running count rather than going silent until everything is done.
- If a creator is missing key data (e.g., no video history), don't guess — recommend "Needs manual review" and say why data was insufficient.
- On a re-run for the same shop, ask whether to create a new sheet or update the existing one (find it via `mcp__Google_Drive__search_files` by name) rather than always creating a duplicate.
