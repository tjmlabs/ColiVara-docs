---
icon: plug-circle-plus
---

# Integrations

{% hint style="info" %}
We are adding more integrations regularly. If you are looking for a specific integration, you can request one via our [Github issues](https://github.com/tjmlabs/ColiVara/issues).&#x20;
{% endhint %}

In this section, we’ll guide you through integrating ColiVara with LLM frameworks. This integration will allow you to seamlessly connect to other moving parts in your RAG project.&#x20;

Currently, we are supporting integrating with&#x20;

* [<mark style="color:purple;">**LlamaIndex**</mark>](https://www.llamaindex.ai)
* [<mark style="color:purple;">**LangChain**</mark>](https://www.langchain.com/)
* [<mark style="color:purple;">**OpenRouter**</mark>](https://openrouter.ai/)



***

## Overview

1. [Install dependencies](integrations.md#id-1.-install-dependencies)
2. [Set up **ColiVara** RAG Client and Process your documents](integrations.md#id-2.-set-up-colivara-rag-client-and-process-your-documents)
3. [Create the Context](integrations.md#id-3.-create-the-context)
4. [Create a Client with your integrated LLM Framework provider of choice](integrations.md#create-a-client-with-your-integrated-llm-framework)
5. [Run your RAG query](integrations.md#id-5.-run-your-rag-query-and-generate-the-answer)

***

## 1. Install Dependencies

<details>

<summary>LlamaIndex</summary>

```bash
pip install colivara-py -q -U
pip install llama-index-llms-openai -q 
pip install llama-index-multi-modal-llms-openai -q -U
pip install llama-index-core -q -U
pip install --force-reinstall llama-index==0.11.22
```

</details>

<details>

<summary>Langchain</summary>

```bash
pip install colivara-py --q
pip install langchain --q
pip install langchain-core --q
pip install langchain-openai --q
```

</details>

<details>

<summary>OpenRouter</summary>

```bash
pip install colivara-py -q -U
```

</details>

***

## 2. Set up **ColiVara** RAG client and Process your documents

{% stepper %}
{% step %}
### Initialize your Colivara RAG client

```python
import base64 # for converting docs binaries to base64
from pathlib import Path 
import os 
import requests 

from colivara_py import ColiVara 

rag_client = ColiVara(
    base_url="https://api.colivara.com", 
    api_key=YOUR_COLIVARA_KEY
)
```
{% endstep %}

{% step %}
### (Optional) Download your documents

You can skip this step if your documents are already downloaded into a folder. Here we are downloading documents using their URLs, and save them in the `docs/`folder

```python
# List of filenames and their URLS
files = [
    {
        "url": "https://github.com/tjmlabs/colivara-demo/raw/main/docs/Work-From-Home%20Guidance.pdf",
        "filename": "docs/Work-From-Home-Guidance.pdf",
    },
    {
        "url": "https://github.com/tjmlabs/colivara-demo/raw/main/docs/StaffVendorPolicy-Jan2019.pdf",
        "filename": "docs/StaffVendorPolicy-Jan2019.pdf",
    },
]

# Downloading the files into the docs/ folder
def download_file(url, local_filename):
    response = requests.get(url)

    if response.status_code == 200:
        os.makedirs("docs", exist_ok=True)

        with open(local_filename, "wb") as f:
            f.write(response.content)
        print(f"Successfully downloaded: {local_filename}")
    else:
        print(f"Failed to download: {url}")

for file in files:
    download_file(file["url"], file["filename"])
```
{% endstep %}

{% step %}
### Upload and Process the documents

```python
def sync_documents():
    documents_dir = Path("docs")
    files = list(documents_dir.glob("**/*"))

    for file in files:
        with open(file, "rb") as f:
            file_content = f.read()
            encoded_content = base64.b64encode(file_content).decode("utf-8")
            rag_client.upsert_document(
                name=file.name,
                document_base64=encoded_content,
                collection_name="openrouter-demo",
                wait=True,
            )
            print(f"Upserted: {file.name}")

sync_documents()
```
{% endstep %}
{% endstepper %}

***

## 3. Create the Context

{% hint style="info" %}
For simplicity, we are skipping _Query transformation_ step. In RAG, query transformation converts user queries into the format the model needs. Instead, we are using the query directly.&#x20;
{% endhint %}

```python
def get_context(query):
    results = rag_client.search(query=query, collection_name="openrouter-demo", top_k=3)
    results = results.results

    context = []
    for result in results:
        base64 = result.img_base64
        if "data:image" not in base64:
            base64 = f"data:image/png;base64,{base64}"
        context.append(base64)
    return context
    
query = "What is the work from home policy?"
context = get_context(query)
```

***

## 4. Create a Client with your integrated LLM Framework

<details>

<summary>Llamaindex</summary>

```python
from llama_index.multi_modal_llms.openai import OpenAIMultiModal
from llama_index.core.schema import ImageNode

openai_api_key = YOUR_OPENAI_API_KEY
gpt_4o = OpenAIMultiModal(model="gpt-4o", max_new_tokens=500, api_key=openai_api_key, temperature=0)
```

</details>

<details>

<summary>Langchain</summary>

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o", temperature=0)
```

</details>

<details>

<summary>OpenRouter</summary>

No need to create a Client as we will be calling OpenRouter via an HTTP request

</details>

***

## 5. Run your RAG query and Generate the answer

<details>

<summary>LlamaIndex</summary>

```python
complete_response = gpt_4o.complete(query, context)
print(complete_response)
```

</details>

<details>

<summary>Langchain</summary>

```python
prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a helpful assistant that answers questions based on the provided images/docs.",
        ),
        (
            "user",
            [
                {
                    "type": "text",
                    "text": "Here are some images for context:",
                },
                *[ 
                    {
                        "type": "image_url",
                        "image_url": {"url": image_data["base64"]},
                    }
                    for image_data in context
                ], 
                {"type": "text", "text": "Now, please answer the following question:"},
                {
                    "type": "text",
                    "text": "{query}",
                },
            ],
        ),
    ]
)

chain = prompt | model

response = chain.invoke({"query": query})
print(response.content)
```

</details>

<details>

<summary>OpenRouter</summary>

```python
url = "https://openrouter.ai/api/v1/chat/completions"

headers = {
    "Authorization": f"Bearer {os.environ['OPEN_ROUTER_API_KEY']}",
    "Content-Type": "application/json",
}


payload = {
    "model": "openai/gpt-4o",
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant that answers questions based on the provided images/docs.",
        },
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "Here are some images for context:",
                },
                *[
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": path,
                        },
                    }
                    for path in context
                ],
                {"type": "text", "text": query},
            ],
        },
    ],
    "max_tokens": 500,
    "temperature": 0.0,
}

response = requests.post(url, headers=headers, json=payload)
print(response.json()["choices"][0]["message"]["content"])
```

</details>
