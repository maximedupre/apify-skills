# Review row fields

Required on every row: `rowType` (`review`), `title`, `url`, `mediaType` (`movie`, `tv`, or `season`), `reviewType` (`critic` or `audience`). Each row also has at least one of `reviewText`, `rating`, or `originalScore`. Other fields appear only when the source gives them.

| Field | Applies to | Meaning |
| --- | --- | --- |
| `reviewId` | both | Source review ID. |
| `reviewText` | both | Review text with HTML entities shown as readable characters. |
| `rating` | audience | Number on the source scale. |
| `originalScore` | critic | Critic's own score, with scale, such as `9/10`. |
| `sentiment` | both | Source sentiment label. |
| `createdAt`, `updatedAt` | both | Source creation date and update date. |
| `reviewer.name` | both | Reviewer name. |
| `reviewer.imageUrl` | critic | Profile image link. |
| `publication.name`, `.url`, `.imageUrl` | critic | Outlet details. |
| `reviewUrl` | critic | Full review on the outlet site. |
| `isTopCritic` | critic | Top Critic flag. |
| `isVerified` | audience | Verified audience flag. |
| `isSuperReviewer` | audience | Super-reviewer flag. |
| `hasSpoilers`, `hasProfanity` | audience | Source content flags. |
| `releaseYear`, `language` | both | Title year, review language. |

Example audience row:

```json
{"rowType":"review","title":"Top Gun: Maverick","url":"https://www.rottentomatoes.com/m/top_gun_maverick","mediaType":"movie","releaseYear":2022,"reviewType":"audience","reviewId":"5f592b65-0ec3-4dd6-8888-a72d3bcba92e","reviewText":"Just a perfect American movie. So watchable and re-watchable.","rating":5,"createdAt":"2025-09-22T02:43:52.597Z","reviewer":{"name":"Michael"},"isVerified":true,"hasSpoilers":false}
```

Example critic row:

```json
{"rowType":"review","title":"Breaking Bad: Season 1","url":"https://www.rottentomatoes.com/tv/breaking_bad/s01","mediaType":"season","reviewType":"critic","reviewText":"Cranston tackles the role with a measured intensity...","originalScore":"9/10","sentiment":"POSITIVE","reviewer":{"name":"Cindy White"},"publication":{"name":"IGN","url":"https://www.ign.com/"},"reviewUrl":"https://www.ign.com/articles/2009/02/27/breaking-bad-the-complete-first-season-dvd-review","isTopCritic":false}
```
