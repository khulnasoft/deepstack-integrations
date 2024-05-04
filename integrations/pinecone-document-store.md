---
layout: integration
name: Pinecone
description: Use a Pinecone database with Deepstack
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
    - name: Ashwin Mathur
      socials:
        github: awinml
        twitter: awinml
        linkedin: ashwin-mathur-ds
    - name: Varun Mathur
      socials:
        github: vrunm
        twitter: vrunmnlp
        linkedin: varun-mathur-ds        
pypi: https://pypi.org/project/pinecone_deepstack/
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/pinecone
type: Document Store
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/pinecone.png
version: Deepstack 2.0
toc: true
---

### Table of Contents

- [Overview](#overview)
- [Deepstack 2.0](#deepstack-20)
  - [Installation](#installation)
  - [Usage](#usage)
- [Deepstack 1.x](#deepstack-1x)
  - [Installation (1.x)](#installation-1x)
  - [Usage (1.x)](#usage-1x)

## Overview

[Pinecone](https://www.pinecone.io/) is a fast and scalable vector database that you can use in Deepstack pipelines with the [PineconeDocumentStore](https://docs.deepstack.khulnasoft.com/docs/pinecone-document-store).

For a detailed overview of all the available methods and settings for the `PineconeDocumentStore`, visit the Deepstack [API Reference](https://docs.deepstack.khulnasoft.com/reference/integrations-pinecone#pineconedocumentstore).

## Deepstack 2.x

### Installation

```bash
pip install pinecone-deepstack
```

### Usage

To use Pinecone as your data storage for your Deepstack LLM pipelines, you must have an account with Pinecone and an API Key. Once you have those, you can initialize a `PineconeDocumentStore` for Deepstack:

```python
from deepstack_integrations.document_stores.pinecone import PineconeDocumentStore

# Make sure you have the PINECONE_API_KEY environment variable set
document_store = PineconeDocumentStore(metric="cosine", dimension=768, index="YOUR_INDEX_NAME", environment = "YOUR_ENVIRONMENT")
```

#### Writing Documents to PineconeDocumentStore

To write documents to your `PineconeDocumentStore`, create an indexing pipeline, or use the `write_documents()` function.
For this step, you may make use of the available [Converters](https://docs.deepstack.khulnasoft.com/docs/converters) and [PreProcessors](https://docs.deepstack.khulnasoft.com/docs/preprocessors), as well as other [Integrations](/integrations) that might help you fetch data from other resources. Below is an example indexing pipeline that indexes your Markdown files into a Pinecone database.

#### Indexing Pipeline

```python
from deepstack import Pipeline
from deepstack.components.converters import MarkdownToDocument
from deepstack.components.writers import DocumentWriter
from deepstack.components.embedders import SentenceTransformersDocumentEmbedder
from deepstack.components.preprocessors import DocumentSplitter
from deepstack_integrations.document_stores.pinecone import PineconeDocumentStore

# Make sure you have the PINECONE_API_KEY environment variable set
document_store = PineconeDocumentStore(metric="cosine", dimension=768, index="YOUR_INDEX_NAME", environment = "YOUR_ENVIRONMENT")

indexing = Pipeline()
indexing.add_component("converter", MarkdownToDocument())
indexing.add_component("splitter", DocumentSplitter(split_by="sentence", split_length=2))
indexing.add_component("embedder", SentenceTransformersDocumentEmbedder())
indexing.add_component("writer", DocumentWriter(document_store))
indexing.connect("converter", "splitter")
indexing.connect("splitter", "embedder")
indexing.connect("embedder", "writer")

indexing.run({"converter": {"sources": ["filename.md"]}})
```

### Using Pinecone in a RAG Pipeline

Once you have documents in your `PineconeDocumentStore`, they can be used in any Deepstack pipeline. Then, you can use [`PineconeEmbeddingRetriever`](https://docs.deepstack.khulnasoft.com/docs/pineconedenseretriever) to retrieve data from your PineconeDocumentStore. For example, below is a pipeline that uses a custom prompt designed to answer questions for the retrieved documents.

```python
from deepstack.utils import Secret
from deepstack.components.embedders import SentenceTransformersTextEmbedder
from deepstack.components.builders import PromptBuilder
from deepstack.components.generators import OpenAIGenerator
from deepstack_integrations.document_stores.pinecone import PineconeDocumentStore
from deepstack_integrations.components.retrievers.pinecone import PineconeEmbeddingRetriever

# Make sure you have the PINECONE_API_KEY environment variable set
document_store = PineconeDocumentStore(metric="cosine", dimension=768, index="YOUR_INDEX_NAME", environment = "YOUR_ENVIRONMENT")
              
prompt_template = """Answer the following query based on the provided context. If the context does
                     not include an answer, reply with 'I don't know'.\n
                     Query: {{query}}
                     Documents:
                     {% for doc in documents %}
                        {{ doc.content }}
                     {% endfor %}
                     Answer: 
                  """

query_pipeline = Pipeline()
query_pipeline.add_component("text_embedder", SentenceTransformersTextEmbedder())
query_pipeline.add_component("retriever", PineconeEmbeddingRetriever(document_store=document_store))
query_pipeline.add_component("prompt_builder", PromptBuilder(template=prompt_template))
query_pipeline.add_component("generator", OpenAIGenerator(api_key=Secret.from_token("YOUR_OPENAI_API_KEY"), model="gpt-4"))
query_pipeline.connect("text_embedder.embedding", "retriever.query_embedding")
query_pipeline.connect("retriever.documents", "prompt_builder.documents")
query_pipeline.connect("prompt_builder", "generator")

query = "What is Pinecone?"
results = query_pipeline.run(
    {
        "text_embedder": {"text": query},
        "prompt_builder": {"query": query},
    }
)
```


## Deepstack 1.x

### Installation

```bash
pip install farm-deepstack[pinecone]
```

### Usage

To use Pinecone as your data storage for your Deepstack LLM pipelines, you must have an account with Pinecone and an API Key. Once you have those, you can initialize a `PineconeDocumentStore` for Deepstack:

```python
from deepstack_integrations.document_stores.pinecone import PineconeDocumentStore


document_store = PineconeDocumentStore(api_key='YOUR_API_KEY',
                                       similarity="cosine",
                                       embedding_dim=768)
```

#### Writing Documents to PineconeDocumentStore

To write documents to your `PineconeDocumentStore`, create an indexing pipeline, or use the `write_documents()` function.
For this step, you may make use of the available [FileConverters](https://docs.deepstack.khulnasoft.com/v1.25/docs/file_converters) and [PreProcessors](https://docs.deepstack.khulnasoft.com/v1.25/docs/preprocessor), as well as other [Integrations](/integrations) that might help you fetch data from other resources. Below is an example indexing pipeline that indexes your Markdown files into a Pinecone database.

#### Indexing Pipeline

```python
from deepstack import Pipeline
from deepstack.nodes import MarkdownConverter, PreProcessor
from deepstack_integrations.document_stores.pinecone import PineconeDocumentStore


document_store = PineconeDocumentStore(api_key='YOUR_API_KEY',
                                       similarity="cosine",
                                       embedding_dim=768)
converter = MarkdownConverter()
preprocessor = PreProcessor()

indexing_pipeline = Pipeline()
indexing_pipeline.add_node(component=converter, name="PDFConverter", inputs=["File"])
indexing_pipeline.add_node(component=preprocessor, name="PreProcessor", inputs=["PDFConverter"])
indexing_pipeline.add_node(component=document_store, name="DocumentStore", inputs=["PreProcessor"])

indexing_pipeline.run(file_paths=["filename.md"])
```

### Using Pinecone in a Query Pipeline

Once you have documents in your `PineconeDocumentStore`, it's ready to be used in any Deepstack pipeline. For example, below is a pipeline that makes use of a custom prompt that is designed to answer questions for the retrieved documents.

```python
from deepstack import Pipeline
from deepstack.nodes import AnswerParser, EmbeddingRetriever, PromptNode, PromptTemplate
from deepstack_integrations.document_stores.pinecone import PineconeDocumentStore


document_store = PineconeDocumentStore(api_key='YOUR_API_KEY',
                                       similarity="cosine",
                                       embedding_dim=768)
              
retriever = EmbeddingRetriever(document_store = document_store,
                               embedding_model="sentence-transformers/multi-qa-mpnet-base-dot-v1")
prompt_template = PromptTemplate(prompt = """"Answer the following query based on the provided context. If the context does
                                              not include an answer, reply with 'I don't know'.\n
                                              Query: {query}\n
                                              Documents: {join(documents)}
                                              Answer: 
                                          """,
                                          output_parser=AnswerParser())
prompt_node = PromptNode(model_name_or_path = "gpt-4",
                         api_key = "YOUR_OPENAI_KEY",
                         default_prompt_template = prompt_template)

query_pipeline = Pipeline()
query_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
query_pipeline.add_node(component=prompt_node, name="PromptNode", inputs=["Retriever"])

query_pipeline.run(query = "What is Pinecone", params={"Retriever" : {"top_k": 5}})
```
