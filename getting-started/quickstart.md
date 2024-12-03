---
icon: bullseye-arrow
---

# Quickstart

<figure><img src="../.gitbook/assets/Screenshot 2024-10-18 at 3.15.09 PM.png" alt="" width="188"><figcaption></figcaption></figure>

ColiVara is an web API that abstracts all the difficult parts about visual RAG. It embeds and saves documents, and then returns the highest matching pages when a user makes a query.

{% hint style="info" %}
We use the Python SDK in this quickstart, but since ColiVara is an API, you can use any language by making standard API calls.
{% endhint %}

### API Keys

Get an API Key from the [ColiVara Website](https://colivara.com) or via self-hosting.&#x20;

### Install the Python SDK

```bash
pip install colivara-py
```

### Index a document

Colivara accepts **a file url, or base64 encoded file, or a file path**. We support over 100 file formats including PDF, DOCX, PPTX, and more. We will also automatically take a screenshot of URLs (webpages) and index them.

```python
import os
from colivara_py import ColiVara

rag_client = ColiVara(
    # this is the default and can be omitted
    api_key=os.environ.get("COLIVARA_API_KEY"),
    # this is the default and can be omitted
    base_url="https://api.colivara.com"
)

# Upload a document to the default collection
document = rag_client.upsert_document(
    name="attention is all you need",
    url="https://arxiv.org/abs/1706.03762",
    metadata={"published_year": "2017"}
)
```

### Search

You can filter by collection name, collection metadata, and document metadata. You can also specify the number of results you want.

```python
results = rag_client.search(query="What is the role of self-attention in transformers?")
print(results) # top 3 pages with the most relevant information
```

### FAQ

<details>

<summary>Do I need a vector database?</summary>

No - ColiVara uses Postgres and pgVector to store vectors for you. You DO NOT need to generate, save, or manage embeddings in anyway.

</details>

<details>

<summary>Do you convert the documents to markdown/text?</summary>

No - ColiVara treats everything as an image, and uses vision models. There are no parsing, chunking, or OCR involved. This method outperforms chunking, and OCR for both text-based documents and visual documents.

</details>

<details>

<summary>How does non-pdf documents or web pages work?</summary>

We run a pipeline to convert them to images, and perform our normal image-based retrieval. This all happen for you under the hood, and you get the top-k pages when performing retrieval.

</details>

<details>

<summary>Can I use my vector database? </summary>

Yes - we have an embedding endpoint that only generates embeddings without saving or doing anything else. You can store these embeddings at your end. Keep in mind that we use late-interaction and multi-vectors, many vector databases do not support this yet.

</details>



