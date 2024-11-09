---
icon: pen-to-square
---

# Retrieval Augmented Generation (RAG)

The common problem ColiVara solves is RAG over visually rich documents where text extraction pipelines fail or presents incomplete data. For example, heavy tabular data with many charts such as policies for a medical facility.&#x20;

You can find the complete code for this demo below.&#x20;

{% embed url="https://github.com/tjmlabs/ColiVara-docs/blob/main/cookbook/RAG.ipynb" %}

## Overview

#### Preparations

You would need:&#x20;

* A **ColiVara API key**&#x20;
* An **LLM API key** of your choice&#x20;
* Appropriate documents for RAG queries

#### From a high-level viewpoint, the process goes as follow:&#x20;

1. Prepare your environment&#x20;
2. Prepare your documents
3. Sync (upload) documents to ColiVara server
4. Transform your search query&#x20;
5. Search processed document for relevant embeddings
6. Generate a factual, grounded response&#x20;

### 1. Prepare your environment

First, install the ColiVara Python SDK. If using Jupyter Notebook:

```bash
!pip install colivara_py
```

If using the command shell:

```bash
pip install colivara_py
```

### 2. Prepare your documents

This is the start of our RAG pipeline. We start by preparing the documents. That could be a directory on your local computer, a S3 bucket, a google drive. Anything you can think of will work with ColiVara. For the purposes of this guide - we will use a local directory with some documents in them.

If using Jupyter Notebook:

```bash
!pip install requests
```

If using the command shell:

```bash
pip install requests
```

Then download the documents from Github. For the purposes of this demo, we will download the smallest 2 files. But, feel free to try with your own documents or all the documents in our demo repository.

```python
import requests
import os

def download_file(url, local_filename):
    # Send a GET request to the URL
    response = requests.get(url)
    
    # Check if the request was successful
    if response.status_code == 200:
        # Ensure the 'docs' directory exists
        os.makedirs('docs', exist_ok=True)
        
        # Write the content to a local file
        with open(local_filename, 'wb') as f:
            f.write(response.content)
        print(f"Successfully downloaded: {local_filename}")
    else:
        print(f"Failed to download: {url}")

# URLs and local filenames
files = [
    {
        "url": "https://github.com/tjmlabs/colivara-demo/raw/main/docs/Work-From-Home%20Guidance.pdf",
        "filename": "docs/Work-From-Home-Guidance.pdf"
    },
    {
        "url": "https://github.com/tjmlabs/colivara-demo/raw/main/docs/StaffVendorPolicy-Jan2019.pdf",
        "filename": "docs/StaffVendorPolicy-Jan2019.pdf"
    }
]

# Download each file
for file in files:
    download_file(file["url"], file["filename"])
```



### 3. Sync your documents

We want to sync our documents to the ColiVara server. So, we can just call this as our documents change or updated. ColiVara logic automatically updates or inserts new documents depending on what changed. The `wait=True` parameter ensures this process is synchronous.&#x20;

```python
from colivara_py import ColiVara
from pathlib import Path
import base64


# set base_url and api_key
rag_client = ColiVara(
        base_url="https://api.colivara.com", api_key="your-api-key"
      )

def sync_documents():    
    # get all the documents under docs/ folder and upsert them to colivara
    documents_dir =  Path('docs')
    files = [f for f in documents_dir.glob('**/*') if f.is_file()]

    for file in files:
        with open(file, 'rb') as f:
            file_content = f.read()
            encoded_content = base64.b64encode(file_content).decode('utf-8')
            rag_client.upsert_document(
                name=file.name, 
                document_base64=encoded_content, 
                collection_name="demo collection", 
                wait=True
            )
            print(f"Upserted: {file.name}")

sync_documents()
        
```

### 4. Transform your query

Next - we we want to to transform a user messages or questions into an appropriate _RAG question_. In retrieval augmented generation - user and AI take turns in a conversation. In each turn, we want to get a factual context - that the AI can use in providing the answer.

We will need an LLM to help us with this transformation. For this guide, we will use gpt-4o but lighter models are also effective.

If using Jupyter Notebook:

```bash
!pip install openai
```

If using the command shell:

```bash
pip install openai
```

Here is the code for the transformation.

```python
from openai import OpenAI

llm_client = OpenAI(api_key="your-api-key")

def transform_query(messages):
    prompt = """ 
    You are given a conversation between a user and an assistant. We need to transform the last message from the user into a question appropriate for a RAG pipeline.
    Given the nature and flow of conversation. 

    Example #1:
    User: What is the capital of Brazil?
    Assistant: Brasília
    User: How about France?
    RAG Query: What is the capital of France?
    <reasoning> 
    Somewhat related query, however, if we simply use "how about france?" without any transformation, the RAG pipeline will not be able to provide a meaningful response.
    The transformation took the previous question (what the capital of Brazil?) as a strong hint about the user intention
    </reasoning>

    Example #2:
    User: What is the policy on working from home? 
    Assistant: <policy details>
    User: What is the side effects of Wegovy?
    RAG Query: What are the side effects of Wegovy?
    <reasoning>
    The user is asking for the side effects of Wegovy, the transformation is straightforward, we just need to slightly adjust. 
    The previous question was about a completely different topic, so it has no influence on the transformation.
    </reasoning>

    Example #3:
    User: What is the highest monetary value of a gift I can recieve from a client?
    Assistant: <policy details>
    User: Is there a time limit between gifts?
    RAG Query: What is the highest monetary value of a gift I can recieve from a client within a specific time frame?
    <reasoning>
    The user queries are very related and a continuation of the same question. He is asking for more details about the same topic.
    The transformation needs to take into account the previous question and the current one.
    </reasoning>

    Example #4:
    User: Hello!
    RAQ Query: Not applicable
    <reasoning>
    The user is simply greeting the assistant, there is no question to transform. This applies to any non-question message,
    </reasoning>

    Coversation:
    """
    for message in messages:
        prompt += f"{message['role']}: {message['content']}\n"

    response = llm_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "assistant", "content": prompt}],
        stream=False,
    )
    query = response.choices[0].message.content
    return query


messages = [{"role": "user", "content": "What is the work from home policy?"}]
transform_query(messages)
```



### 5. Search for context using the RAG pipeline

Finally, with our document and query prepared - we are ready to run our RAG pipeline with ColiVara.

```python
def run_rag_pipeline(query):
    # clean the query that came from the LLM
    if "not applicable" in query.lower():
        # don't need to run RAG or get a context
        return []
    if "rag query:" in query.lower():
        query = query.split("ry:")[1].strip()
    
    if "<reasoning>" in query:
        query = query.split("<reasoning>")[0].strip()

    print(f"This what we will send to the RAG pipeline: {query}")
    results = rag_client.search(query=query, collection_name="demo collection")
    results = results.results
    # example result: 
    """{
  "query": "string",
  "results": [
    {
      "collection_name": "string",
      "collection_id": 0,
      "collection_metadata": {},
      "document_name": "string",
      "document_id": 0,
      "document_metadata": {},
      "page_number": 0,
      "raw_score": 0,
      "normalized_score": 0,
      "img_base64": "string"
    }
    ]
    }"""

    context= []
    for result in results:
        document_title = result.document_name
        page_num = result.page_number
        base64 = result.img_base64
        # base64 doesn't have data: part so we need to add it
        if "data:image" not in base64:
            base64 = f"data:image/png;base64,{base64}"
        context.append(
            {
                "metadata": f"{document_title} - Page {page_num}",
                "base64": base64,
            }
        )
    return context

query = 'RAG Query: What is the work from home policy?'

context = run_rag_pipeline(query)
```

Let's peek what our context looks like:

<div>

<figure><img src="../.gitbook/assets/image_1.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../.gitbook/assets/image_3.png" alt=""><figcaption></figcaption></figure>

 

<figure><img src="../.gitbook/assets/image_2.png" alt=""><figcaption></figcaption></figure>

</div>

### Generate answer

With your context in hand - now you pass this to an LLM with multi-modal/vision capabilities and get a factually grounded answer.

```python
def draft_response(messages):
    query = transform_query(messages) 
    context = run_rag_pipeline(query)
    content = [
                {
                    "type": "text",
                    "text": f"""Use the following images as a reference to answer the following user questions: {query}. 
            """,
                },
                {
                    "type": "image_url",
                    "image_url": {"url": context[0]["base64"]},
                },
                {
                    "type": "image_url",
                    "image_url": {"url": context[1]["base64"]},
                },
                {
                    "type": "image_url",
                    "image_url": {"url": context[2]["base64"]},
                },
            ]
    last_message = messages[-1]
    # the last message is the user question, we enhance it with the RAG query
    last_message["content"] = content
    # pop the last message and append the new message
    messages.pop()
    messages.append(last_message)
    response = llm_client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        stream=False,
    )
    answer = response.choices[0].message.content
    return answer


messages = [
    {"role": "user", "content": "Can I work from home on Fridays?"}
]
answer = draft_response(messages)

print(answer)

"""
This what we will send to the RAG pipeline: What is the policy on working from home on Fridays?
Based on the documents provided, the policy on working from home at Mount Sinai is as follows:

1. **Eligibility and Requirements**: Remote work arrangements can be made available to employees in appropriate positions as determined by Mount Sinai. These arrangements require departmental approval and are subject to periodic review.

2. **Days of Remote Work**: The specific days for remote work, such as Fridays, would be set in the Remote Work Schedule and agreed upon with the manager, as there is a blank section to be filled regarding the days of the week.

3. **Compliance and Reporting**: Employees are required to comply with existing policies and notify their manager when they are working remotely.

4. **Frequency**: Employees engaged in full-time remote work must work at least one scheduled day at a Mount Sinai location in New York State annually.

5. **Approval and Review**: Remote work involving supervisory roles, such as Principal Investigators, requires approval from the Dean’s office.

There is no specific mention of Fridays, implying that any remote work schedule, including Fridays, needs to be established and agreed upon with management.
"""

```

And with that - your RAG pipeline is complete.
