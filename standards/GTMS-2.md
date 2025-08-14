# GTMS-2
## Server Templates Consent Management

### Problem that solved by this standard

Server GTM does not have a built-in consent management system.
Most of the SGTM templates require ad_storage consent.
This standard provides a way to manage consent in the SGTM templates.

### Standard description

- Each template must have a `consentSettingsGroup` parameter.
- `consentGroup` must contain a `adStorageConsent` parameter.
- `adStorageConsent` must contain one of these values `optional`, `required`.
- By default, `adStorageConsent` must be `optional`.
- If `adStorageConsent` is `required`, the template must check if the consent is given and send data only if the consent is given.
- If `adStorageConsent` is `optional`, the template must send data always.
- The template must check if the consent is given by the `consent_state` or `x-ga-gcs` property.

### Example Code

 ```js
const eventData = getAllEventData();

if (!isConsentGivenOrNotRequired()) {
  return data.gtmOnSuccess();
}

function isConsentGivenOrNotRequired() {
  if (data.adStorageConsent !== 'required') return true;
  if (eventData.consent_state) return !!eventData.consent_state.ad_storage;
  const xGaGcs = eventData['x-ga-gcs'] || ''; // x-ga-gcs is a string like "G110"
  return xGaGcs[2] === '1';
}
```


### Example UI

![UI](/images/gtms-2-ui.png)

```json
{
  "type": "GROUP",
  "name": "consentSettingsGroup",
  "displayName": "Consent Settings",
  "groupStyle": "ZIPPY_CLOSED",
  "subParams": [
    {
      "type": "RADIO",
      "name": "adStorageConsent",
      "displayName": "",
      "radioItems": [
        {
          "value": "optional",
          "displayValue": "Send data always"
        },
        {
          "value": "required",
          "displayValue": "Send data in case marketing consent given",
          "help": "Aborts the tag execution if marketing consent (<i>ad_storage</i> Google Consent Mode or Stape's Data Tag parameter) is not given."
        }
      ],
      "simpleValueType": true,
      "defaultValue": "optional"
    }
  ]
}
```
