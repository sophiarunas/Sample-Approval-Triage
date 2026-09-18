---
name: daily-standup
description: "Generate Ember's daily standup update across Reacher shops — automations, sample approvals, videos added, and 7-day metrics — formatted as one paste-ready block per shop for Google Sheets."
---

# Daily Standup Updates

Pulls each shop's day in Reacher and produces one finished, paste-ready update per account, delivered as a single message with one code block per shop.

## Accounts

Reacher shop_ids to cover:
- Spongelle – 8707
- Little Monsters x Kair – 10094
- Sunny Isle US – 9864 (Sunny Isle Beauty)
- Sunny Isle UK – 10526 (report money in the shop's currency, e.g. £)
- True Botanicals – 8813

Timezone: Asia/Manila (UTC+8).
## Data to pull per shop

1. **Automations active in the last ~24h** — `automations_list` with start_date/end_date = yesterday..today (UTC). Summarize launches, messages sent, creators reached/remaining, sample requests, and any status like "Daily limit reached" or "Needs Attention".
2. **Samples approved in the last 12 hours** — `samples_list` with status "Sample Shipped" (exact value), keep rows whose `updated_at` is within the last 12 hours. Group by full product name with counts. Never include creator usernames. Also check the `samples_approved` timeseries for today; if Reacher looks behind TikTok (sync lag), note it briefly outside the code block.
3. **Videos added today, by product** — `videos_list` with start_date = end_date = today's UTC date, sort_by posted_date, page_size 100. Group by product.
4. **Video totals** — `videos_list` with page_size 1 and no dates → `pagination.total_count` = all-time total. Month-to-date: `videos_list` with start_date = 1st of current month, end_date = today, page_size 100, paging through all pages (large results get saved to a file — parse with python/json in bash). Group month-to-date videos by product and count. `products_list` (same date range, sort_by video_count) is a good cross-check for per-product video counts and exact product names.
5. **Last 7 days totals** — `timeseries_metrics` with metrics total_gmv, videos_posted, samples_approved, orders, granularity day; sum each. Note outside the block if recent days show 0 videos (likely sync lag).
## Product naming

Never merge distinct products — each product_id is its own line. On Little Monsters x Kair these are separate products and must never be combined or relabeled as each other:
- Little Monsters Mess Remover Spray 4 fl oz (1732417407191257517)
- Little Monsters Mess Remover Foaming Brush Bundle (1732586721253036461)
- 2-PACK Little Monsters Mess Remover Foaming Brush 5 fl oz (1732417406746530221)
- Little Monsters Complete Mess Remover Bundle — Spray, Foaming Brush & Wipes (1732610685270856109)
- Little Monsters Mess Remover Bundle (1732586823414288813)
- Little Monsters Mess Remover Wipes 5-pack (1732417406246162861)
- Kair Pillow Spray Mist (1732478447586087341), Kair Signature Clothing Wash (1732478441051427245), and the other Kair laundry SKUs

`product_name` is often null — resolve it from the same product_id elsewhere in the results, from `products_list`, or from `samples_list` titles; if still unknown write "Other product (not named in Reacher)"; a null product_id becomes "No product tagged". Shorten only the marketing tail of long TikTok titles (e.g. "Sunny Isle Jamaican Black Castor Oil PURE BUTTER 4oz") — never in a way that makes two different SKUs read the same. In the month breakdown, list products with 2+ videos individually, sorted high→low, and group the rest as "Other products (N)". If most videos for a shop come from the brand's own account, note that outside the block. No usernames anywhere.
## Output format

For each account, produce one finished update in its own code block with the account name as a heading above it, ready to paste into a Google Sheets cell. Send all of them to the user in one message.

Format each block exactly:

```
✅ Done:
• ...
📦 Samples approved – last 12 hrs (TOTAL):
• <Full product name> – <count>
🎥 Videos added – today (TOTAL):
• <Product name> – <count>
🎬 Video count:
• All time: X | <Month name> (so far): X
📊 <Month name> videos by product:
• <Product name> – <count>
🔄 Today:
• ...
📊 Numbers (last 7 days):
• GMV: $X | Videos posted: X | Samples approved: X | Orders: X
⚠️ Blockers:
• ...
```

If a section has nothing, write "• None". No usernames. Keep it concise. If a shop errors, say so and continue with the others.

## Notes

- Runs daily at 18:30 Asia/Manila as a scheduled routine.
