---
layout: integration
name: Hugging Face
description: Use Models on Hugging Face with Deepstack
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/farm-deepstack
repo: https://github.com/khulnasoft/deepstack
type: Model Provider
report_issue: https://github.com/khulnasoft/deepstack/issues
logo: /logos/huggingface.png
version: Deepstack 2.0
toc: true
---

### **Table of Contents**

- [Deepstack 2.0](#deepstack-20)
  - [Installation](#installation)
  - [Usage](#usage)
- [Deepstack 1.x](#deepstack-1x)
  - [Installation (1.x)](#installation-1x)
  - [Usage (1.x)](#usage-1x)

## Deepstack 2.0

You can use models on [Hugging Face](https://huggingface.co/) in your Deepstack 2.0 pipelines with [Generators](https://docs.deepstack.khulnasoft.com/docs/generators), [Embedders](https://docs.deepstack.khulnasoft.com/docs/embedders), [Rankers](https://docs.deepstack.khulnasoft.com/docs/rankers) and [Readers](https://docs.deepstack.khulnasoft.com/docs/readers)!

### Installation

```bash
pip install deepstack-ai
```

### Usage

You can use models on Hugging Face in various ways:

#### Embedding Models

You can leverage embedding models from Hugging Face through two components: [SentenceTransformersTextEmbedder](https://docs.deepstack.khulnasoft.com/docs/sentencetransformerstextembedder) and [SentenceTransformersDocumentEmbedder](https://docs.deepstack.khulnasoft.com/docs/sentencetransformersdocumentembedder).

To create semantic embeddings for documents, use `SentenceTransformersDocumentEmbedder` in your indexing pipeline. For generating embeddings for queries, use `SentenceTransformersTextEmbedder`. Once you've selected the suitable component for your specific use case, initialize the component with the desired model name.

Below is the example indexing pipeline with `InMemoryDocumentStore`, `DocumentWriter` and  `SentenceTransformersDocumentEmbedder`:

```python
from deepstack import Document
from deepstack import Pipeline
from deepstack.document_stores.in_memory import InMemoryDocumentStore
from deepstack.components.embedders import SentenceTransformersDocumentEmbedder
from deepstack.components.writers import DocumentWriter

document_store = InMemoryDocumentStore(embedding_similarity_function="cosine")

documents = [Document(content="My name is Wolfgang and I live in Berlin"),
             Document(content="I saw a black horse running"),
             Document(content="Germany has many big cities")]

indexing_pipeline = Pipeline()
indexing_pipeline.add_component("embedder", SentenceTransformersDocumentEmbedder(model="sentence-transformers/all-MiniLM-L6-v2"))
indexing_pipeline.add_component("writer", DocumentWriter(document_store=document_store))
indexing_pipeline.connect("embedder", "writer")
indexing_pipeline.run({
    "embedder":{"documents":documents}
    })
```

#### Generative Models (LLMs) 

You can leverage text generation models from Hugging Face through three components: [HuggingFaceLocalGenerator](https://docs.deepstack.khulnasoft.com/docs/huggingfacelocalgenerator), [HuggingFaceTGIGenerator](https://docs.deepstack.khulnasoft.com/docs/huggingfacetgigenerator) and [HuggingFaceTGIChatGenerator](https://docs.deepstack.khulnasoft.com/docs/huggingfacetgichatgenerator).

Depending on the model type (chat or text completion) and hosting option (TGI, Inference Endpoint, locally hosted), select the suitable Hugging Face Generator component and initialize it with the model name

Below is the example query pipeline that uses `mistralai/Mistral-7B-v0.1` hosted on Hugging Face Inference endpoints with `HuggingFaceTGIGenerator`:

```python
from deepstack import Pipeline
from deepstack.utils import Secret
from deepstack.components.retrievers.in_memory import InMemoryBM25Retriever
from deepstack.components.builders.prompt_builder import PromptBuilder
from deepstack.components.generators import HuggingFaceTGIGenerator

template = """
Given the following information, answer the question.

Context: 
{% for document in documents %}
    {{ document.text }}
{% endfor %}

Question: What's the official language of {{ country }}?
"""
pipe = Pipeline()

pipe.add_component("retriever", InMemoryBM25Retriever(document_store=docstore))
pipe.add_component("prompt_builder", PromptBuilder(template=template))
pipe.add_component("llm", HuggingFaceTGIGenerator(model="mistralai/Mistral-7B-v0.1", token=Secret.from_token("YOUR_HF_API_TOKEN")))
pipe.connect("retriever", "prompt_builder.documents")
pipe.connect("prompt_builder", "llm")

pipe.run({
    "prompt_builder": {
        "country": "France"
    }
})
```

#### Ranker Models

To use cross encoder models on Hugging Face, initialize a `SentenceTransformersRanker` with the model name. You can then use this `SentenceTransformersRanker` to sort documents based on their relevancy to the query.

Below is the example of document retrieval pipeline with `InMemoryBM25Retriever` and  `SentenceTransformersRanker`:

```python
from deepstack import Document, Pipeline
from deepstack.document_stores.in_memory import InMemoryDocumentStore
from deepstack.components.retrievers.in_memory import InMemoryBM25Retriever
from deepstack.components.rankers import TransformersSimilarityRanker

docs = [Document(content="Paris is in France"), 
        Document(content="Berlin is in Germany"),
        Document(content="Lyon is in France")]
document_store = InMemoryDocumentStore()
document_store.write_documents(docs)

retriever = InMemoryBM25Retriever(document_store = document_store)
ranker = TransformersSimilarityRanker(model="cross-encoder/ms-marco-MiniLM-L-6-v2")

document_ranker_pipeline = Pipeline()
document_ranker_pipeline.add_component(instance=retriever, name="retriever")
document_ranker_pipeline.add_component(instance=ranker, name="ranker")
document_ranker_pipeline.connect("retriever.documents", "ranker.documents")

query = "Cities in France"
document_ranker_pipeline.run(data={"retriever": {"query": query, "top_k": 3}, 
                                   "ranker": {"query": query, "top_k": 2}})
```

#### Reader Models

To use question answering models on Hugging Face, initialize a `ExtractiveReader` with the model name. You can then use this `ExtractiveReader` to extract answers from the relevant context.

Below is the example of extractive question answering pipeline with `InMemoryBM25Retriever` and  `ExtractiveReader`:

```python
from deepstack import Document, Pipeline
from deepstack.document_stores.in_memory import InMemoryDocumentStore
from deepstack.components.retrievers.in_memory import InMemoryBM25Retriever
from deepstack.components.readers import ExtractiveReader

docs = [Document(content="Paris is the capital of France."),
        Document(content="Berlin is the capital of Germany."),
        Document(content="Rome is the capital of Italy."),
        Document(content="Madrid is the capital of Spain.")]
document_store = InMemoryDocumentStore()
document_store.write_documents(docs)

retriever = InMemoryBM25Retriever(document_store = document_store)
reader = ExtractiveReader(model="khulnasoft/roberta-base-squad2-distilled")

extractive_qa_pipeline = Pipeline()
extractive_qa_pipeline.add_component(instance=retriever, name="retriever")
extractive_qa_pipeline.add_component(instance=reader, name="reader")

extractive_qa_pipeline.connect("retriever.documents", "reader.documents")

query = "What is the capital of France?"
extractive_qa_pipeline.run(data={"retriever": {"query": query, "top_k": 3}, 
                                   "reader": {"query": query, "top_k": 2}})
```

## Deepstack 1.x

You can use models on [Hugging Face](https://huggingface.co/) in your Deepstack 1.x pipelines with the [PromptNode](https://docs.deepstack.khulnasoft.com/v1.25/docs/prompt_node), [EmbeddingRetriever](https://docs.deepstack.khulnasoft.com/v1.25/docs/retriever#embedding-retrieval-recommended), [Ranker](https://docs.deepstack.khulnasoft.com/v1.25/docs/ranker), [Reader](https://docs.deepstack.khulnasoft.com/v1.25/docs/reader) and more!

### Installation (1.x)

```bash
pip install farm-deepstack
```

### Usage (1.x)

You can use models on Hugging Face in various ways:

#### Embedding Models

To use embedding models on Hugging Face, initialize an `EmbeddingRetriever` with the model name. You can then use this `EmbeddingRetriever` in an indexing pipeline to create semantic embeddings for documents and index them to a document store. 

Below is the example indexing pipeline with `PreProcessor`, `InMemoryDocumentStore` and  `EmbeddingRetriever`:

```python
from deepstack.nodes import EmbeddingRetriever
from deepstack.document_stores import InMemoryDocumentStore
from deepstack.pipelines import Pipeline
from deepstack.schema import Document

document_store = InMemoryDocumentStore(embedding_dim=384)
preprocessor = PreProcessor()
retriever = EmbeddingRetriever(
    embedding_model="sentence-transformers/all-MiniLM-L6-v2", document_store=document_store
)

indexing_pipeline = Pipeline()
indexing_pipeline.add_node(component=preprocessor, name="Preprocessor", inputs=["File"])
indexing_pipeline.add_node(component=retriever, name="Retriever", inputs=["Preprocessor"])
indexing_pipeline.add_node(component=document_store, name="document_store", inputs=["Retriever"])
indexing_pipeline.run(documents=[Document("This is my document")])
```

#### Generative Models (LLMs) 

To use text generation models on Hugging Face, initialize a `PromptNode` with the model name and the prompt template. You can then use this `PromptNode` to generate questions from the given context.  

Below is the example of question generation pipeline using RAG with `EmbeddingRetriever` and  `PromptNode`:

```python
from deepstack import Pipeline
from deepstack.nodes import BM25Retriever, PromptNode

retriever = EmbeddingRetriever(
    embedding_model="sentence-transformers/all-MiniLM-L6-v2", document_store=document_store
)
prompt_node = PromptNode(model_name_or_path = "mistralai/Mistral-7B-Instruct-v0.1",
                         api_key = "HF_API_KEY",
                         default_prompt_template = "khulnasoft/question-generation")
query_pipeline = Pipeline()
query_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
query_pipeline.add_node(component=prompt_node, name="PromptNode", inputs=["Retriever"])

query_pipeline.run(query = "Berlin")
```

> If you would like to use the [Inference API](https://huggingface.co/inference-api), you need pass your Hugging Face token to PromptNode.


#### Ranker Models

To use cross encoder models on Hugging Face, initialize a `SentenceTransformersRanker` with the model name. You can then use this `SentenceTransformersRanker` to sort documents based on their relevancy to the query.

Below is the example of document retrieval pipeline with `BM25Retriever` and  `SentenceTransformersRanker`:

```python
from deepstack.nodes import SentenceTransformersRanker, BM25Retriever
from deepstack.pipelines import Pipeline

retriever = BM25Retriever(document_store=document_store)
ranker = SentenceTransformersRanker(model_name_or_path="cross-encoder/ms-marco-MiniLM-L-6-v2")

document_retrieval_pipeline = Pipeline()
document_retrieval_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
document_retrieval_pipeline.add_node(component=ranker, name="Ranker", inputs=["Retriever"])
document_retrieval_pipeline.run("YOUR_QUERY")
```

#### Reader Models

To use question answering models on Hugging Face, initialize a `FarmReader` with the model name. You can then use this `FarmReader` to extract answers from the relevant context.

Below is the example of extractive question answering pipeline with `BM25Retriever` and  `FARMReader`:

```python
from deepstack.nodes import BM25Retriever, FARMReader
from deepstack.pipelines import Pipeline

retriever = BM25Retriever(document_store=document_store)
reader = FARMReader(model_name_or_path="khulnasoft/roberta-base-squad2", use_gpu=True)

querying_pipeline = Pipeline()
querying_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
querying_pipeline.add_node(component=reader, name="Reader", inputs=["Retriever"])
querying_pipeline.run("YOUR_QUERY")
```