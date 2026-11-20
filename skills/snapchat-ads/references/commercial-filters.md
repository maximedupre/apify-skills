# Commercial ads filters

- `commercialDateFrom` / `commercialDateTo`: activity window, `YYYY-MM-DD` in UTC. From must not be after to.
- `commercialStartDateFrom` / `commercialStartDateTo`: keep ads whose campaign start date is inside this range.
- `commercialAdvertiserMatch`: `contains` (default) matches any advertiser name containing the text; `exact` matches only the paying advertiser name exactly. Use `exact` when a brand name is a substring of many others.
- `commercialMediaKinds`: any of `video`, `image`, `productCatalogue`. Empty means all.
- `commercialMinImpressions`: whole number, 0 or more, compared to `totalImpressions`.

Example, exact advertiser, active video ads started in 2025 in France:

```json
{"resultType":"commercialAds","advertisers":["Nike"],"commercialAdvertiserMatch":"exact","countries":["fr"],"commercialStatus":"active","commercialMediaKinds":["video"],"commercialStartDateFrom":"2025-01-01","commercialStartDateTo":"2025-12-31","maxItems":100}
```
