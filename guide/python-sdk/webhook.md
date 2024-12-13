---
icon: fishing-rod
---

# Webhook

**ColiVara** supports Webhook function for asynchronous document upsertion. If not set up, you will receive an email document upsertion events such as success or failure. You can create and verify the Webhook using our provided SDK, or via the User Interface at [**www.colivara.com**](https://www.colivara.com). Webhooks can be useful to manage downstream even handling.



## Setting up a webhook via the User Interface

{% stepper %}
{% step %}
### Visit your <mark style="color:purple;">Account Dashboard</mark>

Via [https://colivara.com/accounts/edit-account/](https://colivara.com/accounts/edit-account/)
{% endstep %}

{% step %}
### Enter your webhook URL

Make sure that your URL enpoint is _Publicly Available_

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
{% endstep %}

{% step %}
### Store your Webhook Secret

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
{% endstep %}
{% endstepper %}

## Setting up a web hook via the SDK

The Python SDK library provides the method to regsiter a webhook URL

**`add_webhook`**&#x20;

**Parameters:**

* `url(str)`: The URL of the webhook endpoint to register. This must be a valid, publicly accessible URL.

**Returns:**

* `WebhookOut`: An object containing details about the registered webhook, including the webhook ID, associated app ID, and _webhook secret_.&#x20;

**Exceptions:**

* `ValueError`: Raised if the provided URL is invalid or the request is rejected due to a bad request (e.g., missing or incorrect parameters).
* `requests.HTTPError`: Raised for other HTTP errors, such as server-side issues.

**Example:**

```python
# Register a webhook to handle document upsertion events
webhook = client.add_webhook(
    url="https://your-domain.com/webhook-handler"
)
```

## Verifying Webhook from inside your application

Now that ColiVara have noted your app URL to send notification to, you will start receiving HTTP request containing information regarding your document process status.&#x20;

However, prior to ingesting the data from these HTTP requests, you _should_ validate the webhook to verify that it truly comes from ColiVara. Afterall, any malicious actor can send an HTTP request to a publicly available URL. Your webhook secret is a unique key used to verify that incoming webhook requests are authentic. This protects your application from unauthorized or fake notifications.

{% hint style="warning" %}
Keep your webhook secret safe and never share it publicly. If exposed, attackers could forge requests and compromise your system.
{% endhint %}

The SDK provides a method that you can use to validate the webhook from inside your Python application&#x20;



**`validate_webhook`:**

**Parameters:**

* `webhook_secret (str)`: The secret key provided when the webhook was registered. Used for validation.
* `payload (str)`: The raw body of the incoming webhook request.
* `headers (Dict[str, Any])`: The headers from the incoming webhook request, which include the signature used for validation.

**Returns:**

* `bool`: Returns `True` if the webhook is valid; otherwise, returns `False`.

**Example:**

```python
# Validate an incoming webhook request
is_valid = client.validate_webhook(
    webhook_secret="your_webhook_secret",
    payload=raw_payload,
    headers=request_headers
)

if is_valid:
    print("Webhook is valid!")
else:
    print("Invalid webhook!")

```

