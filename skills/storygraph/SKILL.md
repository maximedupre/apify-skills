---
name: storygraph
description: "Scrapes StoryGraph books into structured rows with community rating, moods, pace, reader questions and content warnings. Use when shortlisting books by rating, comparing moods or pace, checking content warnings before recommending a book, or monitoring a search for new books on thestorygraph.com."
---

# StoryGraph Book Scraper

Runs the Apify Actor `maximedupre~storygraph`. Input is search queries (titles, authors, series), StoryGraph book links, or both. Output is one dataset row per book. Store page: https://apify.com/maximedupre/storygraph?fpr=qn4m5w

Needs the `APIFY_TOKEN` environment variable. Run the Actor with plain curl, as below.

## Run the Actor

1. Start the run. The body is the input JSON from a use case below. The response holds `data.id` (run id) and `data.defaultDatasetId`.

```bash
curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~storygraph/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"searchQueries":["fantasy"],"maxItems":10}'
```

2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`. Wait about 10 seconds between polls.

```bash
curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
```

3. Fetch the rows only after `SUCCEEDED`. On any other status, report the status and stop.

```bash
curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
```

Every input field: https://apify.com/maximedupre/storygraph/input-schema?fpr=qn4m5w. Cost: https://apify.com/maximedupre/storygraph/pricing?fpr=qn4m5w (pay per book saved, plus a small start fee). API details: https://apify.com/maximedupre/storygraph/api?fpr=qn4m5w

## Use case 1: Shortlist well-rated books on a topic

Use when the user wants the best books for a genre, author, or series.

1. Pick queries (`searchQueries`), such as a genre, author, or series name.
2. Set `minRating` (0 to 5, community average). Books below it are not saved and not charged.
3. Set `maxItems` to cap the run and its cost.
4. Run the Actor. Sort the rows by `averageRating`, and use `reviewCount` to drop books with too few ratings to trust.
5. Present `title`, `author`, `series`, `averageRating`, `reviewCount`, and `url`.

```json
{"searchQueries": ["Brandon Sanderson"], "minRating": 4, "maxItems": 30}
```

## Use case 2: Pick a book by mood and pace

Use when the user wants a fast, dark, funny, or slow read, or wants to compare candidates.

1. Collect candidates with `searchQueries`, or with `bookUrls` when the user already has specific books.
2. Run the Actor.
3. Rank rows by the wanted `moods[]` entry (`mood`, `percentage`) and by `pace.fast`, `pace.medium`, `pace.slow` (percent of readers).
4. Present the top books with the matching percentages. Also use `readerQuestions` for plot or character driven, and loveable or flawed characters.

```json
{"bookUrls": [{"url": "https://app.thestorygraph.com/books/ab737e82-11de-4f0c-8ba8-397669d51880"}], "searchQueries": ["cozy fantasy"], "maxItems": 20}
```

## Use case 3: Check content warnings before recommending

Use when the user must screen books for sensitive content, such as for a gift, a classroom, or a book club.

1. Put the exact books in `bookUrls` (or search titles in `searchQueries`).
2. Run the Actor.
3. Read `contentWarnings.graphic`, `.moderate`, and `.minor` for each row. Report graphic warnings first.
4. If a row has no `contentWarnings`, say that readers have not submitted any. Do not say the book is safe.

```json
{"bookUrls": [{"url": "https://app.thestorygraph.com/books/ab737e82-11de-4f0c-8ba8-397669d51880"}]}
```

## Use case 4: Monitor a search for new books

Use for a repeated watch on an author, series, or topic.

1. Run once with the same `searchQueries` and `bookUrls` each time and `onlyNewBooks` set to `true`. The first run returns all matching books and records them.
2. Each later run with the identical queries and links returns only books not seen before.
3. Report the new rows. An empty dataset means no new books.

```json
{"searchQueries": ["Patricia Briggs"], "onlyNewBooks": true}
```

## Gotchas

- Provide at least one search query or valid book link. With neither, the run does nothing.
- `bookUrls` items must be full `https://app.thestorygraph.com/books/<book-id>` links. Other links are skipped with a warning.
- `maxItems` empty or `0` means all available books, which can be large and costly. Always set it unless the user wants everything.
- Rows leave out fields StoryGraph has no value for (`series`, `readerQuestions`, `contentWarnings`). Check that a field exists before you read it.
- `onlyNewBooks` memory is tied to the exact set of queries and links. Changing either starts a fresh history.
