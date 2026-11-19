---
name: viewstats
description: "Scrapes Viewstats YouTube channel analytics (subscribers, views, rankings, growth, revenue estimate ranges, forecasts, similar channels, optional long-form videos and Shorts) from handles, channel links, or keywords. Use when comparing YouTube channels, vetting a creator before a sponsorship, discovering channels in a niche, or tracking a channel list for changes."
metadata:
  actor: "maximedupre/viewstats"
---

# Viewstats YouTube channel analytics

Runs the Apify Actor `maximedupre~viewstats`. One dataset row is one channel. Store page: https://apify.com/maximedupre/viewstats. Every input field: https://apify.com/maximedupre/viewstats/input-schema. Prices: https://apify.com/maximedupre/viewstats/pricing.

## Run protocol

Needs the `APIFY_TOKEN` environment variable. Any run uses these three calls.

1. Start the run. The response holds `data.id` and `data.defaultDatasetId`.
```bash
curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~viewstats/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"findBy":"channels","channels":["@MrBeast","@YouTube"]}'
```
2. Poll `GET https://api.apify.com/v2/actor-runs/<runId>` with the same header every 5-10 seconds until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Runs are asynchronous, so there is no time limit on the wait.
3. Fetch rows with `GET https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json` and the same header. Add `&fields=handle,subscriberCount` to cut size.

Report any status other than `SUCCEEDED` with the run's `statusMessage`; the dataset can still hold partial rows.

## Use case 1: Compare known channels

Goal: one table of stats and ranks for a list of rival or partner channels.

1. Collect handles (`@name`) or `youtube.com/@handle` / `youtube.com/channel/UC...` links.
2. Run with this input:
```json
{"findBy":"channels","channels":["@MrBeast","@YouTube"]}
```
3. Read `subscriberCount`, `viewCount`, `videoCount`, `viewsPerVideo90Days`, and `rankings.*` from each row.
4. Compare on one metric and name the metric in the answer.

## Use case 2: Vet a creator before a sponsorship

Goal: judge reach, momentum, and value of one creator.

1. Run with the channel and both video lists on, so recent performance is visible:
```json
{"findBy":"channels","channels":["@MrBeast"],"includeLongVideos":true,"maxLongVideosPerChannel":10,"includeShorts":true,"maxShortsPerChannel":10}
```
2. Read `growth` (periods `7days`, `28days`, `90days`, `365days`, `allTime`) for subscriber and view change. Read `rankings` for position.
3. Read `growth[].revenueEstimate.low` and `.high`. Viewstats states this range; present it as an estimate, never as reported earnings.
4. Read `longsVsShorts` to see which format drives views.
5. For video field names and typical view ranges, read [references/output-fields.md](references/output-fields.md).

## Use case 3: Discover channels in a niche

Goal: a ranked list of channels for a topic.

1. Pick 1-5 keywords (topics or channel names). Each returns up to 30 channels.
2. Run:
```json
{"findBy":"keywords","keywords":["minecraft speedrun"],"maxChannelsPerKeyword":30}
```
3. Each row has `keyword` showing which search found it. A channel found by two keywords appears once.
4. Sort by `growth` entries for `90days` (`subscriberChange`) to find rising channels, or by `subscriberCount` for size. Use `similarChannels` to widen the list.

## Use case 4: Track a channel list for changes

Goal: on a schedule, learn only which channels changed.

1. Run the same `channels` list each time with `"onlyChanged": true`.
2. The first run returns every channel; later runs return only new channels or those whose Viewstats data changed.
3. An empty dataset on a later run means nothing changed; say so rather than treating it as a failure.

## Gotchas

- Input accepts `@handle`, a handle link, or a `youtube.com/channel/UC...` link. `/c/` and `/user/` links and bare channel IDs are skipped with a warning in the run log, so convert them first.
- `findBy` picks which list is read. `channels` is ignored when `findBy` is `keywords`, and the reverse, so set `findBy` to match the list you filled.
- `maxItems` of 0 or empty means no limit, so API calls get every channel unless you set a cap. Same for `maxChannelsPerKeyword` (limit is 30 per keyword).
- A handle Viewstats does not know gives no row, only a log warning. Compare returned handles against the requested ones.
- Cost: each channel row, each video row, and the run start are charged (pricing tab). Videos cost the most per row, so leave `includeLongVideos` and `includeShorts` off unless the task needs them, and set `maxItems` and the per-channel video caps to bound spend.
