# ColPali Model

Traditional document parsing techniques rely heavily on extracting plain texts, at the cost of overlooking graphical elements. However, complex documents such as technical papers and presentations provide much of their context visual cues such as such as tables, images, charts, and the layout structure. As such, visual context is vital to extract, interpret, and prioritize information in order to generate contexually accurate response.&#x20;

In short, ColPali is an advanced document retrieval model that leverages Vision Language Models to integrate both <mark style="color:purple;">**textual**</mark> and <mark style="color:purple;">**visual**</mark> elements for highly accurate and efficient document search&#x20;

{% hint style="success" %}
**ColiVara** uses **ColQwen**, an improved model from ColPali.&#x20;
{% endhint %}

## Standard Document Retrieval technology

Document Retrieval technology is central to many applications, either ass a standalone ranking system, or as part of a complex Retrieval Augmented Generation (RAG) pipeline.&#x20;

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

There are 2 phases in a Standard  Retrieval :

<table data-view="cards"><thead><tr><th></th></tr></thead><tbody><tr><td>An <mark style="color:purple;"><strong>Offline Indexing</strong></mark> phase, where all the documents from the corpus are indexed.</td></tr><tr><td>An <mark style="color:purple;"><strong>Online Querying</strong></mark> phase, where a user query is matched with a low latency to the pre-computed document index.</td></tr></tbody></table>

To index a standard PDF document, the Offline Indexing process might look like this:&#x20;

* PDF parsers or Optical Character Recognition (OCR) systems extract text from document pages.
* Layout detection models segment the document into structured parts.&#x20;
* Optional captioning step, which provides natural language descriptions for visual elements, making them more compatible with embedding models.
* A chunking strategy groups related text passages to maintain semantic coherence
* Text embeddings are created by mapping vectors meant to represent the text's semantic meaning.

Finally, the documents - with its generated embeddings - are ready to be queried. A Query follows a similar process to be converted into its vector presentation. The query's vector can be used on the document's vector to produce an answer. &#x20;

{% hint style="warning" %}
While there have been large improvements in text embedding models, practical experiments have shown that the performance bottleneck lies in the ingestion pipeline in which the _visually rich_ document was processed, prior to being consumed by an LLM. Additionally, this process is slow, difficult, and prone to propagate errors.&#x20;
{% endhint %}

***

## ColPali Model&#x20;

**ColPali** is a novel approach to Document Retrieval, by doing away with the textual indexing phase, and replacing this process by generating embeddings on the images (or screenshots of documents) directly.&#x20;

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

ColPali was built on previous technlogies:

<details>

<summary>PaliGemma</summary>

A Vision Language Model that processes images by breaking them down into smaller sections, or _patches_. These patches are analyzed by a vision transformer (_SigLIP-So400m)_ in order to produce detailed vector representations, known as _patch embeddings_.&#x20;

These _embeddings_ are fed into a language model (_Gemma 2B_) to create representations of each patch within the language model's space. This results in a multi-vector representation for each page image, with both visual and contextual information.

</details>

<details>

<summary>ColBERT with Late Interaction</summary>

A specialized search model that combines _Multi-Vector Representation_ with a _Late Interaction_ approach&#x20;

* **Multi-Vector Representation**: Unlike traditional retrieval methods that summarize an entire document with a single vector, ColBERT creates separate vectors for each word or phrase in a document and query. As a result, ColBERT can retain more information and identify nuances in contexts.&#x20;
* **Late Interaction Mechanism**: Instead of performing complex interactions between vectors for the query and document upfront, ColBERT initially calculates a broad similarity score to _cheaply_ narrow down potential matches. Detailed similarity comparisons are only one on the most relevant parts of the document.&#x20;

</details>

{% hint style="success" %}
ColPali, a Paligemma-3B extension that is capable of generating ColBERT-style multi-vector representations of text and images, optimizes the ingestion pipeline of visually rich document by creating embeddings on the visual elements using Vision Language Model. This yields much greater performance boost than optimizing the text embedding model.
{% endhint %}

***

### Performance&#x20;

ColPali exhibits high efficiency:&#x20;

* **High retrieval performance**: The use of Vision Language Model creates high-quality contextual embeddings from document images, facilitating quicker retrieval.
* **Low querying latency**: Late interaction mechanism significantly decreases the number of comparisons needed during querying.&#x20;
* **High indexing speed**_:_ Simpler indexing process by directly processing document images, eliminating the need for complex text extraction and segmentation pipelines.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption><p>Both visually demanding and text centric document are better retrieved using the ColPali model</p></figcaption></figure>



***

## Citation

Faysse, M., Sibille, H., Wu, T., Omrani, B., Viaud, G., Hudelot, C., & Colombo, P. _ColPali: Efficient Document Retrieval with Vision Language Models_. Illuin Technology, Equall.ai, CentraleSupélec, Paris-Saclay, ETH Zürich. Available at: [https://arxiv.org/pdf/2407.01449](https://arxiv.org/pdf/2407.01449)
