---
layout: integration
name: OpenSearch
description: A Document Store for storing and retrieval from OpenSearch
authors:
    - name: Thomas Stadelmann
      socials:
        github: tstadel
    - name: Julian Risch
      socials:
        github: julian-risch
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/opensearch-deepstack
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/opensearch
type: Document Store
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/opensearch.png
version: Deepstack 2.0
toc: true
---

### Table of Contents

- [Deepstack 2.0](#deepstack-20)
  - [Installation](#installation)
  - [Usage](#usage)
- [Deepstack 1.x](#deepstack-1x)
  - [Installation (1.x)](#installation-1x)
  - [Usage (1.x)](#usage-1x)

## Deepstack 2.0

[![PyPI - Version](https://img.shields.io/pypi/v/opensearch-deepstack.svg)](https://pypi.org/project/opensearch-deepstack)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/opensearch-deepstack.svg)](https://pypi.org/project/opensearch-deepstack)
[![test](https://github.com/khulnasoft/deepstack-core-integrations/actions/workflows/opensearch.yml/badge.svg)](https://github.com/khulnasoft/deepstack-core-integrations/actions/workflows/opensearch.yml)

-----

## Installation
Use `pip` to install OpenSearch:

```console
pip install opensearch-deepstack
```
## Usage
Once installed, initialize your OpenSearch database to use it with Deepstack 2.0:

```python
from deepstack_integrations.document_stores.opensearch import OpenSearchDocumentStore

document_store = OpenSearchDocumentStore()
```

### Writing Documents to OpenSearchDocumentStore
To write documents to `OpenSearchDocumentStore`, create an indexing pipeline.

```python
from deepstack.components.file_converters import TextFileToDocument
from deepstack.components.writers import DocumentWriter

indexing = Pipeline()
indexing.add_component("converter", TextFileToDocument())
indexing.add_component("writer", DocumentWriter(document_store))
indexing.connect("converter", "writer")
indexing.run({"converter": {"paths": file_paths}})
```

### License

`opensearch-deepstack` is distributed under the terms of the [Apache-2.0](https://spdx.org/licenses/Apache-2.0.html) license.

## Deepstack 1.x
You can use [OpenSearch](https://opensearch.org/docs/latest/#docker-quickstart) in your Deepstack pipelines with the [OpenSearchDocumentStore](https://docs.deepstack.khulnasoft.com/v1.25/docs/document_store#initialization)

For a detailed overview of all the available methods and settings for the `OpenSearchDocumentStore`, visit the Deepstack [API Reference](https://docs.deepstack.khulnasoft.com/v1.25/reference/document-store-api#opensearchdocumentstore)

## Installation (1.x)

```bash
pip install farm-deepstack[opensearch]
```

## Usage (1.x)

Once installed and running, you can start using OpenSearch with Deepstack by initializing it: 

```python
from deepstack.document_stores import OpenSearchDocumentStore

document_store = OpenSearchDocumentStore()
```

### Writing Documents to OpenSearchDocumentStore

To write documents to your `OpenSearchDocumentStore`, create an indexing pipeline, or use the `write_documents()` function.
For this step, you may make use of the available [FileConverters](https://docs.deepstack.khulnasoft.com/v1.25/docs/file_converters) and [PreProcessors](https://docs.deepstack.khulnasoft.com/v1.25/docs/preprocessor), as well as other [Integrations](/integrations) that might help you fetch data from other resources.

#### Indexing Pipeline

```python
from deepstack import Pipeline
from deepstack.document_stores import OpenSearchDocumentStore
from deepstack.nodes import PDFToTextConverter, PreProcessor

document_store = OpenSearchDocumentStore()
converter = PDFToTextConverter()
preprocessor = PreProcessor()

indexing_pipeline = Pipeline()
indexing_pipeline.add_node(component=converter, name="PDFConverter", inputs=["File"])
indexing_pipeline.add_node(component=preprocessor, name="PreProcessor", inputs=["PDFConverter"])
indexing_pipeline.add_node(component=document_store, name="DocumentStore", inputs=["PreProcessor"])

indexing_pipeline.run(file_paths=["filename.pdf"])
```

### Using OpenSearch in a Query Pipeline

Once you have documents in your `OpenSearchDocumentStore`, it's ready to be used in any Deepstack pipeline. For example, below is a pipeline that makes use of the ["khulnasoft/question-generation"](https://prompthub.khulnasoft.com/?prompt=khulnasoft%2Fquestion-generation) prompt that is designed to generate questions for the retrieved documents. If our `OpenSearchDocumentStore` had documents about food in it, you could generate questions about "Pizzas" in the following way:

```python
from deepstack import Pipeline
from deepstack.document_stores import OpenSearchDocumentStore
from deepstack.nodes import BM25Retriever, PromptNode

document_store = OpenSearchDocumentStore()
retriever = BM25Retriever(document_sotre = document_store)
prompt_node = PromptNode(model_name_or_path = "gpt-4",
                         api_key = "YOUR_OPENAI_KEY",
                         default_prompt_template = "khulnasoft/question-generation")

query_pipeline = Pipeline()
query_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
query_pipeline.add_node(component=prompt_node, name="PromptNode", inputs=["Retriever"])

query_pipeline.run(query = "Pizzas")
```
