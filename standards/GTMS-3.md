# GTMS-3
## Server Templates Logging

### Problem that solved by this standard

Server GTM does not have a standard format for output logs.
And there is no common way to log Requests and Responses.
This creates issues for log parsing and monitoring.

### Standard description

- Each log line must be a valid JSON object. (This allows parse logs by ELK, CloudWatch, Loki, etc.)
- Each log object must contain at least: `Type`, `Name` and `TraceId` fields.
- Field `Type` must contain one of these values `Request`, `Response`, `Message`. Witch helps easily filter logs.
- `TraceId` must contain a unique identifier of the current incoming request. This field is used for stitching all logs done by all tags/clients during one request. Highly recommend using the `trace-id` header for this, as a lot of clouds/platforms use it for tracing. 
- Field `Name` must contain tag name.

### Example Code

 ```js
const traceId = loggingEnabled ? getRequestHeader('trace-id') : undefined;
const isLoggingEnabled = determinateIsLoggingEnabled();
const postUrl = 'https://example.com';
const postBody = {'data': 'some data'};

if (isLoggingEnabled) {
    logToConsole(JSON.stringify({
        'Name': 'Example',
        'Type': 'Request',
        'TraceId': traceId,
        'EventName': 'purchase',
        'RequestMethod': 'POST',
        'RequestUrl': postUrl,
        'RequestBody': postBody,
    }));
}

sendHttpRequest(postUrl, (statusCode, headers, body) => {
    if (isLoggingEnabled) {
        logToConsole(JSON.stringify({
            'Name': 'Example',
            'Type': 'Response',
            'TraceId': traceId,
            'EventName': makeString(data.conversionActionId),
            'ResponseStatusCode': statusCode,
            'ResponseHeaders': headers,
            'ResponseBody': body,
        }));
    }
}, {method: 'POST'}, JSON.stringify(postBody));



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


### Example UI

![UI](/images/gtms-3-ui.png)

```json
{
    "displayName": "Logs Settings",
    "name": "logsGroup",
    "groupStyle": "ZIPPY_CLOSED",
    "type": "GROUP",
    "subParams": [
      {
        "type": "RADIO",
        "name": "logType",
        "radioItems": [
          {
            "value": "no",
            "displayValue": "Do not log"
          },
          {
            "value": "debug",
            "displayValue": "Log to console during debug and preview"
          },
          {
            "value": "always",
            "displayValue": "Always log to console"
          }
        ],
        "simpleValueType": true,
        "defaultValue": "debug"
      }
    ]
  }
```


### Thanks

@adatzer 
@paulboocock
@jurajfrank
