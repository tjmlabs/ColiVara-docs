---
icon: pen-to-square
---

# Retrieval Augmented Generation (RAG)

The common problem ColiVara solves is RAG over visually rich documents where text extraction pipelines fail or presents incomplete data. For example, heavy tabular data with many charts such as policies for a medical facility.&#x20;

You can find the complete code for a complex demo below. This guide is a simplified version of this demo that focuses on the retrieval portion.

{% embed url="https://github.com/tjmlabs/colpali-demo" %}

### Prepare your documents

The first step in building a RAG pipeline is to prepare your documents. That could be a directory on your local computer, a S3 bucket, a google drive. Anything you can think of will work with Colivara. For the purposes of this guide - we will use a local directory with some documents in them.

```bash
mkdir colivara-simple-demo
cd colivara-simple-demo
mkdir docs
```

Then download the documents from Github. For the purposes of this demo, we will download the smallest 2 files. But, feel free to try with your own documents or all the documents in our demo repository.

```bash
curl -L -o docs/Work-From-Home-Guidance.pdf https://github.com/tjmlabs/colivara-demo/raw/main/docs/Work-From-Home%20Guidance.pdf
curl -L -o docs/StaffVendorPolicy-Jan2019.pdf https://github.com/tjmlabs/colivara-demo/raw/main/docs/StaffVendorPolicy-Jan2019.pdf
```

### Prepare your environment

Next, we will prepare our Python environment and install the Colivara Python SDK&#x20;

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install colivara-py
touch rag.py
```

### Sync your documents

In `rag.py`  - we want a function that allows us to sync our documents to the Colivara server. So, we can just call this as our documents change or updated. Colivara logic automatically updates or inserts new documents depending on what changed.&#x20;

```python
from colivara import Colivara
from pathlib import Path
import base64
from time import sleep

# set base_url and api_key
rag_client = Colivara(
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
            rag_client.upsert_document(name=file.name, document_base64=encoded_content, collection_name="demo collection", wait=True)
            sleep(0.5)
        
```

### Transform your query

Next - we we want to to transform a user messages or questions into an appropriate RAG question. In retrieval augmented generation - user and AI take turns in a conversation. In each turn, we want to get a factual context - that the AI can use in providing the answer.

We will need an LLM to help us with this transformation. For this guide, we will use `gpt-4o` but lighter models are also effective.&#x20;

```bash
pip install openai
```

Here is the code for the transformation.

```python
#rag.py
from openai import OpenAI

llm_client = OpenAI(api_key="your-openai-api-key")

def transform_query(messages=[{"role": "user", "content": "What is the work from home policy?"}]):
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

```

### Search

Finally, with our document and query prepared - we are ready to run our RAG pipeline with ColiVara.

```python
#rag.py

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
```

### Answer

With your context in hand - now you pass this to an LLM with multi-modal/vision capabilities and get a factually grounded answer.

```python
# rag.py
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

```

And with that - your RAG pipeline is complete. You sync your documents whenever you have an update. And&#x20;

### Example usage&#x20;

To see it in action - we will accept a user question via the terminal and print the response.&#x20;

```bash
touch main.py
```

In `main.py` - we will do a simple CLI program.

```python
import argparse
from .rag import sync_documents, draft_response

def main():
    parser = argparse.ArgumentParser(description="Process a query with optional sync flag.")
    parser.add_argument('query', type=str, help='The query to process')
    parser.add_argument('--sync', action='store_true', help='Optional sync flag')
    
    args = parser.parse_args()

    if args.sync:
        # syncs the documents first
        sync_documents()
    response = draft_response([args.query])
    print(response)

if __name__ == "__main__":
    main()
```





