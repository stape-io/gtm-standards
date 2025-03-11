# GTMS-7
## Server Templates BigQuery Logging

### Problem that is solved by this standard

GTM Server-Side does not have a common format for writing logs to `BigQuery`. This standard ensures structured logging, enabling easier querying, debugging, and monitoring.

### Standard description

- Each log line is a row in BigQuery. Each insertion to the BigQuery table must add only one row at a time.
- Each log line must contain at least: `timestamp`, `type`, `tag_name` and `trace_id` fields.
- Field `timestamp` must contain the Unix epoch timestamp in milliseconds.
- Field `type` must contain one of these values `Request`, `Response`, `Request-Response`, `Message`. Which helps easily filter logs.
- `trace_id` must contain a unique identifier of the current incoming request. This field is used for stitching all logs done by all tags/clients during one request. It's highly recommended using the `trace-id` header for this, as many clouds/platforms use it for tracing. 
- Field `tag_name` must contain the tag name.
- All fields must use snake case naming, e.g. `trace_id`, `response_code`etc. and not `TraceId`, `ResponseCode` etc.
- `ignoreUnknownValues` must be set to `true` to handle schema changes gracefully. This allows new fields to be added without breaking existing logs.


#### Recommended additional fields
Based on the log type, the following additional fields are recommended:

##### For type `Request`

- `request_method`: HTTP method (GET, POST, etc.).
- `request_url`: Full URL of the request.
- `request_body`: String representation of the request body.
- `event_name`: Name of the event being tracked.

##### For type `Response`

- `response_status_code`: HTTP status code.
- `response_body`: String representation of the response body.
- `response_headers`: String representation of response headers.
- `event_name`: Name of the event being tracked.

##### For type `Request-Response`
Combine the fields from both `Request` and `Response` types.

##### For type `Message`

- `message`: Information or error message.
- `reason`: Explanation of why the message was logged.
- `event_name`: Name of the event being tracked (if applicable).


### Required BigQuery Schema

| Field Name       | Type     | Mode     | Description |
|-----------------|-----------|----------|-------------|
| `timestamp`     | TIMESTAMP    | Required | Unix epoch timestamp (milliseconds). |
| `type`          | STRING   | Required | Log type: `Request`, `Response`, `Request-Response`, or `Message`. |
| `trace_id`      | STRING   | Required | Unique identifier for the request (from the `trace-id` header). |
| `tag_name`      | STRING   | Required | Name of the GTM tag generating the log. |


### Expanded BigQuery Schema (suggestion)

| Field Name       | Type     | Mode     | Description |
|-----------------|-----------|----------|-------------|
| `timestamp`     | TIMESTAMP    | Required | Unix epoch timestamp (milliseconds). |
| `type`          | STRING   | Required | Log type: `Request`, `Response`, `Request-Response`, or `Message`. |
| `trace_id`      | STRING   | Required | Unique identifier for the request (from the `trace-id` header). |
| `tag_name`      | STRING   | Required | Name of the GTM tag generating the log. |
| `request_method`   | STRING   | Nullable | HTTP method (GET, POST, etc.) (if applicable). |
| `request_url`   | STRING   | Nullable |  URL of the API request (if applicable). |
| `request_body`  | STRING   | Nullable |  JSON stringified request body (if applicable). |
| `response_status_code` | INTEGER    | Nullable |  HTTP status code from API response (if applicable). |
| `response_body` | STRING   | Nullable |  JSON stringified API response body (if applicable). |
| `response_headers` | STRING   | Nullable |  JSON stringified API response headers (if applicable). |
| `message`       | STRING   | Nullable |  Additional log messages (for `Message` type). |
| `reason`        | STRING   | Nullable |  Reason for failure (for `Message` type). |

### Example Code

```js
const BigQuery = require('BigQuery');
const getContainerVersion = require('getContainerVersion');
const getRequestHeader = require('getRequestHeader');
const isLoggingEnabled = determinateIsLoggingEnabled();
const traceId = isLoggingEnabled ? getRequestHeader('trace-id') : undefined;

const postUrl = 'https://example.com';
const postBody = { 'data': 'some data' };

if (validateMissingRequiredFields()) {
  log({
    'tag_name': 'Example',
    'type': 'Message',
    'trace_id': traceId,
    'timestamp': getTimestampMillis(),
    'event_name': 'purchase',
    'message': 'Request won't be sent.',
    'reason': 'One or more fields are missing: Email, Phone Number or External ID.',
  });
  return data.gtmOnFailure();
}

log({
  'tag_name': 'Example',
  'type': 'Request',
  'trace_id': traceId,
  'timestamp': getTimestampMillis(),
  'event_name': 'purchase',
  'request_method': 'POST',
  'request_url': postUrl,
  'request_body': JSON.stringify(postBody),
});

sendHttpRequest(postUrl, (statusCode, headers, body) => {
  log({
    'tag_name': 'Example',
    'type': 'Response',
    'trace_id': traceId,
    'timestamp': getTimestampMillis(),
    'event_name': 'purchase',
    'response_status_code': statusCode,
    'response_headers': headers,
    'response_body': body,
  });
}, { method: 'POST' }, JSON.stringify(postBody));

sendHttpRequest(postUrl, (statusCode, headers, body) => {
  log({
    'tag_name': 'Example',
    'type': 'Response', // ??????????
    'trace_id': traceId,
    'timestamp': getTimestampMillis(),
    'event_name': 'purchase',
    'request_method': 'POST',
    'request_url': postUrl,
    'request_body': JSON.stringify(postBody),
    'response_status_code': statusCode,
    'response_headers': headers,
    'response_body': body,
  });
}, { method: 'POST' }, JSON.stringify(postBody));

function log(logObject) {
  if (isLoggingEnabled) {
    const connectionInfo = {
      projectId: '<GCP Project ID>',
      datasetId: '<BigQuery Dataset ID>',
      tableId: '<BigQuery Table ID>'
    };
    BigQuery(connectionInfo, [logObject], { ignoreUnknownValues: true });
  }
}

function determinateIsLoggingEnabled() {
   const containerVersion = getContainerVersion();
   const isDebug = !!(
       containerVersion &&
       (containerVersion.debugMode || containerVersion.previewMode)
   );

   if (!data.logType) {
       return isDebug;
   }

   if (data.logType === 'no') {
       return false;
   }

   if (data.logType === 'debug') {
       return isDebug;
   }

   return data.logType === 'always';
}
```

TBD


### Example UI

![UI](/images/gtms-7-ui.png)

TBD
