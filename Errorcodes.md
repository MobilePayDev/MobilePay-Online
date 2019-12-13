# Errors codes

The following will describe the error codes thrown and in which cases they can occur.

The error format will be the following:

```json
{
    "code": "2020",
    "message": "Some description",
    "correlationId": "8d72ece4-1b0b-464b-98d9-6bbb02199dc8"
}
```

## Error codes

| Code | Endpoint(s) | Description
|:---|:---|:---|
| 2000 | POST /payments | Merchant doesn't exist
| 2010 | POST /payments | The merchant isn't created by you
| 2020 | POST /payments | The merchant is deleted
| 2030 | POST /payments | Allowed card types are not set
| 2040 | POST /payments | One or more of the allowed card types are invalid
| 2050 | POST /payments | Currency code is invalid

## Allowed currencies

Currencies must be specified according to the ISO-4217 standard. Either using the alpha or the numeric version.
Allowed currencies are:

| Name | Alphabetic code | Numeric code
|:---|:---|:---|
| Danish kroner | DKK | 208
| Euro | EUR | 978
| Norwegian kroner | NOK | 578
| Swedish kroner | SEK | 752

## Allowed card types

The following card types are allowed:

| Card name | Code
|:---|:---|
| Visa electron | ELEC-DEBIT |
| Mastercard credit | MC-CREDIT |
| Mastercard debit | MC-DEBIT |
| Maestro | MTRO-DEBIT |
| Visa credit | VISA-CREDIT |
| Visa debit | VISA-DEBIT |
| Dankort | DANKORT |
