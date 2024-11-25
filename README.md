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

It is a web-first implementation of the [ColPali paper](https://arxiv.org/abs/2407.01449) using ColQwen2 as the LLM model. It works exactly like RAG from the end-user standpoint - but using vision models instead of chunking and text-processing for documents.

### Performance

{% hint style="info" %}
ColiVara is the **Leading Real-World tool for Practical Solutions and Efficiency in Retrieval-Augmented Generation Technology.** We significantly outperfomed currently in-use methods for document parsing and processing.&#x20;
{% endhint %}

Our detailed [**Benchmark Performance  Evaluation**](https://app.gitbook.com/o/DHjMRQs4Vfjvi2m062WT/s/r0RGXIvkomMWAuxgus2N/~/changes/31/getting-started/about/colivara-detailed-evaluation-result) have illustrated Colivara's superiority across diverse benchmarks. Metrics like **NDCG@5** score(Normalized Discounted Cumulative Gain at rank 5) **Latency** were recorded for a comprehensive analysis.

| Benchmark               | Colivara Score | Avg Latency (s) (lower is better) | Num Docs |
| ----------------------- | -------------- | --------------------------------- | -------- |
| Average                 | 87.6           | N/A                               | N/A      |
| ArxivQA                 | 88.1           | 11.1                              | 500      |
| DocVQA                  | 56.1           | 9.3                               | 500      |
| InfoVQA                 | 91.4           | 8.6                               | 500      |
| Shift Project           | 91.3           | 16.8                              | 1000     |
| Artificial Intelligence | 99.5           | 12.8                              | 1000     |
| Energy                  | 96.3           | 14.1                              | 1000     |
| Government Reports      | 96.7           | 14.0                              | 1000     |
| Healthcare Industry     | 98.3           | 20.0                              | 1000     |
| TabFQuad                | 86.3           | 8.1                               | 280      |
| TatQA                   | 71.7           | 20.0                              | 1663     |

### **Key Findings**

* ColiVara dominated _visual-heavy benchmarks_ like ArxivQA and InfoQA with NDCG@5 score of **88.1**, _double_ the performance of captioning-based systems.
* Even for on _text-centric benchmarks_, ColiVara outperformed traditional methods by up to **30%** on  benchmarks like DocQA and multimodal benchmarks like InfoQA.
* For more comprehensive benchmarks, where a holistic approach of visual and textual analysis is key to query generation, such as the key for queries in specific domains (Sustainability, Energy, AI, Government Report, Healthcare), ColiVara shines overwhelmingly over competions, scoring in the **high 90s** for all benchmarks. This is due to ColiVara's Holistic Multimodal Integration and Spatial Context Awareness. \


### Jump right in

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th></th><th data-card-target data-type="content-ref"></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden></th><th data-hidden data-type="content-ref"></th></tr></thead><tbody><tr><td><a href="getting-started/quickstart.md"><strong>Getting Started</strong></a></td><td>Create your first RAG pipeline with 2 lines of code</td><td></td><td></td><td></td><td></td><td></td><td><a href="getting-started/quickstart.md">quickstart.md</a></td></tr><tr><td><a href="broken-reference"><strong>Guides</strong></a></td><td>Learn all what ColiVara have to offer</td><td></td><td></td><td></td><td></td><td></td><td><a href="broken-reference">Broken link</a></td></tr><tr><td><a href="guide/api-reference.md"><strong>API Reference</strong></a></td><td>Try the API live</td><td></td><td></td><td></td><td></td><td></td><td></td></tr></tbody></table>
