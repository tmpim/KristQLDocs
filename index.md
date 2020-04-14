# KristQL
*KristQL is still in beta. Please report any issues to Lemmmy.*

KristQL is an API allowing you to run arbitrary [SQL](https://dev.mysql.com/doc/refman/5.7/en/sql-syntax.html) queries against the Krist database. This allows you to create rich and complex queries to gather any data you like.

Of course, you can only run SELECT queries - you cannot modify the database.

# Examples

## Get the last 100 transactions
```GET https://query.krist.ceriat.net/query?q=SELECT * FROM `transactions` ORDER BY `id` DESC LIMIT 100```
```json
{
  "ok": true,
  "count": 100,
  "results": [
    {
      "id": 914947,
      "from": null,
      "to": "kya197vexi",
      "value": 1,
      "time": "2019-01-13T17:07:01.000Z",
      "name": null,
      "meta": null
    },
    ...
  ]
}
```

## Get blocks with value larger than 50
```GET https://query.krist.ceriat.net/query?q=SELECT * FROM `blocks` WHERE `value` > 50```
```json
{
  "ok": true,
  "count": 1951,
  "results": [
    {
      "id": 56823,
      "value": 52,
      "hash": "000000030d9962002ceb50da18202ee6a7e9dee9d2dee6e214e4a0418b8e3e77",
      "address": "k3s72l1pfa",
      "time": "2015-05-10T18:06:20.000Z",
      "difficulty": 2328677
    },
    ...
  ]
}
```

# Available Tables
## Addresses
The `addresses` table contains all addresses known to Krist, as well as their balances, and when they were first seen. It has the following fields:
* `id` - The ID of the address.
* `address` - The address itself.
* `balance` - The amount of Krist stored in the address.
* `firstseen` - The time and date the address was first seen by the Krist server. In JSON, this is an [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) string. In CSV, this is a [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) in milliseconds.

## Blocks
The `blocks` table contains all blocks that have been mined. It has the following fields:
* `id` - The ID, or 'height' of the block.
* `value` - The amount of Krist that this block rewarded.
* `hash` - The full 256-bit hash of this block, as a hexadecimal string.
* `address` - The address that mined this block.
* `time` - The time that this block was mined. In JSON, this is an [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) string. In CSV, this is a [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) in milliseconds.
* `difficulty` - The work at the time this block was mined. For the first few thousand blocks, these values may be inaccurate.

## Transactions
The `transactions` table contains all transactions that have been made. It has the following fields:
* `id` - The ID of this transaction.
* `from` - The address that sent this transaction. If the transaction is a mined block, this value can be null.
* `to` - The address that this transaction was sent to. If this transaction was a name purchase, this value will be `name`. For A record changes on a name, this value will be `a`.
* `value` - The amount of Krist transferred in this transaction. For A record changes on a name, this value can be 0.
* `time` - The time that this transaction occured. In JSON, this is an [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) string. In CSV, this is a [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) in milliseconds.
* `name` - The name associated with this transaction. This can be set for name purchases, A record changes, name transfers, or paying to names. If `value` is 0, then it is a name transfer. Otherwise, it is money being paid the owner of a name.
* `meta` - The metadata of this transaction. For A record changes, this is the new A record of the name.

## Names
The `names` table contains all names that have been purchased. It has the following fields:
* `id` - The ID of this name.
* `name` - The name itself.
* `owner` - The address that currently owns this name.
* `registered` - The date and time that this name was registered. In JSON, this is an [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) string. In CSV, this is a [Unix timestamp](https://en.wikipedia.org/wiki/Unix_time) in milliseconds.
* `updated` - The date and time that this name was last transferred, or its A record was last updated.
* `a` - The A record of this name.
* `unpaid` - The amount of blocks remaining until this name is fully paid.

# The API
The query API endpoint is `https://query.krist.ceriat.net/query?q=SELECT`. You can use GET or POST, and supply the query as a query or body parameter.

## Formats
There are three return formats supported: [JSON](https://en.wikipedia.org/wiki/JSON), compact JSON, and [CSV](https://en.wikipedia.org/wiki/Comma-separated_values).

### JSON
[JSON](https://en.wikipedia.org/wiki/JSON) is the default, but you can explicitly set it with `Accepts: application/json`. Output will look like this:

```json
{
  "ok": true,
  "count": 2,
  "results": [
    {
      "id": 1,
      "from": null,
      "to": "0000000000",
      "value": 50,
      "time": "2015-02-14T16:44:40.000Z",
      "name": null,
      "meta": null
    },
    ...
  ]
}
```

### Compact JSON
You can also get compact JSON with the `?compact` query string parameter. It will return a `fields` array, and the `results` will be an array of arrays:

```json
{
  "ok": true,
  "count": 2,
  "fields": [ "id", "from", "to", "value", "time", "name", "meta" ],
  "results": [
    [ 1, null, "0000000000", 50, "2015-02-14T16:44:40.000Z", null, null ],
    [ 2, null, "a5dfb396d3", 50, "2015-02-14T20:42:30.000Z", null, null ]
  ]
}
```

### CSV
Alternatively, you can get the results as [CSV](https://en.wikipedia.org/wiki/Comma-separated_values). You can either do this by specifying the `?csv` query string parameter, or with the `Accepts: text/csv` header. Null values will be empty cells:

```csv
id,from,to,value,time,name,meta
1,,0000000000,50,1423932280000,,
2,,a5dfb396d3,50,1423946550000,,
```

## Errors
The API will return the appropriate HTTP status code for all requests. Additionally, if you are using the JSON format, responses will contain the `ok` parameter. If `ok: false`, it should also contain an error message:

```json
{
  "ok": false,
  "error": "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\"' at line 1"
}
```

## Rate Limits
Currently, you can perform up to 10 requests per minute per IP address. Queries have a 30 second time limit.
