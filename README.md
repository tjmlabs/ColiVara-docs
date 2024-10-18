---
icon: hand-wave
cover: https://gitbookio.github.io/onboarding-template-images/header.png
coverY: 0
layout:
  cover:
    visible: true
    size: full
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Welcome

Welcome to the ColiVara documentation! Here you'll get an overview of all the features ColiVara offers to help you build a state of the art retrieval system.

Colivara is a suite of services that allows you to store, search, and retrieve documents based on their _**visual**_ embeddings.

<details>

<summary>Why visual embeddings?</summary>

Documents are visually rich structures that convey information through text, as well as tables, figures, page layouts, and charts. While legacy document retrieval systems exhibit good performance on query-to-text matching, they struggle to pass visual cues efficiently to large language models, hindering their performance on practical document retrieval applications such as Retrieval Augmented Generation.

</details>

It is a web-first implementation of the [ColiPali paper](https://arxiv.org/abs/2407.01449) using ColQwen2 as the LLM model. It works exactly like RAG from the end-user standpoint - but using vision models instead of chunking and text-processing for documents.



<details>

<summary>Performance</summary>

Vision based retrieval achieves state of the art performance on both latency and accuracy. Beating text extraction with BM25 search and LLM image captioning.

</details>

### Jump right in

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Getting Started</strong></td><td>Create your first RAG pipeline</td><td></td><td></td><td><a href="getting-started/quickstart.md">quickstart.md</a></td></tr><tr><td><strong>Basics</strong></td><td>Learn the basics of ColiVara</td><td></td><td></td><td><a href="broken-reference">Broken link</a></td></tr><tr><td><strong>Advanced Usage</strong></td><td>Unleash the first </td><td></td><td></td><td><a href="getting-started/publish-your-docs.md">publish-your-docs.md</a></td></tr></tbody></table>
