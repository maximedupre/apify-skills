# Post search filters

Set `startDate` and `endDate` with dates or UTC timestamps for a specific period. If the start date is omitted, the search starts 30 days ago unless a `since:` date applies to the entire query. An `until:` date alone keeps the 30-day start. Query dates and date fields both apply when supplied.

To include older posts, set `startDate` to the earliest post date needed or earlier. For a conversation lookup by ID or URL, use the original post's date if you need to include it and all available replies within the date window.

Use `language`, `links`, `media`, and the engagement filters from the input schema to narrow the results. Use `postKinds` to include reposts when needed. The default includes original posts, quote posts, and replies. Keep the query and added filters within X's 512-character search limit because an overlong search returns no posts.
