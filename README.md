# Healthengine Booking Webhooks

Documentation for booking webhooks from the Healthengine platform.

## Australian Privacy Laws
To comply with Australian Privacy Laws (APPs), webhook endpoints must be located in Australia. This is to ensure adherence to Chapter 8: APP 8 Cross-border disclosure of personal information ([https://www.oaic.gov.au/privacy/australian-privacy-principles/australian-privacy-principles-guidelines/chapter-8-app-8-cross-border-disclosure-of-personal-information]). As HealthEngine does not currently support cross-border data transfers, hosting webhook endpoints within Australia is mandatory.

## Synopsis

Webhooks can be utilised to notify external parties of booking events that have occurred within the Healthengine platform.

The table below shows the events that currently support webhooks.

| Event Name                                                            | Description                                                                    |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| [`booking-submitted`](#booking-submitted)                             | A booking was made using the Healthengine platform.                            |
| [`booking-updated`](#booking-updated)                                 | A booking was updated on the Healthengine platform.                            |
| [`booking-cancelled`](#booking-cancelled)                             | A booking has been cancelled on the Healthengine platform.                     |
| [`booking-pre-screening-submitted`](#booking-pre-screening-submitted) | A booking has had a pre-screening form submitted on the Healthengine platform. |
| [`booking-marked-attended`](#booking-marked-attended)                 | A booking has been marked as attended on the Healthengine platform.            |

## Event Details

Webhooks are configured per-practice and will be delivered to the practice’s configured URL.

Some general notes and limitations of event webhooks:

- Imported bookings from data migrations will not trigger booking webhooks.
- Due to the highly available and redundant architecture, duplicate webhooks may be sent, including simultaneously,
- If the HTTP response code is not 2xx or 400, the webhook will be retried,
- A maximum of 3 retries are made before the webhook is marked as failed,
- The retries have a backoff strategy of 5 minutes, 1 hour and 12 hours,
- Webhooks will only be delivered to HTTPS,
- A webhook subscription must be confirmed before events are sent ([see below](#subscribing-to-an-event)),
- A response must be received within 5 seconds or the request will be terminated. Any long running process should be triggered asynchronously,

### `booking-submitted`

This event is triggered after a booking has been made within the Healthengine platform. An example payload is shown below.


```json
{
  "version": 1,
  "type": "booking-submitted",
  "data": {
    "appointment": {
      "datetime": 1577808000,
      "type": "General Appointment",
      "specialty": "General Practice",
    },
    "booking_id": "1234",
    "booker": null |
      {
        "firstname": "Joe",
        "lastname": "Blogs",
        "email": "no-reply@healthengine.com.au",
        "mobile_phone": "0412345678"
      },
    "manage_booking_url": "https://www.google.com",
    "practice": {
      "id": "9876",
      "timezone": "Australia/Perth",
      "name": "Sample Practice WA",
      "address": {
        "postcode": "6000",
      },
    },
    "patient": {
      "address": {
        "postcode": "6000",
        "state": "WA",
        "street": "Wellington St",
        "suburb": "Perth"
      },
      "dob": "1970-01-01",
      "email": null | "noreply@healthengine.com.au",
      "firstname": "Jane",
      "gender": null | "Female" | "Male" | "Other",
      "lastname": "Blogs",
      "medicare":
        null |
        {
          "expiry": "01/2022",
          "number": "1",
          "reference": "1234 56789 0"
        },
      "mobile_phone": null | "0412345678"
    },
    "external_user":
      null |
      {
        "id": "12345"
      },
    "voucher_code": null | "ABC123",
    "selected_medication": null | "abacavir 300 mg/each",
    "external_payments": null | {
      "external_payment_id": "06019f23-5d22-4310-9a7e-304f5e7361dd",
      "amount": 1700,
      "refunded": false
    },
    "exclusive_scheduling": null | {
      "client": "Things Inc.",
      "location": "123 Fake St",
      "additional_message": "Knock twice and provide secret password"
    }
  }
}
```

### `booking-updated`

This event is triggered after a booking has been updated within the Healthengine platform.

At this time the only updateable fields are:

- Patient Details:
  - email
  - mobile_phone
  - firstname
  - lastname
  - address
- Booker Details:
  - email
  - mobile_phone
  - firstname
  - lastname
- Payment Method

An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-updated",
  "data": {
    "appointment": {
      "datetime": 1577808000,
      "type": "General Appointment",
      "specialty": "General Practice"
    },
    "booking_id": "1234",
    "booker": null |
      {
        "firstname": "Joe",
        "lastname": "Blogs",
        "email": "no-reply@healthengine.com.au",
        "mobile_phone": "0412345678"
      },
    "manage_booking_url": "https://www.google.com",
    "practice": {
      "id": "9876",
      "timezone": "Australia/Perth",
      "name": "Sample Practice WA",
      "address": {
        "postcode": "6000",
      },
    },
    "patient": {
      "address": {
        "postcode": "6000",
        "state": "WA",
        "street": "Wellington St",
        "suburb": "Perth"
      },
      "dob": "1970-01-01",
      "email": null | "noreply@healthengine.com.au",
      "firstname": "Jane",
      "lastname": "Blogs",
      "medicare":
        null |
        {
          "expiry": "01/2022",
          "number": "1",
          "reference": "1234 56789 0"
        },
      "mobile_phone": null | "0412345678"
    },
    "external_user":
      null |
      {
        "id": "12345"
      },
    "voucher_code": null | "ABC123",
    "external_payments": null | {
      "external_payment_id": "06019f23-5d22-4310-9a7e-304f5e7361dd",
      "amount": 1700,
      "refunded": false
    },
    "exclusive_scheduling": null | {
      "client": "Things Inc.",
      "location": "123 Fake St",
      "additional_message": "Knock twice and provide secret password"
    }
  }
}
```

### `booking-cancelled`

This event is triggered after a booking has been cancelled on the Healthengine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-cancelled",
  "data": {
    "booking_id": "1234",
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
      "email": null | "noreply@healthengine.com.au",
      "firstname": "Jane",
      "lastname": "Blogs",
      "mobile_phone": null | "0412345678"
    },
    "cancelled_time": 1537809000
  }
}
```

### `booking-pre-screening-submitted`

This event is triggered after a pre-screening form has been submitted on the Healthengine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-pre-screening-submitted",
  "data": {
    "booking_id": "1234",
    "pre_screening_form": {
      "question_answers": [
        {
          "question_id": "aeb110ab-6446-481b-b383-908be888fe37",
          "answer": {
            "value": true | false
          },
          "question_wording": "You have had any vaccine in the past month",
          "variant": "YES_NO"
        },
        {
          "question_id": "f98ae1df-b110-4e0f-83a6-fe6b69a28c48",
          "answer": {
            "value": true | false,
            "specification": "I am a nurse." | null
          },
          "question_wording": "You have an occupation or lifestyle factor(s) for which vaccination may be needed (discuss with doctor/pharmacist/nurse). Please specify below:",
          "variant": "YES_NO_SPECIFY"
        },
        {
          "question_id": "0b039374-5f98-4078-917d-659505528723",
          "question_wording": "Before any vaccination takes place, clinic staff should ask you:
- Is this a bullet point?
- Is **this** in bold?
- Is _this_ italicised?
",
          "answer": null,
          "variant": "INFORMATION"
        }
      ]
    },
    "practice": {
      "id": "9876"
    },
    "submitted_at": 1537809000
  }
}
```

### `booking-marked-attended`

This event is triggered after a booking has been marked as attended on the Healthengine platform. An example payload is shown below.

```json
{
  "version": 1,
  "type": "booking-marked-attended",
  "data": {
    "appointment": {
      "datetime": 1577808000
    },
    "booking_id": "1234",
    "practice": {
      "id": "9876",
      "name": "Practice Name"
    },
    "patient": {
      "address": {
        "postcode": "6000",
        "state": "WA",
        "street": "Wellington St",
        "suburb": "Perth"
      },
      "dob": "1970-01-01",
      "email": null | "noreply@healthengine.com.au",
      "firstname": "Jane",
      "lastname": "Blogs",
      "medicare":
        null |
        {
          "expiry": "01/2022",
          "number": "1",
          "reference": "1234 56789 0"
        },
      "mobile_phone": null | "0412345678"
    },
    "attended": true | false,
    "attended_changed_at": 1577808000
  }
}
```

## Expected Responses

- Any 2xx response code will be treated as successful, any body returned will be discarded.
- A 400 response code can be returned to signify issues with the message, in which case it will not be retried. It is recommended that the response include a payload with a description of the issue. eg.
  ```json
  {
    "error_code": "A-unique-code-to-this-type-of-error",
    "error_message": "A more descriptive explanation of the error"
  }
  ```
- Any other response code will be treated as a failure that should be retried as detailed above.

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

To secure your endpoint it is suggested to whitelist Healthengine's IP addresses being:

- 13.55.48.1
- 52.62.53.70
- 13.54.122.64
