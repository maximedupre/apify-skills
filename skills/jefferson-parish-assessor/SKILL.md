---
name: jefferson-parish-assessor
description: "Looks up Jefferson Parish, Louisiana property tax assessor records by address, owner name, or parcel number through the Apify Actor maximedupre~jefferson-parish-assessor. Returns owner, assessed and market value, tax year, assessment lines, owner history, sales, and the official record link. Use when asked to find who owns a Jefferson Parish property, list an owner's parcels, check assessed values, or review ownership changes and past sales."
---

# Jefferson Parish Tax Assessor

Runs the Apify Actor [`maximedupre/jefferson-parish-assessor`](https://apify.com/maximedupre/jefferson-parish-assessor?fpr=qn4m5w) with curl. It returns one dataset row per assessor record (real estate or business personal property). Requires the `APIFY_TOKEN` environment variable.

Input fields: [input schema tab](https://apify.com/maximedupre/jefferson-parish-assessor/input-schema?fpr=qn4m5w). Cost: [pricing tab](https://apify.com/maximedupre/jefferson-parish-assessor/pricing?fpr=qn4m5w). API details: [api tab](https://apify.com/maximedupre/jefferson-parish-assessor/api?fpr=qn4m5w).

## Run protocol (all use cases)

1. Start the run. The body is the use case input. The response holds `data.id` and `data.defaultDatasetId`.
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~jefferson-parish-assessor/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"searchBy":"address","addresses":["200 DERBIGNY ST"],"maxItems":10}'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait a few seconds between polls.
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. Fetch the rows only when the status is `SUCCEEDED`.
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
4. Report the rows with their `recordUrl`, so the user can verify each record on the assessor site.

One run uses one `searchBy` type. Only the list for that type is read. For a different search type, start a second run.

Each field of the output rows: read [references/output-fields.md](references/output-fields.md) when you need a field name or its meaning.

## Use case 1: Everything at a street address

Use for "who owns 200 Derbigny St" or "what is on this lot". An address search returns the building record plus the business and unit records at that address.

1. Write each address as the assessor stores it: house number and street only, uppercase, no city, state, or zip.
2. Run with `searchBy: "address"`.
3. Check each row's `physicalAddress`. The search matches text inside the address, so `00 DERBIGNY ST` also returns 1000, 300, and 900 Derbigny St. Drop rows that are not the target.
4. Report `ownerName`, `mailingAddress`, `assessedValue`, `marketValue`, and `taxYear` per record.

```json
{"searchBy": "address", "addresses": ["200 DERBIGNY ST", "310 LAVOISIER ST"], "maxItems": 0}
```

## Use case 2: Every parcel an owner holds

Use for portfolio mapping of a person or company.

1. Write the name in `LAST,FIRST` form: comma, no space, uppercase. A company is written as the assessor lists it. Example: `SMITH,JOHN`.
2. Run with `searchBy: "ownerName"` and `maxItems: 0` to get every match.
3. The search matches text inside the name, so `SMITH,JOHN` can also match `SMITH,JOHNNY`. Compare `ownerName` and `ownerHistory` to confirm the owner.
4. Sum `assessedValue` and `marketValue` for the portfolio total. Business personal property rows have no `marketValue`; leave them out of market totals.

```json
{"searchBy": "ownerName", "ownerNames": ["SMITH,JOHN"], "maxItems": 0}
```

## Use case 3: Batch valuation of known parcels

Use when the user has parcel numbers and wants current values, tax year, and assessment breakdowns.

1. Pad each parcel number to 10 digits with leading zeros. `100000086` returns nothing; `0100000086` works.
2. Run with `searchBy: "parcelNumber"`.
3. Compare `assessedValue` and `marketValue` across rows. For a per-class breakdown, read `assessmentLines` (each has `propertyClass`, values, and `units`) against the stated `assessmentTotal`.
4. A parcel number with no row is missing from the assessor data. Report it as not found.

```json
{"searchBy": "parcelNumber", "parcelNumbers": ["0100000086", "0100000928"], "maxItems": 0}
```

## Use case 4: Ownership changes and past sales

Use for title, lien, and due-diligence research on a property.

1. Run a parcel search if the user has the parcel number, otherwise an address search.
2. Read `ownerHistory` for each real estate row: `ownerName`, `ownershipDate`, `isHomestead`, `isPrimaryOwner`, `ownershipPercentage`. Sort by `ownershipDate` to show when ownership changed.
3. Read `transactions`: `deedNumber`, `instrumentType`, `saleDate`, `amount`, `book`, `page`. An empty list means the assessor shows no sales. Use `book` and `page` to point the user to the clerk's deed records.
4. Business personal property rows carry no history or transactions.

```json
{"searchBy": "parcelNumber", "parcelNumbers": ["0100000928"]}
```

## Gotchas

- A list left empty for the chosen `searchBy` ends the run early and saves nothing, so fill the matching list.
- `maxItems` set to 0 or left out returns all matches. The form value 10 is only a prefill, so a cap applies only when you send one.
- An owner name written with a space (`SMITH JOHN`) returns no records, and a short parcel number returns no records. Fix the format before retrying.
- Fields the assessor does not state are missing from a row. Check for presence before computing.
- Duplicate inputs are searched once, and a parcel found twice is saved once.
