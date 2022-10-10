# GTMS-6
## Firebase Settings

### Problem that solved by this standard

Server GTM does not have a standard UI for firebase settings.
For each template that uses firebase, you need to create a UI for firebase settings.

### Standard description
 
- The field name `firebaseProjectId` in the template must be used for the Firebase project id.
- The field name `firebasePath` in the template must be used for the Firestore document/collection path.

### Example Code

 ```js
let firebaseOptions = {};
if (data.firebaseProjectId) firebaseOptions.projectId = data.firebaseProjectId;

Firestore.read(data.firebasePath, firebaseOptions)
    .then((result) => {
        return sendConversionRequest(result.data.access_token, data.refreshToken);
    }, () => updateAccessToken(data.refreshToken));
```


### Example UI

![UI](/images/gtms-6-ui.png)

```json
{
  "displayName": "Firebase Settings",
  "name": "firebaseGroup",
  "groupStyle": "ZIPPY_CLOSED",
  "type": "GROUP",
  "subParams": [
    {
      "type": "TEXT",
      "name": "firebaseProjectId",
      "displayName": "Firebase Project ID",
      "simpleValueType": true
    },
    {
      "type": "TEXT",
      "name": "firebasePath",
      "displayName": "Firebase Path",
      "simpleValueType": true,
      "help": "The tag uses Firebase to store the OAuth access token. You can choose any key for a document that will store the OAuth access token.",
      "valueValidators": [
        {
          "type": "NON_EMPTY"
        }
      ],
      "defaultValue": "stape/gads-offline-auth"
    }
  ]
}
```
