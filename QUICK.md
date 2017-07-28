LivingSocial API Design Quick Reference
=======================================

## Version 1.0.0 - Published 2016 Mar 10
This is a quick reference sheet of the [Complete Guide](README.md).

| Quick Reference |  |
| --- | --- |
| [Requests](#requests) | [(Guide)](README.md#requests)                                     |
| [Versioning](#versioning) | [(Guide)](README.md#versioning)                               |
| [Pagination](#pagination) | [(Guide)](README.md#pagination)                               |
| [Filter/Sort/Search](#filtersortsearch) | [(Guide)](README.md#filtersortsearch)           |
| [Content Negotiation](#content-negotiation) | [(Guide)](README.md#content-negotiation)    |
| [Security](#security) | [(Guide)](README.md#security)                                     |
| [Responses](#responses) | [(Guide)](README.md#responses)                                  |
| [JSON](#json) | [(Guide)](README.md#json)                                                 |
| [Artifacts](#artifacts) | [(Guide)](README.md#artifacts)                                  |
| [Performance](#performance) | [(Guide)](README.md#performance)                            |
| [Testing](#testing) | [(Guide)](README.md#testing)                                        |

## Requests
[(Guide)](README.md#requests)

| Recommendation                                 | Examples |
|:--------------------------------------------|:----------------------------------------------------|
| Plural nouns for resource names, HTTP verbs.| :white_check_mark: `GET /inventory_items/4` <br/> :x: `/get_inventory_by_id/4`
| Singular noun for singleton resource        | :white_check_mark: `GET /status`                                       |                                             |
| Shorten associations                        | :white_check_mark: `customers/1/orders?order_id=54&inventory_item=5900` <br/> :x: `customers/1/orders/54/inventory_items/5900`|

| HTTP Verb / Method | Action             |
|:-------------------|:-------------------|
| `GET`              | Read               |
| `POST`             | Create             |
| `PUT`              | Update (full)      |
| `PATCH`            | Update (partial).  |
| `DELETE`           | Delete             |

Identify resources with SQUUIDs, not Table Row IDs.

## Versioning
[(Guide)](README.md#versioning)

`Accept` Header preferred:
```
GET
/inventory_items/4
Accept: application/vnd.livingsocial.v1+json
```

URI accepted for existing services until they can be re-designed to use `Accept` header.
```
GET
/v1/inventory_items/4
```

Scheme is [semver](http://semver.org/) and [ferver](https://github.com/jonathanong/ferver) based.

- Major.Minor only (e.g. `1.4`).
- Never Major version `0`.
- Increment Major: _multiple_ breaking changes or significant change in direction or philosophy of API.
- Increment Minor: _few_ breaking changes.
- Never increment for non-breaking changes.
- Retire versions after clients upgrade past it.

## Pagination
[(Guide)](README.md#pagination)

- Use `offset` and `limit`: e.g. `/inventory_items?offset=543&limit=25`
- Producer determines field `offset` value is based on, `offset` has semantic meaning.
- `limit` has no semantic meaning, simple integer with no relation to resource's data.
- Lead the Consumer:
```
GET /inventory_items?offset=100&limit=25
{
   "_links": {
     "self": { "href": "/inventory_items?offset=543&limit=25" },
     "next": { "href": "/inventory_items?offset=768&limit=25" },
     "prev": { "href": "/inventory_items?offset=123&limit=25" }
   },
   "items": [{ "name": "foo", "desc": "foo to the bar" }, ...]
}
```

## Filter/Sort/Search
[(Guide)](README.md#filtersortsearch)

| Description | Example |
| --- | --- |
| Use simple HTTP params, separate fields for each parameter. | :white_check_mark:&nbsp;`/offers.json?country_id=2&offer_id=9` |
| Do not pass a single param with delimited values. | :x:&nbsp;`/offers.json?filter="country_id::2|offer_id::9"`               |
| Do not use `filter` or `query` in the URI.                  | :x:&nbsp;`/filter_offers.json?country_id=2&offer_id=9`         |
| Do use `sort` param.                                        | :white_check_mark:&nbsp;`/offers.json?sort=city_id`            |

## Content Negotiation
[(Guide)](README.md#content-negotiation)

- Header: `Accept: application/json`
- URI extension: `.../cities/nearby/zipcode/1234.json`
- For `POST` or `PUT`: `Content-Type: application/json`

## Security
[(Guide)](README.md#security)

- Always use HTTPS for public APIs and internal (AWS) APIs. Internal (IAD) adds a performance hit, use HTTP.
- Pass token with `X-LivingSocial-API-Key` header. 
- [CORS is a thing. Read about it.](README.md##cors-cross-origin-resource-sharing)

## Responses
[(Guide)](README.md#responses)

- [HTTP Status Codes](README.md#http-status-codes)
- Internal API: additional error data in JSON response:
```
{ "message": "a thing broke", "exception": "[stacktrace/details]" }
```
- Consumer-Facing API: 
```
{
  "errors": [{
              "code": "unrecoverable_error",
              "title": "The flux capacitor disintegrated",
              "details": "Hold on, the end is nigh.",
              "user_message": {
                "default": "OMG, panic!",
                "en-GB": "Keep a stiff upper lip",
                "de-DE": "Schnell, schnell!!"
              }}]
}
```
- Use `ls-api_validation` to validate responses in test and production.
- Read about [Response Envelopes and Hypermedia](README.md#response-envelopes-and-hypermedia)
- Response Meta Data
```
GET /offers.json?some=search_query
{
  "meta": {
    "representation": "tile",
  },
  "offers": [ ..list of offers... ],
  ...
}
```

## JSON
[(Guide)](README.md#json)

- Date/Time: [RFC 3339](http://tools.ietf.org/html/rfc3339) (e.g. `2008-07-31T22:47:31Z-05:00`)
- [Use UTF-8](http://utf8everywhere.org/): `Content-Type: application/json; charset=utf-8`
- Currency and Fixed-Point:
```
{"currency_code": "USD", "value": 1000, "exponent": 2 } => $10.00
{"currency_code": "JPY", "value": 200, "exponent": 0 } => 200 JPY
{ "tax_rate": {"value": 875, "exponent": 2 } } => 8.75%
```
- Use of XML should be limited to data exchange with the third parties that still use XML.

## Artifacts
[(Guide)](README.md#artifacts)

- [Swagger](http://swagger.io/) all the things on our Swagger Doc Server.
- API should provide executable examples in `curl` or various REST tools: [Chrome](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?hl=en-US) or [Firefox](https://addons.mozilla.org/en-US/firefox/addon/restclient/) plugins, or client GUI ([Paw](https://luckymarmot.com/paw)).

## Performance
[(Guide)](README.md#performance)

[Make it fast](README.md#performance)

## Testing
[(Guide)](README.md#testing)

[Do it](README.md#testing)
