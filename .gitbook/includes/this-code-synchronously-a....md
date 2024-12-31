---
title: '# This code synchronously a...'
---

{% code lineNumbers="true" %}
```python
# This code synchronously adds/updates an "AI Research Paper" document  in the "AI_Papers" collection
document = client.upsert_document(
    name="AI_Research_Paper",
    metadata={
        "category": "Machine Learning",
        "year": "2024",
        "author": "Dr. AI Researcher"
    },
    collection_name="AI_Papers",
    document_path="/path/to/AI_Research_Paper.pdf",
    wait=True
```
{% endcode %}
