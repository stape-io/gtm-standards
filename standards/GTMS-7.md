# GTMS-7
## Server Templates BigQuery Logging

### Problem that is solved by this standard

GTM Server-Side does not have a common format for writing logs to `BigQuery`. This standard ensures structured logging, enabling easier querying, debugging, and monitoring.

### Standard description

- Each log line is a row in BigQuery.
- Each insertion to the BigQuery table must add only one row at a time.
- Each log line must contain at least: `timestamp`, `type`, `tag_name` and `trace_id` fields.
- Field `timestamp` must contain the Unix epoch timestamp in milliseconds.
- Field `type` must contain one of these values `Request`, `Response`, `Request-Response`, `Message`. Which helps easily filter logs.
- `trace_id` must contain a unique identifier of the current incoming request. This field is used for stitching all logs done by all tags/clients during one request. It's highly recommended using the `trace-id` header for this, as many clouds/platforms use it for tracing. 
- Field `tag_name` must contain the tag name.
- All fields must use snake case naming, e.g. `trace_id`, `response_code`etc. and *not* `TraceId`, `ResponseCode` etc.
- `ignoreUnknownValues` must be set to `true` in `BigQuery.insert(...)` to handle schema changes gracefully. This allows new fields to be added without breaking existing logs.


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

Combine the fields from both `Request` and `Response` types when receiving the response.

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
const getRequestHeader = require('getRequestHeader');
const getTimestampMillis = require('getTimestampMillis');
const isLoggingEnabledForBigQuery = determinateIsLoggingEnabledForBigQuery();
const traceId = isLoggingEnabled ? getRequestHeader('trace-id') : undefined;

const postUrl = 'https://example.com';
const postBody = { 'data': 'some data' };

// 'Message'
if (validateMissingRequiredFields()) {
  logToBigQuery({
    'tag_name': 'Example',
    'type': 'Message',
    'trace_id': traceId,
    'timestamp': getTimestampMillis(),
    'event_name': 'purchase',
    'message': 'Request won\'t be sent.',
    'reason': 'One or more fields are missing: Email, Phone Number or External ID.'
  });
  return data.gtmOnFailure();
}

// 'Request'
logToBigQuery({
  'tag_name': 'Example',
  'type': 'Request',
  'trace_id': traceId,
  'timestamp': getTimestampMillis(),
  'event_name': 'purchase',
  'request_method': 'POST',
  'request_url': postUrl,
  'request_body': JSON.stringify(postBody)
});

// 'Response'
sendHttpRequest(postUrl, (statusCode, headers, body) => {
  logToBigQuery({
    'tag_name': 'Example',
    'type': 'Response',
    'trace_id': traceId,
    'timestamp': getTimestampMillis(),
    'event_name': 'purchase',
    'response_status_code': statusCode,
    'response_headers': JSON.stringify(headers),
    'response_body': body
  });
}, { method: 'POST' }, JSON.stringify(postBody));

// 'Request-Response'
sendHttpRequest(postUrl, (statusCode, headers, body) => {
  logToBigQuery({
    'tag_name': 'Example',
    'type': 'Request-Response',
    'trace_id': traceId,
    'timestamp': getTimestampMillis(),
    'event_name': 'purchase',
    'request_method': 'POST',
    'request_url': postUrl,
    'request_body': JSON.stringify(postBody),
    'response_status_code': statusCode,
    'response_headers': JSON.stringify(headers),
    'response_body': body
  });
}, { method: 'POST' }, JSON.stringify(postBody));

function logToBigQuery(logObject) {
  if (isLoggingEnabledForBigQuery) {
    const connectionInfo = {
      projectId: data.logBigQueryProjectId,
      datasetId: data.logBigQueryDatasetId,
      tableId: data.logBigQueryTableId
    };
    BigQuery.insert(connectionInfo, [logObject], { ignoreUnknownValues: true });
  }
}

function determinateIsLoggingEnabledForBigQuery() {
  if (data.bigQueryLogType === 'no') return false;
  return data.bigQueryLogType === 'always';
}
```

### Example UI

![UI 1](/images/gtms-7-ui-1.png)
![UI 2](/images/gtms-7-ui-2.png)

```json
{
  "displayName": "BigQuery Logs Settings",
  "name": "bigQueryLogsGroup",
  "groupStyle": "ZIPPY_CLOSED",
  "type": "GROUP",
  "subParams": [
    {
      "type": "RADIO",
      "name": "bigQueryLogType",
      "radioItems": [
        {
          "value": "no",
          "displayValue": "Do not log to BigQuery"
        },
        {
          "value": "always",
          "displayValue": "Log to BigQuery"
        }
      ],
      "simpleValueType": true,
      "defaultValue": "no"
    },
    {
      "type": "GROUP",
      "name": "logsBiqQueryConfigGroup",
      "displayName": "",
      "groupStyle": "NO_ZIPPY",
      "subParams": [
        {
          "type": "TEXT",
          "name": "logBigQueryProjectId",
          "displayName": "BigQuery Project ID",
          "simpleValueType": true,
          "help": "Optional.  <br><br>  If omitted, it will be retrieved from the environment variable <I>GOOGLE_CLOUD_PROJECT</i> where the server container is running. If the server container is running on Google Cloud, <I>GOOGLE_CLOUD_PROJECT</i> will already be set to the Google Cloud project's ID."
        },
        {
          "type": "TEXT",
          "name": "logBigQueryDatasetId",
          "displayName": "BigQuery Dataset ID",
          "simpleValueType": true,
          "valueValidators": [
            {
              "type": "NON_EMPTY"
            }
          ]
        },
        {
          "type": "TEXT",
          "name": "logBigQueryTableId",
          "displayName": "BigQuery Table ID",
          "simpleValueType": true,
          "valueValidators": [
            {
              "type": "NON_EMPTY"
            }
          ]
        }
      ],
      "enablingConditions": [
        {
          "paramName": "bigQueryLogType",
          "paramValue": "always",
          "type": "EQUALS"
        }
      ]
    }
  ]
}
```
