# Beer row fields

Read this only when the user needs check-in detail or exact field names.

- Beer row: `recordType` (`beer`), `beerId`, `beerUrl`, `beerName`, `breweryName`, `breweryUrl`, `beerStyle`, `abvPercent`, `ibu`, `globalRating` (0 to 5), `ratingCount`, `totalCheckins`, `totalCheckinsText`, `description`, `imageUrl`, `sourceRank` (`null` for direct beer targets), `checkins`.
- `checkins[]` holds the public check-ins Untappd shows, not a full history. It is empty when none are shown. Each item has `checkinId`, `checkinUrl`, `checkinDate`, `userName`, `rating`, `comment`, `servingStyle`, `venueName`, `badges`.
- Full field list with types: https://apify.com/maximedupre/untappd-scraper/api?fpr=qn4m5w
