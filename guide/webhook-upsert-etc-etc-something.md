---
icon: webhook
---

# Webhook upsert etc etc something

## # What is a webhook

A webhook is a method used by applications to send real-time data to other systems via HTTP POST requests when certain events occur.&#x20;

Unlike traditional APIs where data must be polled periodically, webhooks allow for immediate notification and data transfer, enabling more efficient and responsive communication between systems.&#x20;

They are commonly used for various purposes such as triggering actions, updating information, or integrating different services, and require the receiving endpoint to handle and process the incoming data securely and accurately.

Webhook in Colivar

* upsert async
* will tell you when done
* no need to wait for big doc upload
*

## Why verifying a webhook

Due to the nature of webhooks, there is a risk that attackers could impersonate services by sending fake webhooks to an endpoint. Essentially, it’s just an HTTP POST from an unknown source, which can pose a significant security risk for many applications or, at the very least, create potential problems.

To prevent this, each webhook and its metadata are signed with a unique key specific to each endpoint. This signature can then be used to verify that the webhook is legitimate and should be processed.

Another security risk is known as a replay attack. In a replay attack, an attacker intercepts a valid payload (including the signature) and retransmits it to your endpoint. Because the payload passes signature validation, it will be processed.

To mitigate this, a timestamp indicating when the webhook attempt occurred is included. Webhooks with a timestamp that is more than five minutes different from the current time (either in the past or future) are automatically rejected. This requires your server's clock to be synchronized and accurate, and using NTP is recommended to achieve this.

For more details on webhook vulnerabilities, please refer to the webhook security section of the documentation.

\-- \<Diagram here - will ask Meghan>
