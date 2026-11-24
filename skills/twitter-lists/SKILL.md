---
name: twitter-lists
description: "Scrapes X (Twitter) lists with the Apify Actor maximedupre/twitter-lists: discovers public lists by keyword, exports list members, subscribers, tweets, and list details (name, owner, member count). Use when a task needs to find X lists on a topic, build an expert or lead list from list members, see who subscribes to a list, or read the tweets of a list."
---

# X list scraper

Runs the Apify Actor `maximedupre~twitter-lists` with curl and the `APIFY_TOKEN` environment variable. Each run saves one kind of row, chosen by `resultType`. Store page: https://apify.com/maximedupre/twitter-lists?fpr=qn4m5w. Every input field: https://apify.com/maximedupre/twitter-lists/input-schema?fpr=qn4m5w. Cost: https://apify.com/maximedupre/twitter-lists/pricing?fpr=qn4m5w.

## Run protocol (all use cases)

1. Start the run. The response holds `data.id` and `data.defaultDatasetId`.
```bash
curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~twitter-lists/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '<input JSON>'
```
2. Poll every 5 to 10 seconds until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`.
```bash
curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
```
3. Fetch the rows only after `SUCCEEDED`.
```bash
curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
```

Set `maxItems` on every run. It caps the rows of the whole run, not each list, and so caps the cost; 0 or empty means all available rows.

## 1. Find public lists on a topic

Use to discover curated lists by keyword, or to compare lists by size.

1. Run with `resultType` `list-details`, `findListsBy` `keyword-search`, and your `keywords`.
2. Rank rows by `memberCount` and `subscriberCount`.
3. Keep `listId` or `url` of the chosen lists for use cases 2 to 4.

```json
{"resultType":"list-details","findListsBy":"keyword-search","keywords":["machine learning","climate tech"],"maxItems":50}
```

## 2. Build an expert or lead list from list members

1. Run with `resultType` `list-members` and `lists` set to list URLs or IDs.
2. Each row is a public profile: `username`, `name`, `description`, `location`, `url`, `followersCount`, `isVerified`.
3. Sort by `followersCount` or filter by bio keywords.

```json
{"resultType":"list-members","lists":["https://x.com/i/lists/1234567890"],"maxItems":500}
```

## 3. See who subscribes to a list

1. Run with `resultType` `list-subscribers` and `lists`.
2. Rows have the same profile fields as members.

```json
{"resultType":"list-subscribers","lists":["1234567890"],"maxItems":500}
```

## 4. Read what the accounts of a list post

1. Run with `resultType` `list-tweets` and `lists`.
2. Rows hold `text`, `createdAt`, `language`, `replyCount`, `retweetCount`, `likeCount`, `viewCount`, and the `author` profile.

```json
{"resultType":"list-tweets","lists":["1234567890"],"maxItems":200}
```

## 5. Get details of known lists

1. Run with `resultType` `list-details`, `findListsBy` `urls-or-ids`, and `lists`.
2. Rows hold `name`, `description`, `owner`, `memberCount`, `subscriberCount`, `isPrivate`, `createdAt`.

```json
{"resultType":"list-details","findListsBy":"urls-or-ids","lists":["1234567890"],"maxItems":50}
```

## Gotchas

- One `resultType` per run. For members and tweets of the same lists, start two runs.
- `lists` takes list URLs (`https://x.com/i/lists/<id>`) or bare numeric IDs. It is required for every result type except `list-details` with `keyword-search`.
- `keywords` is used only with `findListsBy` `keyword-search`; `findListsBy` applies only to `list-details`. An omitted `findListsBy` means `urls-or-ids`.
- Keyword search finds public lists only. A list matching several keywords is saved once, and duplicate list inputs are read once.
- Read `data.status` before fetching: a `FAILED` run can still hold partial rows.
