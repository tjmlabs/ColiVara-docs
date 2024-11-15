---
icon: hand-pointer
---

# Python SDK

## Using the SDK

The SDK can be installed:

```bash
pip install colivara_py
```

Then imported into your project:

```python
from colivara_py import ColiVara
```

## Essential Methods

<details>

<summary><code>search:</code> Sends a query to the server.</summary>

The query species which collection to search within, the number of top results to return, and optional filters to refine the search. It returns the most relevant results based on the given parameters.

#### **Parameters**

* `query` (`str`): The search query string. This value cannot be null or empty.
* `collection_name` (`str`, optional): The name of the collection to search within. Defaults to `"all"`, which searches across all collections.
* `top_k` (`int`, optional): Specifies the maximum number of results to return. Defaults to `3`.
* `query_filter` (`Dict[str, Any]`, optional): An optional filter to narrow down search results. Read more about[ **Advance Filtering**](filtering.md) options here.&#x20;
  * `on` (`str`): Specifies whether the filter applies to `"document"` or `"collection"`.
  * `key` (`str`, `List[str]]`): A single key or a list of keys to match.
  * `value` (`str`, `int`, `float`, `bool`, `List[str, int, float, bool]`): The value(s) to match for the specified key(s).
  * `lookup` (`str`): Defines the matching condition. Options include `"key_lookup"`, `"contains"`, `"contained_by"`, `"has_key"`, `"has_keys"`, and `"has_any_keys"`.&#x20;

#### Returns

* `QueryOut`: An object that includes the search query and a list of relevant pages based on the specified parameters.

#### Exceptions

* `ValueError`: Raised if the `query` is empty, the specified `collection_name` does not exist, or the `query_filter` is improperly configured.

#### Example

```python
# searches for pages within the "my_collection" collection that contains
# content related to "Gemini" and are categorized under "AI".
# returns 5 tops results
results = client.search(
    query="Gemini",
    collection_name=my_collection,
    top_k=5,
    query_filter={
      "on": "document",
      "key": "category",
      "value": "AI",
      "lookup": "contains"
    }
)
```

</details>

<details>

<summary><code>filter:</code> Retrieves documents or collections that match specific criteria based</summary>

This allows you to apply flexible criteria to retrieve specific documents or collections. It supports advanced lookups like filtering by key presence or matching values. This method is useful for narrowing down results by metadata or other attributes.

#### **Parameters**

* `query_filter` (`Dict[str, Any]`): A dictionary specifying the filter criteria. The dictionary must contain:
  * `on` (`str`): Specifies the target, either `"document"` or `"collection"`.
  * `lookup` (`str`): The type of lookup.  Options include `"key_lookup"`, `"contains"`, `"contained_by"`, `"has_key"`, `"has_keys"`, and `"has_any_keys"`.  Read more about[ **Advance Filtering**](filtering.md) options here.&#x20;
  * `key` (`str` or `List[str]`): The key(s) to filter by.
  * `value` (`str, int, float, bool,` optional): The value(s) the key(s) should match (optional, depending on the filter option used).
* `expand` (`str`, optional):&#x20;
  * Currently, only `"pages"` is supported, which when used to filter documnets will include their pages in the returned value.&#x20;

#### Returns

* `List[DocumentOut]`, or `List[CollectionOut]`: A list of objects representing either documents or collections that match the filter criteria.

#### Exceptions

* `ValueError`: Raised if the filter is invalid.

#### Example

```python
# Filters documents with category containing "AI" and 
# includes document pages in the output data
results = client.filter(
    query_filter= {
        "on": "document",
        "key": "category",
        "value": "AI",
        "lookup": "contains" }, 
    expand="pages",
)
```

</details>

<details>

<summary><code>create_collection:</code>Creates a new collection </summary>

&#x20;A collection is a storage within ColiVara server that your documents could be uploaded into for search purposes.  A collection is created with a specified name and optional metadata.

#### Parameters

* `name` (`str`): The name of the collection to be created. This value cannot be null or empty.
* `metadata` (`Dict[str, Any]`, optional): A dictionary containing metadata for the new collection. This is optional and can include any relevant key-value pairs.

#### Returns

* `CollectionOut`: An object representing the newly created collection, including details such as its name and metadata.

#### Exceptions

* `Exception`with the message `Conflict error`: Raised if there’s a conflict, such as when a collection with the same name already exists.

#### Example

```python
# searches for pages within the "my_collection" collection that contains
# content related to "Gemini" and are categorized under "AI".
# returns 5 tops results
results = client.search(
    query="Gemini",
    collection_name=my_collection,
    top_k=5,
    query_filter={
      "on": "document",
      "key": "category",
      "value": "AI",
      "lookup": "contains"
    }
)
```

</details>

<details>

<summary><code>upsert:</code> Adds or updates a document.</summary>

This operation also supports adding metadata and providing document content through a URL, a base64-encoded string, or a file path.

#### **Parameters**

* `name` (`str`): The name of the document to be added or updated. This value cannot be null.
* `metadata` (`Dict[str, Any]`, optional): Additional metadata for the document, such as tags or descriptive information.
* `collection_name` (`str`, optional): The collection to add the document to. Defaults to `"default collection"`.
* `document_url` (`str`, optional): The URL of the document if it’s available online.
* `document_base64` (`str`, optional): The document content encoded in base64.
* `document_path` (`str`, optional): The file path to the document, which will be read and converted to base64.
* `wait` (`bool`, optional): If `True`, the method will be synchronous, which mean it will wait for the document processing to complete before returning, making . The default for this value is `False`, making asynchronous the default behavior .&#x20;

#### Returns

* `DocumentOut`: An object containing details of the created or updated document. This is returned for synchronous processing
* `GenericMessage`: A message object returned if the document is accepted for processing .This is returned for asynchronous processing

#### Exceptions

* `ValueError`: Raised if no valid document source (URL, base64, or file path) is provided, or if there is an issue with the file path.
* `FileNotFoundError`: Raised if the specified file path does not exist.
* `PermissionError`: Raised if there is no read permission for the specified file.

#### Example

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
)
```

</details>

<details>

<summary><code>create_embedding</code>: Generates embeddings on the processed images or text</summary>

Embeddings are vector representations of the data. This method can generate vectors on either image data, or on a text query. After generation, these vectors comparison is processed to generate query result.

#### Parameters

* `input_data` (`str`, `List[str]`): A single string or a list of strings representing the data for which embeddings need to be generated. This could be text (for a query) or paths to image files.
* `task` (`str,`, optional): Specifies the type of embedding task.&#x20;
  * Acceptable values are `"query"` (default) for text queries or `"image"` for images.&#x20;

#### Returns

* `EmbeddingsOut`: An object containing the generated embeddings, along with information about the model and usage data.

#### Exceptions

* `ValueError`: Raised if an invalid task type is provided (i.e., not `"query"` or `"image"`) or if the input data is improperly formatted.

#### Example

```python
# Create embeddings for a text query
text_embeddings = client.create_embedding(
  input_data="What is artificial intelligence?", 
  task="query")

# Create embeddings for a list of image paths
image_paths = 
image_embeddings = client.create_embedding(
  input_data=["image1.jpg", "image2.jpg"], 
  task="image")
```

</details>

## Collection Manipulation Methods

<details>

<summary><code>get_collection</code>: Retrieves a collection</summary>

#### Parameters

* `collection_name` (`str`): The name of the collection to retrieve. This value cannot be null or empty and must match an existing collection.

#### Returns

* `CollectionOut`: An object representing the retrieved collection, including details such as its name and metadata.

#### Exceptions

* `Exception`with message `Collection not found` : Raised if the specified collection does not exist.&#x20;

#### Example

```python
# retrieves the "AI_Research_Papers" collection
collection = client.get_collection(collection_name="AI_Research_Papers")
```

</details>

<details>

<summary><code>list_collections:</code>Retrieves a list of all collections available to the user</summary>

#### **Parameters**

#### Returns

* `List[CollectionOut]`: A list of `CollectionOut` objects, each representing a collection with details such as name and metadata.

#### Exceptions

* `ValueError`: Raised if the server response format is unexpected (e.g., not a list).

#### Example

```python
# Retrieve the list of all collections
collections = client.list_collections()
```

</details>

<details>

<summary><code>partial_update_collection</code>: Partially updates a collection's metadata.</summary>

Only the fields provided in the parameters will be updated. Metadata already exists for the collection but was not provided will not be removed.

#### Parameters

* `collection_name` (`str`): The name of the collection to update. This value cannot be null.
* `name` (`str`, optional): A new name for the collection, if you wish to rename it.
* `metadata` (`Dict[str, Any]`, optional): New metadata for the collection. This replaces or adds to the existing metadata.

#### Returns

* `CollectionOut`: An object representing the updated collection, including details such as its name and metadata.

#### Exceptions

* `Exception`with message `Collection not found` : Raised if the specified collection does not exist.&#x20;

#### Example

```python
# updates the "AI_Projects" collection name to "AI_Research_Projects" and adds more metadata
updated_collection = client.partial_update_collection(
    collection_name="AI_Projects",
    name="AI_Research_Projects",
    metadata={
      "updated_by": "admin", 
      "status": "active"}
)
```

</details>

<details>

<summary><code>delete_collection:</code>Removes a collection</summary>

The collection will be deleted from ColiVara server. This action is permanent and final.

#### **Parameters**

* `collection_name` (`str`): The name of the collection to delete. This must match an existing collection.

#### Returns

#### Exceptions

* `Exception`with message `Collection not found` : Raised if the specified collection does not exist.&#x20;

#### Example

```python
# Delete the specified collection
client.delete_collection(collection_name="Obsolete_Collection")
```

</details>

## Document Manipulation Methods

<details>

<summary><code>get_document</code>: Retrieves a document</summary>

#### Parameters

* `document_name` (`str`): The name of the document to retrieve.
* `collection_name` (`str`, optional): The name of the collection containing the document. Defaults to `"default collection"`.
* `expand` (`str`, optional): if the value is `"pages"`,the method will include the document’s pages.

#### Returns

* `DocumentOut`: An object containing details of the retrieved document, including details such as its name and metadata.

#### Exceptions

* `ValueError` with message `Document not found` : Raised if the document is not found because the document name does not exist

#### Example

```python
# retrieves a document with its pages included
document = client.get_document(
    document_name="Research_Paper",
    collection_name="AI_Research",
    expand=["pages"]
)
```

</details>

<details>

<summary><code>list_documents:</code>Retrieves a list of all documents in a collection or in all collections available to the user</summary>

#### **Parameters**

* `collection_name` (`str`, optional): The name of the collection to fetch documents from. Defaults to `"default collection"`.&#x20;
  * Use `"all"` to fetch documents from all collections.

<!---->

* `expand` (`str`, optional): if the value is `"pages"`,the method will include all the documents’ pages.

#### Returns

* `List[DocumentOut]`: A list of `DocumentOut` objects, each representing a document with its details.

#### Exceptions

#### Example

```python
# Retrieves all documents in the "AI_Research" collection, including pages for each document
documents = client.list_documents(
  collection_name="AI_Research", 
  expand="pages")
```

</details>

<details>

<summary><code>partial_update_document</code>: Partially updates a document</summary>

This method can update either or both the document's content or metadata. Only the fields provided in the parameters will be updated. Metadata already exists for the document but was not provided will not be removed.

#### Parameters

* `document_name` (`str`): The name of the document to be updated.
* `name` (`str`, optional): A new name for the document, if renaming.
* `metadata` (`Dict[str, Any]`, optional): Updated metadata for the document.
* `collection_name` (`str`, optional): The new collection name if you wish to move the document to a different collection.
* `document_url` (`str`, optional): A new URL for the document content, if changing.
* `document_base64` (`str`, optional): A new base64-encoded string of the document content, if changing.

#### Returns

* `DocumentOut`: An object containing details of the retrieved document, including details such as its name and metadata.

#### Exceptions

* `ValueError`  with message `Update failed`: Raised if the document is not found or if there is an issue with the update

#### Example

```python
# Update a document's name from "Research_Paper" to "Updated_Research_Paper"
# Also updates its metadata, and URL
updated_document = client.partial_update_document(
    document_name="Research_Paper",
    name="Updated_Research_Paper",
    metadata={
      "author": "Dr. AI Researcher", 
      "year": "2024"
    },
    document_url="https://example.com/updated_paper.pdf"
)
```

</details>

<details>

<summary><code>delete_document:</code>Removes a document </summary>

The document to be deleted can be identified from a specific collection, or from all collections. The document will be deleted from ColiVara server. This action is permanent and final.

#### **Parameters**

* `document_name` (`str`): The name of the document to delete.
* `collection_name` (`str`, optional): The name of the collection containing the document.&#x20;
  * Defaults to `"default collection"`.&#x20;
  * Use `"all"` to access documents across all collections belonging to the user.

#### Returns

#### Exceptions

* `ValueError`: Raised if the document does not exist or if there is an issue with the deletion&#x20;

#### Example

```python
# Deletes the "Old_Report" document from the "Archived_Documents" collection
client.delete_document(
  document_name="Old_Report", 
  collection_name="Archived_Documents"
)
```

</details>

## Other Methods&#x20;

<details>

<summary><code>file_to_base64</code>: Converts file content to a base64-encoded string</summary>

This method is useful to update a document's content - if the document is short or has only 1 page - by first converting the file content into a base64 string, then submitted this string as a parameter.&#x20;

#### **Parameters**

* `file_path` (`str`): The path to the file you want to convert.

#### Returns

* `str`: A base64-encoded string representing the file's content

#### Exceptions

* `Exception`: Raised if there’s an error during the file reading or encoding process.

#### Example

```python
# Converts the contents of document.pdf to a base64 string.
base64_string = client.file_to_base64("/path/to/document.pdf")
```

</details>

<details>

<summary><code>file_to_imgbase64:</code>Convert a file into a list of base64 strings, each represents a page from the document</summary>

This method is useful to update a document's content if the document has multiple pages

#### **Parameters**

* `file_path` (`str`): The path to the file you want to convert.

#### Returns

* `List[FileOut]`: A list of `FileOut` objects, each containing a base64-encoded string of an image, and the page number within the document.

#### Exceptions

* `Exception`: Raised if there’s an error during the file reading or encoding process.

#### Example

```python
# Converts the contents of multi_page_document.pdf to a list of base64 string
base64_images = client.file_to_imgbase64("/path/to/multi_page_document.pdf")
```

</details>

