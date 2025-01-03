---
icon: filters
---

# Advanced filtering

## You can use ColiVara to filter documents based on their metadata

**Colivara** offers fine-grained control over data retrieval by implementing various **Query Filters**, which boosts precision and lowers latency in response generation by filtering by document's metadata (or collection's metadata).

Queries running on document can be filtered by:

* **Direct Matching (**[**`key_lookup`**](filtering.md#id-1.-key_lookup)**,** [**`has_key`**](filtering.md#id-4.-has_key)**,** [**`has_keys`**](filtering.md#id-5.-has_keys)**,** [**`has_any_keys`**](filtering.md#id-6.-has_any_keys)**)**: Allows for exact matches, making it ideal for queries where the exact value matters (e.g., retrieving specific  data where exact metrics are critical).
* **Partial Containment (**[**`contains`**](filtering.md#id-2.-contains)**,** [**`contained_by`**](filtering.md#id-3.-contained_by)**)**: These enable partial matches, so if a user asks for a general topic (like "budget plans"), the model can pull relevant documents even if they do not perfectly match but contain related content.

Advanced filters can be used on metadata from a  single document, or from the entire collection

This leads to _higher relevance_, as users get nuanced responses tailored to both broad and specific information requests.&#x20;

***

## How to utilize query filter

To use Advanced Filtering on your documents or collection, pass in a `query_filter` parameter to the `search` function call.&#x20;

First - add relevant metadata to your documents (or collections) on document insertion.&#x20;

```python
from colivara_py import ColiVara

rag_client = ColiVara(
    # this is the default and can be omitted
    api_key=os.environ.get("COLIVARA_API_KEY"),
    # this is the default and can be omitted
    base_url="https://api.colivara.com"
)

# Upload a document to the default_collection
document = rag_client.upsert_document(
    name="sample_document",
    url="https://example.com/sample.pdf",
    metadata={"author": "John Doe"},
    # optional - specify a collection
    collection_name="user_1_collection", 
    # optional - wait for the document to index
    wait=True
)

```

Then - use pass in a `query_filter` parameter to the `search` function call.&#x20;

```python
query_filter = {
    "on": "document", # or collection
    "lookup": "contains", # or any of the supported lookup method
    "key": "auther",  
    "value": "John Doe"  
}

search_results = rag_client.search(
    query="my_query",
    query_filter=query_filter,
    collection_name="user_1_collection",
    top_k=3
)
```

***

## Available Filters

#### 1. `key_lookup`

* **Description**: Performs an exact match between a specified metadata key and a value. Useful when a precise key-value pair match is required.
* **Parameters**:
  * `key (str)`: Metadata key to match. Must be a single string.
  * `value(str, int, float, bool)`: Exact value for key. Required.
*   **Example Usage**:

    ```python
    # Retrieve documents where the author is exactly "John Smith"
    query_filter = {
        "lookup": "key_lookup",
        "on": "document",  # or "collection"
        "key": "author", 
        "value": "John Smith", 
    }
    ```

***

#### 2. `contains`

* **Description**: checks if a specified metadata key contains a particular substring or element. Useful for partial matches.
* **Parameters**:
  * `key (str)`: Metadata key in which to search for the substring.
  * `value (str)`: Substring to find within the key’s value. Required.
*   **Example Usage**:

    ```python
    # Find documents where tags include the substring "science"
    query_filter = {
        "lookup": "contains",
        "on": "document", 
        "key": "tag", 
        "value": "science", 
    }
    ```

***

#### 3. `contained_by`

* **Description**: verifies if a metadata value is fully contained within a provided structure, such as a string or list.  Useful for partial matches.
* **Parameters**:
  * `key (str)`: Metadata key to evaluate.
  * `value (str, List[str])`: A master string, or a list of string, of which the key's value is a substring. Required.
*   **Example Usage**:

    ```python
    # Find collections where keywords are contained within "machine learning research"
    query_filter = {
        "lookup": "contain_by",
        "on": "collection", 
        "key": "machine learning research", 
        "value": "science", 
    }
    ```

***

#### 4. `has_key`

* **Description**: checks if a specified key exists within the metadata, without verifying its value. Useful for filtering documents or collections based on the presence of a particular metadata field.
* **Parameters**:
  * `key (str)`: Metadata key to check for existence
  * Must not contain a `value` parameter
* **Validation**:
  * `key` must be a string.
  * Must not contain a `value` parameter
*   **Example Usage**:

    ```python
    # Retrieve documents where "publication_year" field exists
    query_filter = {
        "lookup": "has_key",
        "on": "document", 
        "key": "publication_year", 
    }
    ```

***

#### 5. `has_keys`

* **Description**: ensures that all specified keys in a list exist within the metadata. Useful for filtering documents or collections based on the presence of a list of metadata fields.
* **Parameters**:
  * `key (List[str])`: Must be a list of strings.
  * Must not contain a `value` parameter
*   **Example Usage**:

    ```python
    # Retrieve documents with both "author" and "date" fields
    query_filter = {
        "lookup": "has_key",
        "on": "document", 
        "key": ["author", "date"], 
    }
    ```

***

#### 6. `has_any_keys`

* **Description**: checks if at least one of the keys in a specified list exists in the metadata
* **Parameters**:
  * `key (List[str])`: Must be a list of strings.
  * Must not contain a `value` parameter
*   **Example Usage**:

    ```python
    # Retrieve documents with either "author" or "date" fields
    query_filter = {
        "lookup": "has_any_keys",
        "on": "document", 
        "key": ["author", "date"], 
    }
    ```

