# HealthEngine Booking Webhooks

Documentation for booking webhooks from the HealthEngine platform.

## Synopsis

Webhooks can be utilised to notify external parties of booking events that have occurred within the HealthEngine platform.

The table below shows the events that currently support webhooks.

| Event Name | Description |
|---|---|
| [`booking-submitted`](#booking-submitted) | A booking was made using the HealthEngine platform. |

## Event Details

Webhooks are configured per-practice and will be delivered to the practice’s configured URL.

Some general notes and limitations of event webhooks:

- Due to the highly available and redundant architecture, duplicate webhooks may be sent, including simultaneously,
- If the HTTP response code is not 200, the webhook will be retried,
- A maximum of 3 retries are made before the webhook is marked as failed,
- The retries have a backoff strategy of 5 minutes, 1 hour and 12 hours,
- Webhooks will only be delivered to HTTPS,
- A webhook subscription must be confirmed before events are sent ([see below](#subscribing-to-an-event)),
- A response must be received within 5 seconds or the request will be terminated. Any long running process should be triggered asynchronously,

### `booking-submitted`

This event is triggered after a booking has been made within the HealthEngine platform. An example payload is shown below.

```json
{
    "version": 1,
    "type": "booking-submitted",
    "data": {
        "booking_id": "1234",
        "appointment": {
            "datetime": "2020-01-01 10:00",
            "type": "General Appointment",
            "timezone": "Australia/Sydney"
        }
        "practice": {
            "id": "9876"
        },
        "patient": {
            "address": {
                "postcode": "6000",
                "state": "WA",
                "street": "Wellington St",
                "suburb": "Perth"
            },
            "dob": "1970-01-01",
            "email": "noreply@healthengine.com.au",
            "firstname": "Jane",
            "lastname": "Blogs",
            "mobile_phone": "0412345678"
        },
    }
}
```

## Subscribing to an event

Before real events are sent, the webhook URL subscription must be confirmed. This ensures that the URL is valid and correct.  
When a new webhook is configured, an example payload is sent to the URL with the JSON payload below.  
The third party must retrieve the confirmation_url from the body and make a HTTPS request to that endpoint to confirm the subscription.  
Once we receive this request the webhook will be enabled and events will begin being delivered to the endpoint.

```json
{
    "version": 1,
    "type": "subscription-confirmation",
    "data": {
        "confirmation_url": "https://healthengine.com.au/api/4/practice-webhook-subscription?id=1234&signature=9023a0ef234"
    }
}
```

_Example:_ Here is an example cURL request to confirm a webhook subscription.

`curl --request GET --url 'https://healthengine.com.au/api/4/practice-webhook-subscription?id=1234&signature=9023a0ef234' --header 'Accept: application/json'`

_Note:_ Since the configured endpoint can receive payloads of different formats, the third party can inspect the type parameter in the JSON body to determine which type of payload has been received. This can be used to infer the structure of the data parameter.

_Note:_ The `confirmation_url` is only valid for 1 hour.

_Note:_ When making the confirmation request, you must set the Accept header to application/json. The response has the format: `{"success": true}`.

## Whitelist IPs
To secure your endpoint it is suggested to whitelist HealthEngine's IP addresses being:

- 13.55.48.1
- 52.62.53.70
- 13.54.122.64
