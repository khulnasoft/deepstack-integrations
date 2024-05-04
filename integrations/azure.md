---
layout: integration
name: Azure
description: Use OpenAI models deployed through Azure services with Deepstack
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: https://www.linkedin.com/company/khulnasoft
pypi: https://pypi.org/project/deepstack-ai/
repo: https://github.com/khulnasoft/deepstack
type: Model Provider
report_issue: https://github.com/khulnasoft/deepstack/issues
logo: /logos/azure.png
version: Deepstack 2.0
toc: true
---

### Table of Contents

- [Overview](#overview)
- [Deepstack 2.0](#deepstack-20)
  - [Installation](#installation)
  - [Usage](#usage)
    - [Embedding Models](#embedding-models)
    - [Generative Models (LLMs)](#generative-models-llms)

## Overview

[Azure OpenAI Service](https://learn.microsoft.com/en-us/azure/ai-services/openai/overview) provides REST API access to OpenAI's powerful language models including the GPT-4, GPT-4 Turbo with Vision, GPT-3.5-Turbo, and Embeddings model series. To get access to Azure OpenAI endpoints, visit [Azure OpenAI Service REST API reference](https://learn.microsoft.com/en-us/azure/ai-services/openai/reference).

## Installation

Install Deepstack 2.0:

```bash
pip install deepstack-ai
```

## Usage

To work with Azure components, you will need an Azure OpenAI API key, an [Azure Active Directory Token](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id) as well as an Azure OpenAI Endpoint.

### Components

- [AzureOpenAIGenerator](https://docs.deepstack.khulnasoft.com/docs/azureopenaigenerator)
- [AzureOpenAIChatGenerator](https://docs.deepstack.khulnasoft.com/docs/azureopenaichatgenerator)
- [AzureOpenAITextEmbedder](https://docs.deepstack.khulnasoft.com/docs/azureopenaitextembedder)
- [AzureOpenAIDocumentEmbedder](https://docs.deepstack.khulnasoft.com/docs/azureopenaidocumentembedder)

All components use `AZURE_OPENAI_API_KEY` and `AZURE_OPENAI_AD_TOKEN` environment variables by default. Otherwise, you can pass `api_key` and `azure_ad_token` at initialization using `Secret` class. Read more about [Secret Handling](https://docs.deepstack.khulnasoft.com/docs/secret-management#structured-secret-handling).

### Embedding Models

You can leverage embedding models from Azure OpenAI through two components: [AzureOpenAITextEmbedder](https://docs.deepstack.khulnasoft.com/docs/azureopenaitextembedder) and [AzureOpenAIDocumentEmbedder](https://docs.deepstack.khulnasoft.com/docs/azureopenaidocumentembedder).

To create semantic embeddings for documents, use `AzureOpenAIDocumentEmbedder` in your indexing pipeline. For generating embeddings for queries, use `AzureOpenAITextEmbedder`. Once you've selected the suitable component for your specific use case, initialize the component with required parameters.

Below is the example indexing pipeline with `InMemoryDocumentStore`, `AzureOpenAIDocumentEmbedder` and  `DocumentWriter`:

```python
from deepstack import Document, Pipeline
from deepstack.document_stores.in_memory import InMemoryDocumentStore
from deepstack.components.embedders import AzureOpenAITextEmbedder, AzureOpenAIDocumentEmbedder
from deepstack.components.writers import DocumentWriter

os.environ["AZURE_OPENAI_API_KEY"] = "Your Azure OpenAI API key"
os.environ["AZURE_OPENAI_AD_TOKEN"] = "Your Azure Active Directory Token"

document_store = InMemoryDocumentStore(embedding_similarity_function="cosine")

documents = [Document(content="My name is Wolfgang and I live in Berlin"),
             Document(content="I saw a black horse running"),
             Document(content="Germany has many big cities")]

indexing_pipeline = Pipeline()
indexing_pipeline.add_component("embedder", AzureOpenAIDocumentEmbedder(azure_endpoint="https://example-resource.azure.openai.com/", azure_deployment="text-embedding-ada-002"))
indexing_pipeline.add_component("writer", DocumentWriter(document_store=document_store))
indexing_pipeline.connect("embedder", "writer")

indexing_pipeline.run({"embedder": {"documents": documents}})
```

### Generative Models (LLMs)

You can leverage Azure OpenAI models through two components: [AzureOpenAIGenerator](https://docs.deepstack.khulnasoft.com/docs/azureopenaigenerator) and [AzureOpenAIChatGenerator](https://docs.deepstack.khulnasoft.com/docs/azureopenaichatgenerator).

To use OpenAI models deployed through Azure services for text generation, initialize a `AzureOpenAIGenerator` with `azure_deployment` and `azure_endpoint`. You can then use the `AzureOpenAIGenerator` instance in a pipeline after the `PromptBuilder`.  

Below is the example of generative questions answering pipeline using RAG with `PromptBuilder` and  `AzureOpenAIGenerator`:

```python
from deepstack import Pipeline
from deepstack.components.retrievers.in_memory import InMemoryBM25Retriever
from deepstack.components.builders.prompt_builder import PromptBuilder
from deepstack.components.generators import AzureOpenAIGenerator

os.environ["AZURE_OPENAI_API_KEY"] = "Your Azure OpenAI API key"
os.environ["AZURE_OPENAI_AD_TOKEN"] = "Your Azure Active Directory Token"

template = """
Given the following information, answer the question.

Context: 
{% for document in documents %}
    {{ document.content }}
{% endfor %}

Question: What's the official language of {{ country }}?
"""
pipe = Pipeline()

pipe.add_component("retriever", InMemoryBM25Retriever(document_store=document_store))
pipe.add_component("prompt_builder", PromptBuilder(template=template))
pipe.add_component("llm", AzureOpenAIGenerator(azure_endpoint="https://example-resource.azure.openai.com/", azure_deployment="gpt-35-turbo"))
pipe.connect("retriever", "prompt_builder.documents")
pipe.connect("prompt_builder", "llm")

pipe.run({
    "prompt_builder": {
        "country": "France"
    }
})   



