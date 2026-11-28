# Keyword results

- The Actor searches each keyword across the available selected countries until a result limit stops it. Omit `regions` or pass an empty list to search the United States. `searchCountry` identifies the search that found the room. TikTok may show creators from other countries, so use that field only to label search coverage.
- All selected countries share the `maxResultsPerKeyword` cap for each keyword.
- For recent chat, read `latestComments` when present. Use `text`, `sentAt`, and `author.handle` when available. Treat it as a sample from the run.
- For top gifters, read `gifters` when present. Report `gifters[].rank`, `gifters[].coinAmount` in coins, and available account details. The Actor saves gifters with positive coin amounts. Report a missing array as unavailable.

## Empty or incomplete keyword results

- If the log says the search could not finish, report the affected search as incomplete. If the task still needs it, retry only the affected inputs and keep the requested viewer filters.
- If the log says no live rooms were found, report that TikTok returned no live rooms for the keyword in the searches that ran.
- If the log says no rooms matched the selected options, report that the search found no eligible rooms under those options. Relax the viewer filters only if the task allows it.
- If the log says a country could not be searched or the proxy provider had no exits there, report missing coverage for that country. A result limit can also prevent searches in later countries.
- When the run reaches `maxItems`, it can skip later keywords without a warning for each skipped keyword. For a keyword with no saved rows and no warning, report coverage as unknown unless the log confirms what happened.
