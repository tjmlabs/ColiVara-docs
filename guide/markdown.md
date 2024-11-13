---
icon: file-chart-column
---

# Data Extraction

## Retrieval-Augmented Generation (RAG) vs Data Extraction

While similar, RAG and Data Extraction are fundamentally different in their use cases and desired goals.&#x20;

**RAG** is combines _information retrieval_ with _generative AI_ to create responses based on relevant context. RAG is ideal for answering specific questions or generating **context-aware responses** from a large corpus, like handling FAQs, policy summaries, or dynamic information.

Let’s consider a practical example using an employee policy document that contains information about work-from-home guidelines, employee benefits, and leave policies.&#x20;

For a RAG-based system, the user might ask a question, such as:

> _How much paid time off does an employee with 5 years of service receive?_

The system might then respond with an answer:&#x20;

> _Employees with 5 years of service are eligible for 20 days of paid time off per year. Leave requests should be submitted at least two weeks in advance, and PTO is restricted during company-wide blackout periods_

{% hint style="info" %}
**RAG** provides <mark style="color:purple;">**contextual responses**</mark> in natural language, suitable for conversational or FAQ-type applications.
{% endhint %}

**Data extraction** aims to extract **structured data** information from documents (e.g., names, dates, specific values) _without generating responses_. The goal is to capture and structure data for easier access and analysis.

Using the previous example when querying from an employee policy document containing work-from-home policy, an user might query the document for "PTO detail". After which, a data extraction system might give back a JSON data as follow:&#x20;

```json
"pto_details" : {
    "years_of_service": 5,
    "pto_amount": "20 days", 
    "notice_requirement": "2 weeks", 
    "black_out_applied?": true
}
```

{% hint style="info" %}
**Data Extraction** gives <mark style="color:purple;">**structured data**</mark> outputs, ideal for populating databases, automating forms, or generating reports where individual fields are needed without additional narrative context.
{% endhint %}

***

## Data Extraction using ColiVara

### Step 1: Client Setup

Install the `colivara-py` SDK library&#x20;

If using Jupyter Notebook:

```python
!pip install --no-cache-dir --upgrade colivara_py
```

If using the command shell

```bash
pip install --no-cache-dir --upgrade colivara_py
```

### Step 2: Prepare Documents



{% stepper %}
{% step %}
**Download Files**: Download the desired files to your machine. This code specifically will download them into your `docs/` folder

```python
import requests
import os

def download_file(url, local_filename):
    response = requests.get(url)
    
    if response.status_code == 200:
        os.makedirs('docs', exist_ok=True)
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
{% endstep %}

{% step %}
**Upload Files to be procesed**:  Sync documents to the ColiVara server. The server will process these files to generate the necessary embeddings

```python
from colivara_py import ColiVara
from pathlib import Path
import base64

rag_client = ColiVara(
  base_url="https://api.colivara.com", 
  api_key="your-api-key"
)

new_collection = rag_client.create_collection(
    name="my_collection", 
    metadata={"description": "A sample collection"}
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
                collection_name="my_collection", 
                wait=True
            )
            print(f"Upserted: {file.name}")

sync_documents()
```



If using the a Python code editor (such as VSCode):
{% endstep %}

{% step %}
**(Optional) Verify that documents have been processed by ColiVara**:  Documents have been convert into "screenshots" to generate embeddings

If using Jupyter Notebook:

```python
from IPython.display import display, HTML, Image

def display_image_from_document(document):
    pages = doc.pages
    for page in pages:
        base64_img = page.img_base64
        if base64_img.startswith('data:image'):
            base64_img = base64_img.split(',')[1]
        image = Image(data=base64.b64decode(base64_img))
        display(image)

document_name = "Work-From-Home-Guidance.pdf"
doc = client.get_document(document_name=document_name, collection_name="all", expand="pages")

display_image_from_document(doc)
```

If using a code editor such as VSCode:

```python
from IPython.display import Image
from io import BytesIO
from PIL import Image

def display_image_from_document(document):
    pages = doc.pages
    for page in pages:
        base64_img = page.img_base64
        if base64_img.startswith('data:image'):
            base64_img = base64_img.split(',')[1]
        image_data = base64.b64decode(base64_img)
        image = Image.open(BytesIO(image_data))
        image.show()

document_name = "Work-From-Home-Guidance.pdf"
doc = rag_client.get_document(
    document_name=document_name, 
    collection_name="my_collection", 
    expand="pages")

display_image_from_document(doc)
```
{% endstep %}
{% endstepper %}

### Step 3: Extract data



{% stepper %}
{% step %}
**Install the LLM of choice**. Here, we are using OpenAI's GPT model&#x20;

If using Jupyter Notebook:

```bash
!pip install openai
```

If using the command shell:

```bash
!pip install openai
```
{% endstep %}

{% step %}
**Extract the JSON data**

```python
import json
from openai import OpenAI

llm_client = OpenAI(api_key="your-api-key")

# if your document is too big (20+ pages), or you want to extract data across multiple documents, 
# consider a pipeline where you search first and get top 3 pages, and then do this step.
# here since our document is small - we are passing the whole document at once.
def extract_data(data_to_extract, colivara_document):
    string_json = json.dumps(data_to_extract)
    content = [ 
                {
                "type": "text", 
                "text": f"""Use the following images as a reference to extract structured data with the following user example as a guide: {string_json}.\n
                If information is not available, keep the value blank.
                """,
                }
            ]
    pages = colivara_document.pages
    for page in pages:
        base64 = f"data:image/png;base64,{page.img_base64}"
        content.append(
            {
                    "type": "image_url",
                    "image_url": {"url": base64}
            }        
            )
    messages = [
                    {"role": "system", "content": "Our goal is to find out when a policy was issued to remind our users to review it at regular intervals. Always respond in JSON"},
                    {"role": "user", "content": content}
    ]
    completion = llm_client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        response_format= { "type": "json_object" },
        temperature=0.25,
        seed=123
        )
    return completion.choices[0].message.content


document_name = "Work-From-Home-Guidance.pdf"
doc = rag_client.get_document(
    document_name=document_name, 
    collection_name="my_collection", 
    expand="pages")
    
data_to_extract = {"year_issued": 2014, "month_issued": 3}

data = extract_data(data_to_extract, doc)
print(data)
```
{% endstep %}
{% endstepper %}

The result output data should be:&#x20;

> ```json
> {
>     "year_issued": 2020, 
>     "month_issued": 3
> }
> ```



