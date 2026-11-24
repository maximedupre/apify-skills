---
name: snapchat-ads
description: "Scrapes Snapchat's public Ads Gallery (commercial ads by advertiser and country), the political and advocacy ad archive, and sponsored creator content into structured rows. Use when researching a brand's Snapchat ads, competitor ad impressions, ad targeting, political ad spend, or creator sponsorships."
---

# Snapchat Ads Scraper

Runs the Apify Actor `maximedupre~snapchat-ads` with curl and the `APIFY_TOKEN` environment variable. It reads Snapchat's public pages only; no Snapchat login is needed. Store page: https://apify.com/maximedupre/snapchat-ads?fpr=qn4m5w

Every input field is listed at https://apify.com/maximedupre/snapchat-ads/input-schema?fpr=qn4m5w. Prices are at https://apify.com/maximedupre/snapchat-ads/pricing?fpr=qn4m5w. Pick the use case below that matches the request, then follow the run protocol.

## Run protocol (all use cases)

1. Start the run. Send the token in a header so it stays out of URLs:
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~snapchat-ads/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '<INPUT JSON>'
   ```
   Read `data.id` and `data.defaultDatasetId` from the response.
2. Poll `GET https://api.apify.com/v2/actor-runs/<runId>` with the same header every 5-10 seconds until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`.
3. Fetch rows: `GET https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json` with the same header.
4. If the status is not `SUCCEEDED`, or the dataset is empty, read the run log at `GET https://api.apify.com/v2/actor-runs/<runId>/log` before retrying. An empty dataset with `SUCCEEDED` means no match (see Gotchas).

Each row has a `resultType` field that tells which shape it has.

## Use case 1: Audit a brand's commercial Snapchat ads in the EU/EEA

Use for competitor ad research: which creatives run, their impressions per country, targeting, and landing pages.

1. Set `resultType` to `commercialAds`, `advertisers` to the brand names, and `countries` to two-letter codes (Greece is `el`).
2. Add filters only when asked: `commercialStatus` (`active`, `paused`, `both`), `commercialMinImpressions`, `commercialMediaKinds`, date fields.
3. Set `maxItems` to cap rows. Start with a small number to check the match.
4. Run the protocol and read `advertiser`, `headline`, `adStatus`, `startDate`, `totalImpressions`, `impressionsByCountry`, `targeting`, `destinationUrls`.

```json
{"resultType":"commercialAds","advertisers":["Amazon"],"countries":["de"],"commercialStatus":"active","commercialMinImpressions":100000,"maxItems":50}
```

Read [references/commercial-filters.md](references/commercial-filters.md) when the request uses date windows, exact advertiser matching, or media kinds.

## Use case 2: Pull political and advocacy ads with spend

Use for journalists or researchers tracking political ad spend, targeting, and candidate or ballot text.

1. Set `resultType` to `politicalAds` and `politicalYears` to strings such as `["2024"]` (at least one year; allowed values are on the input-schema page).
2. Narrow with `advertisers` (see Gotchas for its default), `politicalKeyword`, or `countries`. Here `countries` takes one full lowercase name such as `united states` or `germany`, not a code.
3. Set `maxItems`, run the protocol.
4. Read `spend.amount`, `spend.currency`, `impressions`, `organization`, `country`, `candidateOrBallotInformation`, `targeting`, `creative`.

```json
{"resultType":"politicalAds","politicalYears":["2024"],"countries":["united states"],"politicalKeyword":"election","maxItems":50}
```

## Use case 3: Find sponsored content by a creator

Use to check which brands sponsor a creator.

1. Set `resultType` to `sponsoredCreatorContent` and `creatorName` to one name. One name per run: start separate runs for more creators.
2. Set `maxItems`, run the protocol.
3. Read `creator` and `content` links in each row.

```json
{"resultType":"sponsoredCreatorContent","creatorName":"example creator","maxItems":20}
```

## Gotchas

- Only fields for the chosen `resultType` apply. Fields for other surfaces are ignored, so a mismatch gives wrong results silently, not an error.
- The input schema prefills `maxItems` to `1` and `countries` to `["de"]` in the web form. Through the API there are no such defaults: omitting `maxItems` returns every match, which costs more.
- Commercial ads cover only the EU/EEA gallery. For a non-EU market use political ads or tell the user it is out of scope.
- `advertisers` defaults to `["Amazon"]` when omitted, for commercial and political runs, so an omitted field silently filters to Amazon. Commercial ads need at least one advertiser. For political ads with no advertiser filter, pass `"advertisers": []`. An unsupported commercial country code fails the run with a message in the log.
- Optional fields are absent when Snapchat does not publish them. Do not treat absence as zero.
- The same source ad appears once per run; the first match is kept.
- Many media links are private on Snapchat's side and are left out of `mediaUrls`.
