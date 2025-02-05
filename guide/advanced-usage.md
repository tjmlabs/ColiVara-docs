---
icon: bow-arrow
---

# Advanced usage

### Overview

The `use_proxy` parameter in the Colivara API is a boolean flag that allows users to route their requests through a proxy. This can be useful when accessing certain sites that may block direct requests. However, enabling this feature incurs an additional cost of **10 extra credits per request**.

### Default Behavior

By default, `use_proxy` is set to `False`, meaning requests are sent directly without using a proxy.

### Usage

To enable proxy usage, simply set `use_proxy` to `True` in your request payload. Below is an example of how to use the parameter in a request:

```python
import requests

response = requests.post(
    "https://api.colivara.com/v1/documents/upsert-document/",
    headers={"Authorization":"Bearer <token>","Content-Type":"application/json"},
    json={"name":"text",
          "metadata":{},
          "collection_name":"default_collection",
          "url": "string",
          "wait": False,
          "use_proxy": True
    }
)
data = response.json()
```



### Considerations

* **Additional Cost**: Enabling `use_proxy` incurs an additional charge of **10 extra credits per request**.
* **When to Use**: Use this option when dealing with sites that restrict direct access. Proxies are helpful for accessing some sites that may block us when not using a proxy.
