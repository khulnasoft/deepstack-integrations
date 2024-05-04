---
layout: integration
name: Elasticsearch
description: Use an Elasticsearch database with Deepstack
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/elasticsearch-deepstack
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/opensearch
type: Document Store
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/elastic.png
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

The `ElasticsearchDocumentStore` is maintained in [deepstack-core-integrations](https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/elasticsearch) repo. It allows you to use [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html) as data storage for your Deepstack pipelines.

For a details on available methods, visit the [API Reference](https://docs.deepstack.khulnasoft.com/v1.25/reference/document-store-api#elasticsearchdocumentstore-1)

### Installation

To run an Elasticsearch instance locally, first follow the [installation](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html) and [start up](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html) guides. 

```bash
pip install elasticsearch-deepstack
```

### Usage

Once installed, you can start using your Elasticsearch database with Deepstack by initializing it: 

```python
from deepstack_integrations.document_stores.elasticsearch import ElasticsearchDocumentStore

document_store = ElasticsearchDocumentStore(hosts = "http://localhost:9200")
```

#### Writing Documents to ElasticsearchDocumentStore

To write documents to your `ElasticsearchDocumentStore`, create an indexing pipeline with a [DocumentWriter](https://docs.deepstack.khulnasoft.com/docs/documentwriter), or use the `write_documents()` function.
For this step, you can use the available [TextFileToDocument](https://docs.deepstack.khulnasoft.com/docs/textfiletodocument) and [DocumentSplitter](https://docs.deepstack.khulnasoft.com/docs/documentsplitter), as well as other [Integrations](/integrations) that might help you fetch data from other resources.

#### Indexing Pipeline

```python
from deepstack_integrations.document_stores.elasticsearch import ElasticsearchDocumentStore
from deepstack import Pipeline
from deepstack.components.embedders import SentenceTransformersDocumentEmbedder
from deepstack.components.converters import TextFileToDocument
from deepstack.components.preprocessors import DocumentSplitter
from deepstack.components.writers import DocumentWriter 

document_store = ElasticsearchDocumentStore(hosts = "http://localhost:9200")
converter = TextFileToDocument()
splitter = DocumentSplitter()
doc_embedder = SentenceTransformersDocumentEmbedder(model="sentence-transformers/multi-qa-mpnet-base-dot-v1")
writer = DocumentWriter(document_store)

indexing_pipeline = Pipeline()
indexing_pipeline.add_component("converter", converter)
indexing_pipeline.add_component("splitter", splitter)
indexing_pipeline.add_component("doc_embedder", doc_embedder)
indexing_pipeline.add_component("writer", writer)

indexing_pipeline.connect("converter", "splitter")
indexing_pipeline.connect("splitter", "doc_embedder")
indexing_pipeline.connect("doc_embedder", "writer")

indexing_pipeline.run({
    "converter":{"sources":["filename.txt"]}
    })
```

### Using Elasticsearch in a Query Pipeline

Once you have documents in your `ElasticsearchDocumentStore`, it's ready to be used with with [ElasticsearchEmbeddingRetriever](https://docs.deepstack.khulnasoft.com/docs/elasticsearchembeddingretriever) in the retrieval step of any Deepstack pipeline such as a Retrieval Augmented Generation (RAG) pipelines. Learn more about [Retrievers](https://docs.deepstack.khulnasoft.com/docs/retrievers) to make use of vector search within your LLM pipelines.

```python
from deepstack_integrations.document_stores.elasticsearch import ElasticsearchDocumentStore
from deepstack import Pipeline
from deepstack.components.embedders import SentenceTransformersTextEmbedder 
from deepstack_integrations.components.retrievers.elasticsearch import ElasticsearchEmbeddingRetriever

model = "sentence-transformers/multi-qa-mpnet-base-dot-v1"

document_store = ElasticsearchDocumentStore(hosts = "http://localhost:9200")


retriever = ElasticsearchEmbeddingRetriever(document_store=document_store)
text_embedder = SentenceTransformersTextEmbedder(model=model)

query_pipeline = Pipeline()
query_pipeline.add_component("text_embedder", text_embedder)
query_pipeline.add_component("retriever", retriever)
query_pipeline.connect("text_embedder.embedding", "retriever.query_embedding")

result = query_pipeline.run({"text_embedder": {"text": "historical places in Instanbul"}})

print(result)
```

## Deepstack 1.x

The `ElasticsearchDocumentStore` is maintained within the core Deepstack project. It allows you to use [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html) as data storage for your Deepstack pipelines.

For a details on available methods, visit the [API Reference](https://docs.deepstack.khulnasoft.com/v1.25/reference/document-store-api#elasticsearchdocumentstore-1)

### Installation (1.x)

To run an Elasticsearch instance locally, first follow the [installation](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html) and [start up](https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html) guides. 

```bash
pip install farm-deepstack[elasticsearch]
```

To install Elasticsearch 7, you can run `pip install farm-deepstac[elasticsearch7]`.

### Usage (1.x)

Once installed, you can start using your Elasticsearch database with Deepstack by initializing it: 

```python
from deepstack.document_stores import ElasticsearchDocumentStore

document_store = ElasticsearchDocumentStore(host = "localhost",
                                            port = 9200,
                                            embedding_dim = 768)
```

#### Writing Documents to ElasticsearchDocumentStore

To write documents to your `ElasticsearchDocumentStore`, create an indexing pipeline, or use the `write_documents()` function.
For this step, you may make use of the available [FileConverters](https://docs.deepstack.khulnasoft.com/v1.25/docs/file_converters) and [PreProcessors](https://docs.deepstack.khulnasoft.com/v1.25/docs/preprocessor), as well as other [Integrations](/integrations) that might help you fetch data from other resources.

#### Indexing Pipeline

```python
from deepstack import Pipeline
from deepstack.document_stores import ElasticsearchDocumentStore
from deepstack.nodes import TextConverter, PreProcessor

document_store = ElasticsearchDocumentStore(host = "localhost", port = 9200)
converter = TextConverter()
preprocessor = PreProcessor()

indexing_pipeline = Pipeline()
indexing_pipeline.add_node(component=converter, name="TextConverter", inputs=["File"])
indexing_pipeline.add_node(component=preprocessor, name="PreProcessor", inputs=["TextConverter"])
indexing_pipeline.add_node(component=document_store, name="DocumentStore", inputs=["PreProcessor"])

indexing_pipeline.run(file_paths=["filename.txt"])
```

### Using Elasticsearch in a Query Pipeline

Once you have documents in your `ElasitsearchDocumentStore`, it's ready to be used in any Deepstack pipeline. Such as a Retrieval Augmented Generation (RAG) pipeline. Learn more about [Retrievers](https://docs.deepstack.khulnasoft.com/v1.25/docs/retriever) to make use of vector search within your LLM pipelines.

```python
from deepstack import Pipeline
from deepstack.document_stores import ElasticsearchDocumentStore
from deepstack.nodes import EmbeddingRetriever, PromptNode

document_store = ElasticsearchDocumentStore()
retriever = EmbeddingRetriever(document_store = document_store,
                               embedding_model="sentence-transformers/multi-qa-mpnet-base-dot-v1")
prompt_node = PromptNode(model_name_or_path = "google/flan-t5-xl", default_prompt_template = "khulnasoft/question-answering")

query_pipeline = Pipeline()
query_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
query_pipeline.add_node(component=prompt_node, name="PromptNode", inputs=["Retriever"])

query_pipeline.run(query = "Where is Istanbul?")
```
