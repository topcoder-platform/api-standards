# Topcoder API Standards

### Guidelines


#### API Endpoints

An "endpoint" is a combination of two things:

* The verb (e.g. `GET` or `POST`)
* The URL path (e.g. `/articles`)

Generally speaking:

* **Avoid single-endpoint APIs.** Don't jam multiple operations into the same endpoint with the same HTTP verb.
* **Prioritize simplicity.** It should be easy to guess what an endpoint does by looking at the URL and HTTP verb, without needing to see a query string.

#### RESTful URLs
* A URL identifies a resource.
* URLs should include nouns, not verbs.
* Use **plural nouns** only for consistency (no singular nouns).
* Use HTTP verbs (GET, POST, PUT, DELETE) to operate on the collections and elements.
* You should not need to go deeper than resource/identifier/resource.
* Put the version number at the base of your URL, for example http://example.com/v1/path/to/resource.

#### HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.
The action taken on the representation will be contextual to the media type being worked on and its current state. Here's an example of how HTTP verbs map to create, read, update, delete operations in a particular context:

| HTTP METHOD | POST            | GET       | PUT         | PATCH | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE      | UPDATE (partial) | DELETE |
| /dogs       | Create new dogs | List dogs | Bulk update | Bulk update | Delete all dogs |
| /dogs/1234  | Error           | Show Bo   | If exists, update Bo; If not, error | If exists, update Bo; If not, error | Delete Bo |

#### Just use JSON

[JSON](https://en.wikipedia.org/wiki/JSON) is an excellent, widely supported transport format, suitable for many web APIs.

Supporting JSON and only JSON is a practical default for APIs, and generally reduces complexity for both the API provider and consumer.

General JSON guidelines:

* Responses should be **a JSON object** (not an array). Using an array to return results limits the ability to include metadata about results, and limits the API's ability to add additional top-level keys in the future.
* **Don't use unpredictable keys**. Parsing a JSON response where keys are unpredictable (e.g. derived from data) is difficult, and adds friction for clients.
* **Use consistent case for keys**. We're using **camelCase** for all keys.

#### Versioning
* Never release an API without a version number.
* Versions should be integers, not decimal numbers, prefixed with ‘v’. For example:
  * Good: v1, v2, v3
  * Bad: v-1.1, v1.2, 1.3
* Maintain APIs at least one version back.


#### Use a consistent date format

And specifically, [use ISO 8601](https://xkcd.com/1179/), in UTC.

For just dates, that looks like `2013-02-27`. For full times, that's of the form `2013-02-27T10:00:00Z`.

This date format is used all over the web, and puts each field in consistent order -- from least granular to most granular.

#### Use ISO Constants where applicable
* **Country Codes** - [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)
* **Currency Codes** - [ISO 4217  alpha-3](https://en.wikipedia.org/wiki/ISO_4217)


#### Default fields to be included
All resources should include the following default fields:
* createdBy - UserId of who created the record.
* createdAt - Timestamp of when the record was created.
* updatedBy - UserId of who updated the record.
* updatedAt - Timestamp of when the record was updated.

#### Error handling

Handle all errors (including otherwise uncaught exceptions) and return a data structure in the same format as the rest of the API.

For example, a JSON API might provide the following when an uncaught exception occurs:

```json
{
  "message": "Description of the error.",
  "exception": "[detailed stacktrace]"
}
```

HTTP responses with error details should use a `4XX` status code to indicate a client-side failure (such as invalid authorization, or an invalid parameter), and a `5XX` status code to indicate server-side failure (such as an uncaught exception).

#### Security
Use HTTP 'Authentication' header to pass [JWT](https://jwt.io/introduction/) eg:
`Authentication: Bearer {topcoder JWT}`

## Record limits


#### Support ETags for Caching
Include an ETag header in all responses, identifying the specific version of the returned resource. This allows users to cache resources and use requests with this value in the If-None-Match header to determine if the cache should be updated.

#### Provide Request-Ids for Introspection
Include a Request-Id header in each API response, populated with a UUID value. By logging these values on the client, server and any backing services, it provides a mechanism to trace, diagnose and debug requests.

### Use UTF-8

Just [use UTF-8](http://utf8everywhere.org).

Expect accented characters or "smart quotes" in API output, even if they're not expected.

An API should tell clients to expect UTF-8 by including a charset notation in the `Content-Type` header for responses.

An API that returns JSON should use:

```
Content-Type: application/json; charset=utf-8
```

### CORS

For clients to be able to use an API from inside web browsers, the API must [enable CORS](http://enable-cors.org).

For the simplest and most common use case, where the entire API should be accessible from inside the browser, enabling CORS is as simple as including this HTTP header in all responses:

```
Access-Control-Allow-Origin: *
```

It's supported by [every modern browser](http://enable-cors.org/client.html), and will Just Work in many JavaScript clients, like [jQuery](https://jquery.com).

For more advanced configuration, see the [W3C spec](http://www.w3.org/TR/cors/) or [Mozilla's guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).

### Requests
#### Filtering
`?filter=URLENCODE({fieldName1}={fieldValue1}&...{fieldNameN}={fieldValueN})`
* filedValue can contain multiple values using in() format
`{fieldName}=in({fieldValue1},{fieldValue1})`


#### Field Selection
Mobile clients display just a few attributes in a list. They don’t need all attributes of a resource. Give the API consumer the ability to choose returned fields. This will also reduce the network traffic and speed up the usage of the API.

`GET /cars?fields=manufacturer,model,id,color`


#### Paging / Limiting
Use limit and offset. It is flexible for the user and common in leading databases. The default should be `limit=20` and `offset=0`

To get records 51 through 75 do this:
```
http://example.gov/magazines?limit=25&offset=50
```
* offset=50 means, ‘skip the first 50 records’
* limit=25 means, ‘return a maximum of 25 records’

Information about record limits and total available count should also be included in the response. Example:
``` json
  {
    "metadata": {
      "count": 227,
      "offset": 25,
      "limit": 25
    },
    "results": []
}
```

#### Sort
Support `orderBy` field name to sort


#### POST / PUT / PATCH
Use "param" as parameter key in request body
Example of creating new environment:
```
URL
POST https://api.tc.appirio.net/v3/environments
Body
{
   "param" : {"name":"some name", "platform":"salesforce", .....},
   “options” : {“notify”:”true”}			-- (optional) additional parameters that do NOT belong to resource attributes
   "method" : "post",				-- (optional) overrides method if specified
   "return" : "(id,name,dummy)",		-- (optional) if not specified returns entire resource
   "debug" : true/false				-- (optional)
}
```

### Response

#### Format
Response is in JSON, always has response "id" and “result”
```
{
  "id": "<generated response id>",
  "result": {
     "success": true|false,
     "status": 200,
     "metadata": {<metadata_of_response>}
     "content": {<content_of_response>}
   }
}
```

Error response:
```
{
  id: <backend_created_response_id>,
  result: {
     success: false,
     status: 400,
     content: {<error_message>}
     debug: {
         detail: <detailed_message>
         stackTrace: <stack_trace>
      }
   }
}
```
