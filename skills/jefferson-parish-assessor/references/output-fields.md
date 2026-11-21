# Output fields

Each dataset row is one assessor record. Fields the assessor does not state are left out.

| Field | Type | Meaning |
| --- | --- | --- |
| `parcelNumber` | string | 10-digit parcel number with leading zeros |
| `recordKind` | string | `Real estate` or `Business personal property` |
| `physicalAddress` | string | Address of the property or business |
| `ownerName` | string | Owner name, `LAST,FIRST` form |
| `mailingAddress` | string | Owner mailing address |
| `assessedValue` | number | Assessed value, USD |
| `marketValue` | number | Market value, USD (real estate only) |
| `taxYear` | string | Assessment tax year |
| `recordUrl` | string | Link to the official assessor record |
| `legalDescription`, `ward`, `parcelType`, `subdivision`, `block`, `lot` | string | Parcel details, when listed |
| `assessmentLines[]` | objects | `propertyClass`, `assessedValue`, `marketValue`, `units` |
| `assessmentTotal` | object | Stated `assessedValue`, `marketValue`, `units` |
| `ownerHistory[]` | objects | `ownerName`, `ownershipDate`, `isHomestead`, `isPrimaryOwner`, `ownershipPercentage` |
| `transactions[]` | objects | `deedNumber`, `instrumentType`, `saleDate`, `amount`, `book`, `page` |

Example row:

```json
{
  "parcelNumber": "0100000928",
  "recordKind": "Real estate",
  "physicalAddress": "310 LAVOISIER ST",
  "ownerName": "WILSON,AARON M",
  "assessedValue": 19000,
  "marketValue": 190000,
  "taxYear": "2026",
  "recordUrl": "https://www.jpassessor.com/Parcel/Details?ParcelID=0100000928",
  "assessmentTotal": {"assessedValue": 19000, "marketValue": 190000, "units": "3.00"}
}
```
