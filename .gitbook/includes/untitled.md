---
title: Untitled
---



{% tabs %}
{% tab title="First Tab" %}
`search` sends a query to the server, specifying which collection to search within, the number of top results to return, and optional filters to refine the search. It returns the most relevant results based on the given parameters.
{% endtab %}

{% tab title="Second Tab" %}


* `query` (`str`): The main search string used to find similar documents or pages.
* `collection_name` (`str`, optional): Specifies which collection to search in. The default value is `"all"`, meaning it searches across all collections.
* `top_k` (`int`, optional): Limits the number of results returned. Defaults to `3`, returning the top three matches.
* `query_filter` (`Optional[Dict[str, Any]]`, optional): An optional dictionary to filter results based on specific criteria. This dictionary can include:
  * `on`: Specifies the filter target, either `"document"` or `"collection"`.
  * `key`: Key(s) to filter by; can be a string or a list of strings.
  * `value`: The value(s) the key(s) should match; could be a string, integer, float, or boolean.
  * `lookup`: Specifies how to match the filter criteria, with options like `"key_lookup"`, `"contains"`, `"contained_by"`, `"has_key"`, `"has_keys"`, or `"has_any_keys"`.
{% endtab %}

{% tab title="Untitled" %}
```
afasd
```
{% endtab %}
{% endtabs %}
