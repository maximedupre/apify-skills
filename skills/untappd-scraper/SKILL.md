---
name: untappd-scraper
description: "Collects public Untappd beers (ratings, ABV, IBU, check-ins), breweries, and venues (address, contact, drink menus) by search term, link, or ID by running the Untappd Scraper Apify Actor. Use when the user wants Untappd beer ratings, a brewery's full beer list, a top-rated beer chart filtered by style or ABV, brewery or competitor research, or bar and bottle-shop venue lookups with drink menus."
metadata:
  actor: "maximedupre/untappd-scraper"
---

# Untappd Scraper

Runs the Apify Actor `maximedupre~untappd-scraper` with curl. Input is search terms, Untappd links, or IDs plus filters. Output is one dataset row per public beer, brewery, or venue; `recordType` says which. No Untappd login is needed. Store page: https://apify.com/maximedupre/untappd-scraper

Requires the `APIFY_TOKEN` environment variable. All use cases share the run steps below.

## Run the Actor

1. Start the run with the use case's input. Save `data.id` and `data.defaultDatasetId` from the response.
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~untappd-scraper/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"resultType":"beers","targets":["Pliny the Elder"],"maxRecords":10}'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait 10 seconds between polls.
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. On `SUCCEEDED`, fetch the rows.
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
4. Done when the status is terminal and the dataset is fetched. On any other terminal status, or an empty dataset, follow Gotchas.

Every input field: https://apify.com/maximedupre/untappd-scraper/input-schema. Cost: https://apify.com/maximedupre/untappd-scraper/pricing. Always set `maxRecords`; empty or `0` means every available row, which can be a long, costly run.

## Use case 1: Beer ratings for named beers

For comparing ratings, ABV, IBU, and check-in totals of specific beers.

1. Put beer names, beer links (`https://untappd.com/b/<slug>/<id>`), or numeric beer IDs in `targets`. Set `maxRecords`.
2. Run the Actor with `{"resultType":"beers","targets":["Pliny the Elder","Heady Topper"],"maxRecords":20}`.
3. Present `beerName`, `breweryName`, `beerStyle`, `abvPercent`, `globalRating`, `ratingCount`, `totalCheckins`, `beerUrl`. A search term can match many beers, so the first rows per term are the best matches (`sourceRank`).
4. Visible public check-ins are in the nested `checkins` array. Read [references/output-fields.md](references/output-fields.md) only when the user needs them.

## Use case 2: Everything a brewery makes

For a brewery's full public beer list, such as competitor or portfolio research.

1. Take the brewery link (`https://untappd.com/w/<slug>/<id>`) from the user, or find it with use case 3.
2. Put the link in `targets`. All its public beers are saved, including beers no longer in production and homebrew. Filters do not apply to a brewery link, and the rows have no in-production flag, so say so if the user wants only current beers.
3. Run with `{"resultType":"beers","targets":["https://untappd.com/w/sierra-nevada-brewing-co/1142"],"maxRecords":100}`.

## Use case 3: Find and profile breweries

For brewery lists by name, keyword, or country: location, type, beer count, website, social links, parent company, rating.

1. Put names, keywords, or brewery links in `breweryTargets`. Add `country` exactly as Untappd writes it (`United States`, `England`, `Scotland`).
2. Run with `{"resultType":"breweries","breweryTargets":["sierra nevada"],"country":"United States","maxRecords":20}`.
3. Present `breweryName`, `breweryType`, `city`, `country`, `beerCount`, `website`, `breweryUrl`. The `breweryUrl` feeds use case 2.

## Use case 4: Top-rated beers by style, ABV, or country

For a ranked chart of the best beers with filters, with no target needed.

1. Leave `targets` empty. Add `beerStyles` (exact Untappd names such as `IPA - Imperial / Double`), `minRating`, `minRatingCount`, `minCheckins`, `minAbv`, `maxAbv`, and `country` as the user asked.
2. Run with `{"resultType":"beers","beerStyles":["IPA - Imperial / Double"],"minRating":4.2,"minRatingCount":1000,"country":"United States","maxRecords":50}`.
3. Rows keep the chart order in `sourceRank`. `beerSortOrder` does not change this chart.

## Use case 5: Venues and drink menus

For finding bars, taprooms, and bottle shops with address, contact details, hours, and menus.

1. Put venue names, links, or IDs in `venueTargets`, or leave it empty for the most popular venues worldwide. Add `venueCity` to narrow name searches and `venueVerifiedOnly` for owner-verified venues.
2. Run with `{"resultType":"venues","venueTargets":["taproom"],"venueCity":"Portland","maxRecords":20}`.
3. Extra menus (`includeVenueExtraMenus`) and check-in comments (`includeVenueCheckinComments`) each add a charge per item read. Turn them on only when asked, and keep `maxVenueExtraMenus` and `maxVenueCheckinComments` low.

## Gotchas

- Filters in use cases 1 to 4 apply to search terms and the top-rated chart. A beer link, brewery link, or ID is saved as is.
- A search term with no public match saves no rows and the run still ends `SUCCEEDED`. An empty dataset after a `SUCCEEDED` run means no match found or filters too strict; the run log shows `No public Untappd beer matched "<term>"` for each missed term. Loosen filters, then retry with a broader term or a direct link before telling the user nothing exists.
- A `FAILED` run with an Untappd access error means Untappd did not answer after retries (the Actor already rotates its connection on each retry). Start a new run later; do not change input to work around it.
- `country` must match Untappd's spelling; a wrong name finds nothing. Venues use the local spelling (`Deutschland`, not `Germany`). There is no `United Kingdom`; use `England`, `Scotland`, `Wales`, or `Northern Ireland` for breweries.
- Missing beer values are `null`; optional brewery and venue values are left out.
