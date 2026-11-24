---
name: tiktok-shop-shops
description: "Gets TikTok Shop shops ranked by 7-day sales, or found by keyword, in 15 markets, with revenue, units sold, growth, average price, products on sale, 28 days of daily revenue and top product IDs. Use when the user wants top TikTok Shop sellers, competitor shops in a niche, small or rising shops, or a market comparison of TikTok Shop stores."
---

# TikTok Shop Shops Scraper

Runs the Apify Actor `maximedupre~tiktok-shop-shops` and returns one dataset row per shop. Store page: https://apify.com/maximedupre/tiktok-shop-shops?fpr=qn4m5w

All figures cover the last 7 days, except `dailyRevenue` (28 days). Each search returns at most 10 shops: a top shops run saves up to 10, and a keyword run saves up to 10 for each keyword.

## Run protocol

Needs the `APIFY_TOKEN` environment variable. Use these three calls for every use case below.

1. Start the run. Read `data.id` and `data.defaultDatasetId` from the response.

```bash
curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~tiktok-shop-shops/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"findBy":"topShops","market":"US","maxItems":10}'
```

2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`. Wait a few seconds between polls.

```bash
curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
```

3. Fetch the shops (only if the status is `SUCCEEDED`).

```bash
curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
```

Every input field is listed at https://apify.com/maximedupre/tiktok-shop-shops/input-schema?fpr=qn4m5w. Cost: https://apify.com/maximedupre/tiktok-shop-shops/pricing?fpr=qn4m5w. Other API calls: https://apify.com/maximedupre/tiktok-shop-shops/api?fpr=qn4m5w

## Use case 1: Rank the top shops of a market

For sellers, agencies and analysts who want to know who sells most.

1. Pick the `market` (US, GB, ID, TH, VN, MY, PH, SG, JP, BR, MX, ES, FR, DE, IT).
2. Pick `sortBy`: `revenue`, `unitsSold` or `productsOnSale`.
3. Run with `findBy` set to `topShops`.
4. Report each shop by `name`, `revenue`, `unitsSold`, `revenueGrowth` and `shopUrl`, in the dataset order.

```json
{"findBy":"topShops","market":"US","sortBy":"revenue","sortOrder":"desc","currency":"USD","maxItems":10}
```

## Use case 2: Find competitor shops in a niche

For sellers who need the shops that sell a product type.

1. Choose short, specific keywords, for example `pet grooming`. Put one keyword in each array entry.
2. Run with `findBy` set to `keywords` and the `keywords` array.
3. Group the rows by `matchedKeyword`. A shop appears once, under the first keyword that found it.
4. Use `topProductIds` to look up the best products of a shop, and `dailyRevenue` to check whether its sales rise or fall.

```json
{"findBy":"keywords","keywords":["pet grooming","dog toys"],"market":"US","sortBy":"revenue","currency":"USD"}
```

## Use case 3: Find small or rising shops

For dropshippers and scouts who look for shops that are not yet large.

1. Set `sortOrder` to `asc`, or set `minRevenue` and `maxRevenue` to a band, or set `minAveragePrice` and `maxAveragePrice`.
2. Run with `findBy` `topShops` or `keywords`.
3. Keep rows with a high `revenueGrowth`. Check the last 7 values of `dailyRevenue` to confirm a trend.

```json
{"findBy":"keywords","keywords":["phone case"],"market":"US","minRevenue":1000,"maxRevenue":50000,"maxAveragePrice":30}
```

## Use case 4: Compare markets

For analysts who compare the same ranking across countries.

1. Run once for each market with the same input except `market`, as in the example.
2. Start all runs, then poll each run.
3. Compare `revenue`, `averagePrice` and `productsOnSale` across the result sets.

```json
{"findBy":"topShops","market":"GB","sortBy":"revenue","currency":"USD","maxItems":10}
```

## Output fields

`shopId`, `name`, `shopUrl`, `imageUrl`, `matchedKeyword` (keyword runs only), `currency`, `revenue`, `unitsSold`, `revenueGrowth` (percent against the 7 days before), `averagePrice`, `productsOnSale`, `dailyRevenue` (array of `{date, revenue}`), `creatorCount`, `productCount`, `videoCount`, `liveStreamCount`, `topProductIds`.

## Gotchas

- `revenueGrowth` and `averagePrice` are missing when there is nothing to compare or no sales. Handle absent fields.
- The filters apply to the shops a search returns, so they can leave fewer than 10 shops.
- A minimum above its maximum stops the run with a message. Check the run status and fix the input.
- `shopUrl` opens the store page for US and UK shops. In other markets it may not open.
- Use `currency` `USD` when you compare markets. `LOCAL` gives each market its own currency, so the rows do not compare.
- `findBy` `keywords` needs at least one keyword. The `keywords` field is ignored when `findBy` is `topShops`.
- `maxItems` of 0 or empty means no limit. In keyword runs a low value caps the total across all keywords, so leave it empty to get every keyword.
