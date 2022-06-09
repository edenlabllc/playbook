
# Edenlab API Manifest

# Interacting with API

Our API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). 
It has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. 
We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. 

## HTTP Verbs

As per RESTful design patterns, API implements following HTTP verbs:

- `HEAD` - Can be issued against any resource to get just the HTTP header info.
- `GET` - Read resources. Should hever change resources on server (idempotency) since most of http proxies and services may reply same request multiple times.
- `POST` - Create new resources.
- `PUT` - Replace resources. Idempotent. (Can be used as `insert or update` method, when we want clients to don't care whenever entity is exists in our system.)
- `PATCH` - Same as `PUT`, but replaces subset of fields instead of full entity. Should never be used as `insert of update` pattern.
- `DELETE` - Remove resources.

## HTTP status codes

HTTP Code | Description
--------- | -----------
`200` | Everything worked as expected.
`201` | Created.
`202` | Accepted. Returned when created a finite-state machine or business process, that is not yet in final state to suggest front-end that it should re-query results and take appropriate actions.
`204` | No Content. Deletion of not-existent resource.
`400` | Bad Request. The request was unacceptable, often due to missing a required parameter. Or request contains invalid JSON. Duplicate idempotency key.
`401` | Unauthorized. No valid API key provided or API key doesn't match project.
`402` | The parameters were valid but the request failed.
`403` | Source or destination account is disabled.
`404` | Not Found. The requested resource doesn't exist.
`415` | Incorrect ```Content-Type``` HTTP header.
`422` | Logical error. For example, POST'ed resource did not pass validation.
`429` | Too Many Requests. Rate limit is exceeded.
`500` `502` `503` `504` | Server Errors. Something went wrong on our end. (These are rare.)

## Field naming

Fields are named in `shake_case` since it's more common for our stack and [20% easier to read](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?tp=&arnumber=5521745).

## Versioning

All API calls should be made with a ```X-API-Version``` header which guarantees that your call is using the correct API version. Version is passed in as a date (UTC) of the implementation in YYYY-MM-DD format.

If no version is passed, the newest will be used and a warning will be shown. Under no circumstances should you always pass in the current date as that will return the current version which might break your implementation.

Optionally API can store `version` field for each of its consumers and allow to stick with same version on all futher request, notifying user that he can switch to another version in APP UI.

Imlementation details can be found in: [Multiverse](https://github.com/Nebo15/multiverse).

## Authentication

In general there are many flows that we use:

1. oAuth
2. JVT
3. HTTP Basic Token

### oAuth Authentification

For oAuth flow client access oAuth server and exchanges it's credentials to a oAuth token.
All requests that require authorization should be made with `Authorization: Bearer :token` header.

#### Scopes Naming

1. Do not create negative scopes (that disallow or turn off some features). Scopes are pure **whitelists**.
2. Avoid using scopes as status or a role model. They are access rights for a single API resource.

You can look at [Facebook oAuth Scopes List](https://developers.facebook.com/docs/facebook-login/permissions), it's a good example of scopes for a big and complex system.

### JVT Authentification (mainly Auth0)

For JVT flow client access authorization server and exchanges it's credentials to a oAuth token.
All requests that require authorization should be made with `Authorization: Token :token` header.

### Token-Based Authentification (for API-only projects)

To use our service you need to authenticate your application. Authentication to the API is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Provide your API key as the basic auth username value. You do not need to provide a password. You can manage your API keys in the Dashboard.

```curl
curl https://example.com/resource \
   -u WgLodNU5wCdbSw4f:
```

## Request structure

Request consist with a root object that contain resource and metadata as it's childs:

**Request example:**

```json
{
  "resource": {
    "name": "Alex",
    "surname": "Gray",
    "email": "me@example.com",
    "credentials: [
      {"method": "passsword", "password": "notasecret"}
    ]
  },
  "format_response": true,
  "redirect_browser": true
}
```

## Response structure

Response can consist of 5 root properties:

- `meta` - URL of the requested resource (`url`); requested response type (`type`, may be either `list` or `object`); HTTP status (`code`); idempotency key (`idempotency_key`); request id (`request_id`).
- `urgent` - Notifications and counters.
- `errors` - Error description. MUST be present when status code is not 2XX, 3XX. CAN NOT be present when `data` is present.
- `data` - Requested resource. MUST be present when status code is 2XX, 3XX. CAN NOT be present when `error` is present.
- `paging` - Pagination data.
- `sandbox` - Optional data provided by `sandbox` environment, for development purposes.

**Response example:**

```json
{
  "meta": {
    "url": "https://qbill.ly/transactions/",
    "type": "list",
    "code": "200",
    "idempotency_key": "iXXekd88DKqo",
    "request_id": "qudk48fFlaP"
  },
  "urgent": {
    "notifications": ["Read new emails!"],
    "unseen_payments": 10
  },
  "data": {
    "type": "resource_name",
    <...>
  },
  "paging": {
    "limit": 50,
    "cursors": {
      "starting_after": "MTAxNTExOTQ1MjAwNzI5NDE=",
      "ending_before": "NDMyNzQyODI3OTQw"
    },
    "has_more": true
  },
  "sandbox": {
    "debug_varibale": "39384",
    "live": "false"
  }
}
```

## Errors

All errors is returned in JSON format if another `Content-Type` is not specified. This means that your will receive JSON for requests with HTTP 415 code when incorrect `Content-Type` is provided.

### Error Object Properties

You can find all necessary information about occurred error in a response `error` property. It can have following fields:

Fields | Optional? | Description
--------- | ----------- | -----------
`type` | no | General error type. You should return human-readable error message based on this field as a key. Type descriptions is listed in next section.
`invalid` | yes | Collection of validation errors for your request.
`invalid[].entry_type` | no | Type of invalid field.
`invalid[].entry` | yes | Name or JSON path to invalid data property.
`invalid[].rules` | yes | Failed rules for invalid property. You can find supported validation rules at [Request Validators](#request-validators) table. Server may optinally skip this field, when validation rules is perefferd to stay secret or hard to extract from validator.
`invalid[].params` | yes | Optional parameters that can be used in a human-readable error message, to make it easier to understand. Usually it contains limit values for failed validator.
`message` | yes | Human readable message for API developer.

**Example error response:**

```json
{
  "error": {
    "type": "validation_failed",
    "invalid": [
      {
        "entry_type": "json_data_proprty",
        "entry": "#/product_type",
        "rules": [
          {
            "rule": "min:6",
            "params": {"min": 6}
          },
          {
            "rule": "length:2",
            "params": {"lenght": 2}
          }
        ]
      },
      {
        "entry_type": "form_data_field",
        "entry": "email",
        "rules": [
          {"rule": "empty"}
        ]
      },
      {
        "entry_type": "header",
        "entry":"Timezone",
        "rules": [
          {"rule": "date"}
        ]
      },
      {
        "entry_type": "request",
        "rules": [
          {"rule": "json"}
        ]
      }
    ],
    "message": "Validation failed. Return human-readable error message. You find all possible validation rules at https://docs.qbill.ly/#request-validators."
  }
}
```

### Error Types

Parameter | Description
--------- | -----------
`idempotency_key_duplicated` | You sent request with duplicate idempotency key, but with different parameters.
`content_type_invalid` | Invalid HTTP request `content-type`.
`validation_failed` | Failed validation on JSON request body or form data. 
`resource_duplicated` | Consumer tried to create entity with same unique constraint.
`token_invalid` | Request requires authorization. Token was received, but it's validation is failed.
`token_expired` | Token was received but it's expired. Retry request with a refreshed token.
`token_not_found` | Request requires authorization, but access token was not present.
`request_too_large` | Request body was to large to process it.

### Error Entry Types

Entry Type | Description
--------- | -----------
`json_data_proprty` | JSON document is invalid. `entry` will contain JSON Path to invalid property.
`form_data_field` | HTTP request field (for `form-data` requests) is invalid. 
`query_param` | HTTP query parameter is invalid.
`body` | Mailformed request body.
`header` | HTTP request header is invalid.

### Validation Rules

List of possible validation rules:

Rule | Params | Description
-------------- | ----------- | -----------
`cast` | `[types]` | The field type should be in `types`.
`required` | `[]` | The field is required and should not be `nil`.
`format` | `[patterns]` | The field should match all regular expressions in `patterns` list.
`inclusion` | `[enum]` | Field value should be in `enum`.
`exclusion` | `[enum]` | Field value should NOT be in `enum`.
`subset` | `[set]` | Field value should be subset of `set`.
`length` | `{min:_, max:_, is:_, exclusive:true|false}` | List or string length should be equal to `is` value, less than `max` and bigger than `min` value. Params is optional, but one of them should always be present. `exclusive` is false by default.
`number` | `{greater_than:_, less_than:_, greater_than_or_equal_to:_, less_than_or_equal_to:_, multiple_of:_}` | The field value must be a number that is greater than `greater_than` and less than `less_than`.
`confirmation` | `[]` | The field value of field must match confirmed field (eg. `password_confirmation` should match `password`).
`acceptance` | `[]` | The field value should be true.
`email` | `[patterns]` | The field value must be a valid email address respectively to `patterns`.
`phone_number` | `[patterns]` | The field value must be a valid phone number respectively to `patterns`. *Usually it means that is should be in international format (with `+3`).*
`card_number` | `[]` | The field value should be a valid Visa or MasterCard card number. (By luhn algoryghm.)
`metadata` | `[]` | The field must be a [valid metadata object](http://docs.apimanifest.apiary.io/#introduction/optional-features/metadata).
`unique` | `[]` | The field must be a list with all it's items unique.
`schemata` | `[]` | JSON Schema specific error core.
`schema` | `[]` | JSON Schema specific error core.
`dependency` | `[json_paths]` | Property expects another property to exist.
`json` | `[]` | The field under validation must be a valid JSON string.
`authentification_code` | `{resent: true}` | OTP or other authentification code was invalid. User should retry with re-entering it. `resent` property will tel is code was automatically resent upon failed validation.
`date`, `datetime`, `timestamp` | `[after:_, before:_, after_or_at:_, before_or_at:_]` | Invalid datetime representation.
`url` | `[patterns]` | Field should contain a valid URL.

## Pagination

All top-level API resources with root type `list` have support of pagination over a "list" API methods. These methods share a common structure, taking at least these three parameters: `limit`, `starting_after`, and `ending_before`.

### Cursor-Based

Recommended way is to provide a cursor-based pagination via the `starting_after` and `ending_before` parameters. Both take an existing object ID value (see below). The ```ending_before``` parameter returns objects created before the named object, in descending chronological order. The `starting_after` parameter returns objects created after the named object, in ascending chronological order. If both parameters are provided, only ending_before is used.

Arguments:

- `limit` (optional) - A limit on the number of objects to be returned, between 1 and 100. Default: 50;
- `starting_after` (optional) - A cursor for use in pagination. `starting_after` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with ```obj_foo```, your subsequent call can include `starting_after=obj_foo` in order to fetch the next page of the list;
- `ending_before` (optional) - A cursor for use in pagination. `ending_before` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with ```obj_bar```, your subsequent call can include `ending_before=obj_bar` in order to fetch the previous page of the list.

### Offset-Based Pagination

Sometimes you might want to give your users ability to traverse trough pages, in this case offset-based pagination MAY be available.

Arguments:

- `limit` (optional) - A limit on the number of objects to be returned, between 1 and 100. Default: 50;
- `offset` (optional) - This offsets the start of each page by the number specified.

_This method is not resommended, whenever you users want to traverse history it's better to give them ability to query data and apply cursor based pagination on it's results._

## Content Type

You should send `Content-Type` header in all your requests, otherwise API will return HTTP 415 status. Right now we support two content types:

- `application/json` - Response is sent in a JSON format.
- `text/csv` - Response is trimmed to a requested resource and sent in a CSV format.
- `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` - Response is trimmed to a requested resource and sent in a XLSX format.

This header doesn't affect any outgoing requests, they are always sent in JSON format.

> To make CSV look better in spreadsheets viewers we are stripping all response metadata (everything except data that is returned in `data` field). All embed collections will be embedded with root property name prefix, all collections will be flattened to a single line divided by `|`.

## Timezones

All requests allow to provide a `Time-Zone` header to receive all date and time fields in a local timezone.

Explicitly provide an ISO 8601 timestamp with timezone information to use this feature. Also it is possible to supply a Time-Zone header which defines a timezone according to the list of names from the [Olson database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

```shell
curl -H "Time-Zone: Europe/Amsterdam" ..
curl -H "Time-Zone: Europe/Kiev" ..
```

*When time zone is not specified, response will be sent in UTC time zone.*

## Request ID

Each API request has an associated request identifier. You can find it in ```X-Request-ID``` response header or inside ```meta.request_id``` property of returned JSON.

You can also find request identifiers in the URLs of individual request logs in your Dashboard. If you need to contact us about a specific request, providing the request identifier will ensure the fastest possible resolution.

All Request ID's are prefixed with a human-readable name of server that served your request.

```
X-Request-ID: flash-99spDSoim3i
```

## HTTP Redirects

API uses HTTP redirection where appropriate. Clients should assume that any request may result in a redirection. Receiving an HTTP redirection is not an error and clients should follow that redirect. Redirect responses will have a Location header field which contains the URI of the resource to which the client should repeat the requests.

Status codes:

- `301` - Permanent redirection. The URI you used to make the request has been superseded by the one specified in the Location header field. This and all future requests to this resource should be directed to the new URI.
- `302`, `307` - Temporary redirection. The request should be repeated verbatim to the URI specified in the Location header field but clients should continue to use the original URI for future requests.

Other redirection status codes may be used in accordance with the HTTP 1.1 spec.

## File Uploads

### For files less than 10 MB

For small files it's appropriate to send base64 encoded file contents as JSON field:

{
    name: "My Name",
    image: "sdasdpjoijhwhwe83280u2380wef=="
}

### For larger files - Multi-Stage Uploads (Network Transactions)

Inspired by Twitter API, Youtube API and [github issue](https://github.com/yahoo/elide/issues/203). The idea is to sort of transactionize the process of file upload using in multiple HTTP calls. The procedure goes as follows:

1. The client sends an initial JSON-API HTTP request, to provide the metadata of the files to be upload as well as a `X-Upload-Content-Length` header field to indicate the total accounted size of the content.
2. The server receives the initial call, processes the metadata(save to database, add to job queue, etc.), marks the metadata record as unfinished, assigns a transaction id, comes up with a timeout, and responds to client with a `Location` header which contains a tailored URL to the file upload endpoint
3. The client receives the response, and sends the file to the file upload endpoint.
4. The server receives the file upload request and processes it. Response will be the same as the response to the initial call.
5. The client sends an finialize request, to mark that file upload is complete.
6. The server receives the finalize request, and close the network transaction
7. If the upload processes still haven't finished after the timeout, the server close the network transaction, marks the metadata record as aborted. Responds to further upload requests with reject messages.

# Optional Features

## Resource Actions

Whenever you want to implement a business process or a Finite-State Machine that is controlled trough an API you must start from listing an [afforances](https://en.wikipedia.org/wiki/Affordance) that is available for current resource (client may decide which actionable items will be available for front-end consumer based on them):

```json
{
  "data": {
    "id": 1,
    "author": "Drew",
    "message": "Hola, como estas?"
    "afforances": [
      {"action": "approve", "url": "https://example.com/resources/1/actions/approve"},
      {"action": "decline", "url": "https://example.com/resources/1/actions/decline"}
    ]
  }
}
```

Thus API consumer MAY make request to an action url to initiate a state transition.

API design in this case MUST have a [FSM description](http://madebyevan.com/fsm/).

> Before adding an actionable transitions it's better to thing twice how can be this done in other methods, since they make API much more complicated for consumers and developers.

References:
- [HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)
- [Resource Blueprint](https://github.com/resource-blueprint/resource-blueprint#def-syntax)

## Updating relations with parent resource

Imagine a situation when you have resource that returns number of embed resources in its structure:

```
{
    "id": "post_id",
    "title": "My Blog Post",
    "description": "Long content",
    "authors": [
        {
            "id": 1,
            "name": "Andrew"
        },
        {
            "id": 2,
            "name": "Max"
        }
    ]
}
```

Making a PUT request to this resource MAY expect you to pass whole `authors` collection, and it `authors` property is present in root object your API should:

1. Create new records for each resource in embed list that doesn't have an `id` or `id` was not found.
2. Update all embed resources that match by `id`.
3. Delete all embed resources that consumer did not sent.

This approach makes easier for browser applications to think about resources that looks simple in API, but have complex relations in your application logic.

## Rate Limits (Throttling)

We throttle our APIs by default to ensure maximum performance for all developers.

Rate limiting of the API is primarily considered on a per-consumer basis. All your projects share a same rate limit, to avoid API-consuming fraud. Rate limits depend on your account type.

Currently free accounts is rate limited to 1000 API calls every 15 minutes, but this value may be adjusted at our discretion.

For your convenience, all requests is sent with 3 additional headers:

HTTP Header | Description
------------- | -----------
`X-Rate-Limit` | Current rate limit for your application.
`X-Rate-Limit-Remaining` | Remaining rate limit for your application.
`X-Rate-Limit-Reset` | The time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time).

> When limit is exceeded all requests will return `HTTP 429` status code with corresponding error in ```meta`` response object.

```
X-Rate-Limit: 5000
X-Rate-Limit-Remaining: 4966
X-Rate-Limit-Reset: 1372700873
```

## Cross Origin Resource Sharing

If API supports Cross Origin Resource Sharing (CORS) for AJAX requests from any origin. 
Read the [CORS W3C Recommendation](http://www.w3.org/TR/cors/), or [this intro](http://code.google.com/p/html5security/wiki/CrossOriginRequestSecurity) from the HTML 5 Security Guide.

<aside type="notice">
Never publish your application or project tokens in any kind of Front-End applications. They carry all the project privileges, and would be exposed to a third-parties. You can find more info about client authentication in [Adding oAuth provider](#adding-oauth-provider) section.
</aside>

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-Request-ID, X-Idempotency-Key
Access-Control-Allow-Credentials: true
```

## Token, ID lengths and formats

In order to avoid interruptions in processing, it's best to make minimal assumptions about what our gateway-generated tokens and identifiers will look like in the future. The length and format of these identifiers – including payment method tokens and transaction IDs – can change at any time, with or without advance notice. However, it is safe to assume that they will remain 1 to 64 upper- and lowercase alphanumeric characters with minuses (```-```) and underscores (```_```).

We use ISO 8601 formats for all dates in our API.

We use E.123 telephone number in international notation for all phone numbers in our API.

All tokens have a human-readable prefix, so you can always see what scope it carries. Example: `project-lksdlkfjf8ds8dfsl`.

```
user-Skd90i0d
project-dkkfi49dkkf
admin-3kkd9re0fdkspmv
```

All ID's have a human-readable prefix that carries first 3 characters from a entity name. Example: ```acc_ssj8988udj```.

## Idempotent Requests

The API supports idempotency for safely retrying write requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.

To perform an idempotent request, attach a unique key to any `POST`, `PATCH` or `DELETE` request made to the API via the `Idempotency-Key: <key>` header.

How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters we will return `HTTP 400` error. The keys expire after 24 hours.

> If you have an "Account" entity, within your system, one of best ways to generate right Idempotency key is to add a additional property `nonce` to a Account, and re-generate it every time transaction is created. Thus you will be sure that there no way to create a transaction without knowing latest state of the user account.

## Limiting Response Fields

By default, all the fields in a node are returned when you make a query. You can choose the fields you want returned with the ```fields``` query parameter. This is really useful for making your API calls more efficient and fast.

```curl
/resource?fields=id,name,balance
```

## Expanding Response Fields into Objects

Many response fields contain the ID of a related object in their values. For example a ```Transfer``` can have an associated ```User``` object linked by a ```sender``` field. Those objects can be expanded inline with the ```expand``` request parameter. Objects that can be expanded are noted in this documentation. This parameter is available on all API requests, and applies to the response of that request only.

You can expand object by querying list, expands will be applied on all matched elements. Expanded lists return up to 25 elements, limit can be specified in brackets.

You can expand multiple objects at once by identifying multiple items divided by comma.

```curl
/accounts?expand=transactions(5)
```

You can nest expand requests with the dot property. For example, requesting ```sender.payments``` on a ```Transfer``` list will expand the ```sender``` property into a full ```User``` object, and will then expand the ```payments``` property of that ```User``` into a full ```Transactions``` collection.

```curl
/accounts?expand=transactions(5).recipients
```

## Ordering Lists and Collections

By default, all collections are ordered in ascending chronological order. You can specify different order by providing the "order" query parameter.

```curl
/accounts?order=account.created_at(reverse_chronological)
```

Available orders:

- `ascending_chronological` - Ascending chronological order.
- `reverse_chronological` - Descending chronological order.

## Filtering Lists

You can filter all lists by a data. Filter can refer to any fields inside a ```data``` response property, except entities that was expanded. Also you can filter by a metadata value.

To apply filter you should provide [Base64 encoded](https://en.wikipedia.org/wiki/Base64) encoded JSON string with a filtering rules in a ```filter``` query parameter.

> Create a JSON object and convert it to string:

```bash
$ echo "{predicates:[<predicate_1>,<predicate_1>,...]}" | base64
e3ByZWRpY2F0ZXM6WzxwcmVkaWNhdGUxPiw8cHJlZGljYXRlMT4sLi4uXX0K
```

> Send encoded string as filter query parameter:

```
GET /v1/accounts?filter=e3ByZWRpY2F0ZXM6WzxwcmVkaWNhdGUxPiw8cHJlZGljYXRlMT4sLi4uXX0K
```

Predicate is an object with at least 3 fields:

- `field` - Resource filed that should be used for current filter predicate.
- `comparison` - Comparison type that is applied to a field value and ```value``` predicate property.
- `value` - Value that should be compared with resource field value.

Available comparison methods:

Name | Description
------------- | -----------
eq | Matches values that are equal to a specified value.
ne | Matches all values that are not equal to a specified value.
gt | Matches values that are greater than a specified value.
gte |  Matches values that are greater than or equal to a specified value.
lt | Matches values that are less than a specified value.
lte |  Matches values that are less than or equal to a specified value.
in | Matches any of the values specified in an array. Array is set to a ```value``` predicate property.
nin |  Matches none of the values specified in an array.
ewi | Matches all values that ends with a specified value.
swi | Matches all values that starts with a specified value.

```json
{
  predicates:[
    {
      "attribute":"balance",
      "comparison":"eq",
      "value":"27"
    },
    {
      "attribute":"metadata.user_id",
      "comparison":"in",
      "value":[1, 2, 3]
    }
  ]
}
```

To apply filters with logical rules you can add with logical type. Available types:

Name | Description
------------- | -----------
or | Joins query clauses with a logical OR returns all objects that match the conditions of either clause.
and | Joins query clauses with a logical AND returns all objects that match the conditions of both clauses.
not | Inverts the effect of a query expression and returns objects that do not match the query expression.
nor | Joins query clauses with a logical NOR returns all objects that fail to match both clauses.

```json
{
  "predicates":[
    {
      "attribute":"metadata.user_id",
      "comparison":"eq",
      "value":"3994kdkd8"
    },
    {
      "type":"or",
      "predicates":[
        {
          "attribute":"balance",
          "comparison":"gt",
          "value":"100"
        },
        {
          "attribute":"transactions.count",
          "comparison":"gt",
          "value":"10"
        }
      ]
    }
  ]
}
```

## Aggregating lists

All lists can be queried to get aggregations on given set of rules. When you are using aggregation default filtered period is one month. For example you can get aggregated count of ```Accounts``` to know how many accounts was created at each day for last month.

To use aggregation you need to specify only one query parameter:

- ```aggregate``` - [Base64 encoded](https://en.wikipedia.org/wiki/Base64) JSON string with a aggregation rules.

Also you can change aggregation dispensation by providing ```tick``` query parameter:

- ```tick``` - Aggregation period particle. Default value: ```day```. Optional.

> Create a JSON object and convert it to string:

```bash
$ echo "{aggregate:[<aggregate_1>,<aggregate_1>,...]}" | base64
e2FnZ3JlZ2F0ZXM6WzxhZ2dyZWdhdGVfMT4sPGFnZ3JlZ2F0ZV8xPiwuLi5dfQo=
```

> Send encoded string as filter query parameter:

```
GET /v1/accounts?aggregate=e3ByZWRpY2F0ZXM6WzxwcmVkaWNhdGUxPiw8cHJlZGljYXRlMT4sLi4uXX0K&tick=day
```

Aggregate is an object with at least 2 fields:

- ```name``` - Property name in a response aggregate object.
- ```strategy``` - Aggregation strategy.
- ```field``` - Field that will be used for aggregation.

```json
{
  predicates:[
    {
      "name": "accounts_count",
      "strategy": "count",
      "field": "id"
    },
    {
      "name": "accounts_liquidity",
      "strategy": "sum",
      "field": "balance"
    }
  ]
}
```

**Response:**

```json
{
  meta: {
    "type": "list"
  }
  data: [
    {
     "tick":"2015-07-11",
     "aggregates": {
       "accounts_count": 123,
       "accounts_equity": 1000000
     }
    },
    {
     "tick":"2015-07-12",
     "aggregates": {
       "accounts_count": 98,
       "accounts_equity": 900000
     },
     ...
    }
  ]
}
```

Available aggregation strategies:

Name | Description
------------- | -----------
count | Returns count of returned aggregated fields.
sum | For integers and floats returns total for all field values.
arr | Returns array of all possible values. (Very useful for anti-frauds.)
max | Returns maximum value of a field.
min | Returns minimum value of a field.
avg | Returns average value of a field.

Available ```tick``` values:

Name | Description
------------- | -----------
hour | Return aggregation for each hour in a filtered period.
day | Return aggregation for each day in a filtered period. Default value.
month | Return aggregation for each month in a filtered period.
year | Return aggregation for each year in a filtered period.

## Testing

All accounts have test project that is created for you on account creation. Just use test project API secret for your test environment. We don't have any policy for data retention in test accounts and we can drop all data once a while. You can do it manually from your dashboard.

Responses in ```sandbox``` environment will always return a ```sandbox``` property.

```
{
  "meta":{},
  "data":{},
  "sandbox":{
    ...
  }
}
```

Right now test accounts is not rate limited, but we will manually limit test project that will consume too many resources.

## Metadata

We support ```metadata``` field for every objects in our API. It allows the API developer to store custom information related to accounts and transactions. This information can be for example:

- Internal order ID.
- Customers name and email address.

Metadata field supports key-value pairs with the following limitations:

- Up to 24 keys.
- Up to 100 characters for the key (alphanumeric characters, hyphens and underscores).
- Up to 500 characters for the value.
- String, integer, decimals, lists and boolean values only. All other types will be converted into string.

For lists:

- Up to 25 elements.
- Up to 100 characters in element.

## Batching Requests

There are number of situations when you need to download few entities at once, for example when your customers have multiple accounts and you need to return balance for all of them in a single request. For this cases we have request multiplexing.

The batch API takes in an array of logical HTTP requests represented as JSON arrays - each request has a method (corresponding to HTTP method GET/PUT/POST/DELETE etc.), a uri (the portion of the URL after domain), optional headers array (corresponding to HTTP headers) and an optional body (for POST and PUT requests). The Batch API returns an array of logical HTTP responses represented as JSON arrays - each response has a status code, an optional headers array and an optional body (which is a JSON encoded string).

To make batched requests, you build a JSON object which describes each individual operation you'd like to perform and POST this to the batch API endpoint at https://batcher.qbill.co.

```
curl \
    -F ‘batch=[{“method”:”GET",“relative_url”:/accounts/:id1”},{“method”:”GET",“relative_url”:/accounts/:id1”}]’ \
    https://graph.facebook.com
```

As an alternative you can use simpler alias for a batching requests to the objects. Whenever you can provide an ID inside URL to retrieve resource, you can also provide multiple ID's separated by comma. For example, you can get two user accounts in one request.

```
GET /projects/:project_id/accounts/:id1,id2,...
```

> Response would be a list of requested items.

```
{
  data: {
    "<user1_id>": {"<user1>"},
    "<user2_id>": {"<user2>"}
  }
}

```

### Timeouts

Large or complex batches may timeout if it takes too long to complete all the requests within the batch. In such a circumstance, the result is a partially-completed batch. In partially-completed batches, responses from operations that complete successfully will look normal whereas responses for operations that are not completed will have corresponding error code in ```meta``` property.

### Limits

You can batch up to 10 request. Every request in batch will be counted as separate requests in Rate Limits.

## Geographic Redundancy and Optimization

To ensure that you will always have the lowest response time we can provide, we are automatically detecting nearest datacenter to you, so all your projects have master servers in it. To migrate data to a different region please contact our support team.

## Data Storage and Backup Policy

To ensure that you won't loose your data we use geographical redundant MongoDB replica sets. It means that at least one of your secondary DB's is hosted in another region, and will save all data in case main datacenter would be unavailable.

## Providing urgent data for your users

Sometimes you want to update account balance and notifications list on each request you made. You can provide additional HTTP header ```X-Urgent-Account-ID``` with an Account ID that should be queried. All result data will be in ```urgent``` response field.

Requesting urgent data counts as a separate request and affects your rate limits.

```
X-Urgent-Data-ID: acc_3idjdjkd9
```

```
{
  "meta": {
  },
  "urgent": {
    "account": "acc_3idjdjkd9",
    "notifications": [],
    "unseen_payments": 0,
    "holds": 0,
    "balance": 0
  },
  "data": {
  }
}
```

## Conditional requests

Most responses return an ETag header. Many responses also return a Last-Modified header. You can use the values of these headers to make subsequent requests to those resources using the ```If-None-Match``` and ```If-Modified-Since``` headers, respectively. If the resource has not changed, the server will return a 304 Not Modified.

Also note: making a conditional request and receiving a 304 response does not count against your Rate Limit, so we encourage you to use it whenever possible.

## SSL certificates

We recommend that all consumers obtain an SSL certificate and serve any data that is stored in our service over HTTPS. Also we don't allow to add a webhook that doesn't support SSL encryption.

We don't have a specific preferred vendor for SSL certificates, but we recommend that you stick with a well-known provider (e.g. Network Solutions, GoDaddy, Namecheap). Generally speaking, most certificates will be similar, so it's up to you to determine what fits your needs best.

You can test your server with a [SSL Server Test](https://www.ssllabs.com/ssltest/).

# Database Fields Naming

We think about our data as tree structures, because there are almost no application where data can be flat.

This way of thinking requires naming where we need to imagine how our data whould look if everything is an object, and flatten values by `_` separator when we don't want to add whole object (which may be overkill for a few fields). At the moment we don't recommend to add objects when they have less than 5 fields.

Example:

```
{
  "tick":"2015-07-12",
  "aggregates": {
    "accounts_count": 98,
    "accounts_equity": 900000
  }
}
```

This data may be represented in different ways, `as-is` in example or flatten:

```
"tick":"2015-07-12",
"aggregates_accounts_count": 98,
"aggregates_accounts_equity": 900000
```

This way of thinking gives additional values:

- It is simples to visually see related objects and developers may group them togather.
- It's easier to notice when object gets more fields and when it's better to crate separate resource for it.

### Boolean Fields

Should have `is_` prefix, which allows to understand type of field by it's name.

### Date/Time Fields

May continue with `_at` suffix.

### Key Value

K-V objects are stored in a JSONB (or similar) structures. No K-V tables are needed.

# Apiary Data Structure Helpers

```
## Data Structures
## Responses
### `Response_Collection`
+ meta (Response__Meta, fixed-type)
+ data (array[], fixed-type)
+ paging (Response__Pagination_Cursor, fixed-type)

### `Response_OK`
+ meta (Response__Meta, fixed-type)
+ data (object, fixed-type)

### `Response_Error`
+ meta (Response__Meta, fixed-type)
    + code: 400 (number)
+ error (Response__Error, fixed-type)

### `Response__Meta`
+ code: 200 (number) - HTTP response code.
+ url: http://example.com/resource (string) - URL to requested resource.
+ type (enum) - Type of data that is located in `data` attribute.
    - object (string) - `data` attribute is a JSON object.
    - list (string) - `data` attribute is a list.
+ `idempotency_key`: `idemp-ssjssdjoa8308u0us0` (string, optional) - [Idempotency key](http://docs.apimanifest.apiary.io/#introduction/optional-features/idempotent-requests). Send it trough `X-Idempotency-Key` header.
+ `request_id`: `req-adasdoijasdojsda` (string) - [Request ID](http://docs.apimanifest.apiary.io/#introduction/interacting-with-api/request-id). Send it with `X-Request-ID` header.

### `Response__Error`
+ type: type_atom (string) - Atom that represents error type.
+ message: Error description (string) - Human-readable error message. This is for developers, not end-users.

### `Response__Error_DuplicateEntity`
+ type: `object_already_exists` (string) - Atom that represents error type.
+ message: This API already exists (string) - Human-readable error message. This is for developers, not end-users.

### `Response__Error_ValidationFailed`
+ type: validation_failed (string) - type of an error.
+ message: Validation failed. You can find validators description at our API Manifest: http://docs.apimanifest.apiary.io/#introduction/interacting-with-api/errors. (string)
+ invalid (array)
  + `entry_type`: `json_data_proprty` (string) - Type of error.
  + entry: $.cvv (string) - JSON Path to an invalid property.
  + rules (array)
    + rule: required (string) - String constant that represents validation rule type. List of all types can be found in [API Manifest](http://docs.apimanifest.apiary.io/#introduction/interacting-with-api/errors).
    + params (array) - Validation Parameters.

### `Response__Pagination_Cursor`
+ limit: 20 (number) - A limit on the number of objects to be returned, between 1 and 100. Default: 50.
+ size: 1000 (number) - Total number of objects in collection.
+ has_more: true
+ cursors (object)
    + `starting_after`: 56c31536a60ad644060041af (string) - A cursor for use in pagination. An object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with `obj_foo`, your subsequent call can include `starting_after=obj_foo` in order to fetch the next page of the list.
    + `ending_before`: 56c31536a60ad644060041aa (string) - A cursor for use in pagination. An object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with `obj_bar`, your subsequent call can include `ending_before=obj_bar` in order to fetch the previous page of the list.

### `Response__Pagination_Offset`
+ limit: 20 (number) - A limit on the number of objects to be returned, between 1 and 100. Default: 50.
+ size: 1000 (number) - Total number of objects in collection.
+ offset: 10 (number) - Offset of first item on current page.
