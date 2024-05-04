---
layout: integration
name: FAISS
description: Use a FAISS vector database with Deepstack
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/farm-deepstack
repo: https://github.com/khulnasoft/deepstack
type: Document Store
report_issue: https://github.com/khulnasoft/deepstack/issues
logo: /logos/meta.png
---

[Faiss](https://github.com/facebookresearch/faiss#readme) is a project by Meta, for efficient vector search. You can use it in your Deepstack pipelines with the [FAISSDocumentStore](https://docs.deepstack.khulnasoft.com/v1.25/docs/document_store#initialization)

For a detailed explanation on different initialization options of the `FAISSDocumentStore`, please visit the [Deepstack Documentation](https://docs.deepstack.khulnasoft.com/v1.25/docs/document_store#initialization) and [API Reference](https://docs.deepstack.khulnasoft.com/v1.25/reference/document-store-api#faissdocumentstore). Below are some examples of how you might use it within a Deepstack Pipeline.

## Installation

```bash
pip install farm-deepstack[faiss]
```

or to install `FAISSDocumentStore` with GPU support, you may install:
```bash
pip install farm-deepstack[faiss-gpu]
```

## Usage

Once installed, you can start using FAISS with Deepstack by initializing it: 

```python
from deepstack.document_stores import FAISSDocumentStore

document_store = FAISSDocumentStore()
```

### Writing Documents to FAISSDocumentStore

To write documents to your `FAISSDocumentStore`, create an indexing pipeline, or use the `write_documents()` function.
For this step, you may make use of the available [FileConverters](https://docs.deepstack.khulnasoft.com/v1.25/docs/file_converters) and [PreProcessors](https://docs.deepstack.khulnasoft.com/v1.25/docs/preprocessor), as well as other [Integrations](/integrations) that might help you fetch data from other resources.

#### Indexing Pipeline

```python
from deepstack import Pipeline
from deepstack.document_stores import FAISSDocumentStore
from deepstack.nodes import PDFToTextConverter, PreProcessor

document_store = FAISSDocumentStore()
converter = PDFToTextConverter()
preprocessor = PreProcessor()

indexing_pipeline = Pipeline()
indexing_pipeline.add_node(component=converter, name="PDFConverter", inputs=["File"])
indexing_pipeline.add_node(component=preprocessor, name="PreProcessor", inputs=["PDFConverter"])
indexing_pipeline.add_node(component=document_store, name="DocumentStore", inputs=["PreProcessor"])

indexing_pipeline.run(file_paths=["filename.pdf"])
```

### Using Faiss in a Query Pipeline

Once you have documents in your `FAISSDocumentStore`, it's ready to be used in any Deepstack pipeline. Such as a Retrieval Augmented Generation (RAG) pipeline. Learn more about [Retrievers](https://docs.deepstack.khulnasoft.com/v1.25/docs/retriever) to make use of vector search within your LLM pipelines.

```python
from deepstack import Pipeline
from deepstack.document_stores import FAISSDocumentStore
from deepstack.nodes import EmbeddingRetriever, PromptNode

document_store = FAISSDocumentStore()
retriever = EmbeddingRetriever(document_store = document_store,
                               embedding_model="sentence-transformers/multi-qa-mpnet-base-dot-v1")
prompt_node = PromptNode(model_name_or_path = "gpt-4",
                         api_key = "YOUR_OPENAI_KEY",
                         default_prompt_template = "khulnasoft/question-answering-with-references")

query_pipeline = Pipeline()
query_pipeline.add_node(component=retriever, name="Retriever", inputs=["Query"])
query_pipeline.add_node(component=prompt_node, name="PromptNode", inputs=["Retriever"])

query_pipeline.run(query = "What is Deepstack?")
```