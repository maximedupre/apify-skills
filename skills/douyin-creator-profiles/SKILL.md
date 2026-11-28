---
name: douyin-creator-profiles
description: "Collects public Douyin creator profiles, videos, and photo posts from profile links or IDs. Use when comparing follower counts across a creator shortlist or reviewing the recent posts and engagement of known Douyin creators."
---

# Douyin Creator Profiles and Posts Scraper

Choose a workflow below and save its adapted JSON as `input.json`. Put one creator per entry in `creators`. Use a full profile link, profile share link or message, `sec_uid`, or numeric user ID as a string.

See the [Actor Store page](https://apify.com/maximedupre/douyin-creator-profiles?fpr=qn4m5w), the [input schema](https://apify.com/maximedupre/douyin-creator-profiles/input-schema?fpr=qn4m5w) for all fields, and the [API tab](https://apify.com/maximedupre/douyin-creator-profiles/api?fpr=qn4m5w) for endpoint details. See [pricing](https://apify.com/maximedupre/douyin-creator-profiles/pricing?fpr=qn4m5w) for current rates. Starting a run incurs a charge even if it saves no rows. Profile and post rows have separate rates.

## Run and fetch results

1. Start the run with `APIFY_TOKEN` set in the environment. Read and save `data.id` from the JSON response as the run ID. If the response is lost, check recent runs before sending another POST to avoid a duplicate charge.

   ```sh
   curl -fsS -X POST 'https://api.apify.com/v2/acts/maximedupre~douyin-creator-profiles/runs' \
     -H "Authorization: Bearer $APIFY_TOKEN" \
     -H 'Content-Type: application/json' --data-binary @input.json
   ```

2. Replace `RUN_ID` with the saved run ID. Repeat this request about every five seconds until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. If a poll fails, resume with the same run ID. Read `data.defaultDatasetId` from the terminal response.

   ```sh
   curl -fsS 'https://api.apify.com/v2/actor-runs/RUN_ID' \
     -H "Authorization: Bearer $APIFY_TOKEN"
   ```

3. Replace `DATASET_ID` with that dataset ID and save the rows to `results.json`. If the run ended with `FAILED`, `ABORTED`, or `TIMED-OUT`, label any fetched rows as partial and report the run ID and status.

   ```sh
   curl -fsS 'https://api.apify.com/v2/datasets/DATASET_ID/items?format=json' \
     -H "Authorization: Bearer $APIFY_TOKEN" -o results.json
   ```

4. Check the run log before reporting results, including when the status is `SUCCEEDED`. Replace `RUN_ID` with the same run ID.

   ```sh
   curl -fsS 'https://api.apify.com/v2/actor-runs/RUN_ID/log' \
     -H "Authorization: Bearer $APIFY_TOKEN"
   ```

   If the log says “Some results could not be saved or read”, include that warning and the run ID in the creator comparison or post review. Describe the returned rows as partial coverage. If you cannot read the log, report that you could not verify coverage from it.

## Compare creator profiles

1. Replace the example creator with the shortlist. Set `maxItems` to the number of creators to allow a profile row for each.

   ```json
   {
     "resultType": "profiles",
     "creators": ["https://www.douyin.com/user/MS4wLjABAAAAGhTVHcr1tmaen43gOCk7JxI2ZHioZL3UzgmydgHUFFQ"],
     "maxItems": 1
   }
   ```

2. Run [Run and fetch results](#run-and-fetch-results) and read the profile rows in `results.json`.
3. Compare creators using `profileUrl`, `nickname`, `uniqueId`, `followerCount`, and `totalLikesReceived`. Keep each profile link with its counts so the reader can verify the match. Report which requested creators have no returned profile when you can identify them.

## Review recent posts

1. Replace the example creator with the creators to review. Set the date window and post cap for the review. Read [post filters and output](references/posts.md) if you need photo posts, profiles and posts in one run, a fixed cutoff date, pinned post exclusion, details on how filters and post formats affect the per-creator cap, or media links.

   ```json
   {
     "resultType": "posts",
     "creators": ["https://www.douyin.com/user/MS4wLjABAAAAGhTVHcr1tmaen43gOCk7JxI2ZHioZL3UzgmydgHUFFQ"],
     "postType": "videos",
     "recentPostsSince": "30 days",
     "maxPostsPerCreator": 10
   }
   ```

2. Run [Run and fetch results](#run-and-fetch-results) and select rows with `resultType: "post"`. Group by `author.secUid` when present. Keep rows with unknown authors separate.
3. Report `url`, `publishedAt`, and the requested engagement counts, such as `likeCount`, `commentCount`, and `shareCount`. Include the number of rows reviewed per creator so the reader can judge coverage.

## Gotchas

- Fields depend on what Douyin returns. Keep missing counts unknown so they do not skew comparisons.
- `maxItems` caps the whole run across creators. JSON input with `maxItems` omitted or set to `0` has no overall cap. The Store form starts at `1`.
