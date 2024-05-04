---
layout: integration
name: NVIDIA
description: Use NVIDIA models with Deepstack.
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/nvidia-deepstack
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/nvidia
type: Model Provider
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/nvidia.png
version: Deepstack 2.0
toc: true
---
### **Table of Contents**
- [Overview](#overview)
- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Overview

[NVIDIA AI Foundation Models](https://www.nvidia.com/en-us/ai-data-science/foundation-models/) and [NVIDIA Inference Microservices](https://build.nvidia.com/explore/discover) allow you to reach optimal performance on NVIDIA accelerated infrastructure. With pretrained generative AI models, enterprises can create custom models faster and take advantage of the latest training and inference techniques. 

This integration allows you to use NVIDIA Foundation Models and NVIDIA Inference Microservices in your Deepstack pipelines.


In order to use this integration, you'll need a [NVIDIA API key](https://org.ngc.nvidia.com/setup). Set it as an environment variable, `NVIDIA_API_KEY`.

## Installation

```bash
pip install nvidia-deepstack
```

## Usage
### Components
This integration introduces the following components:

- `NvidiaTextEmbedder`: 
    A component for embedding strings, using NVIDIA AI Foundation and NVIDIA Inference Microservices embedding models.

    For models that differentiate between query and document inputs,
    this component embeds the input string as a query.
  
- `NvidiaDocumentEmbedder`:
    A component for embedding documents, using NVIDIA AI Foundation and NVIDIA Inference Microservices embedding models.

- `NvidiaGenerator`:     A component for generating text using generative models provided by NVIDIA AI Foundation Endpoints and NVIDIA Inference Microservices.

## Use the components on their own:
  
### `NvidiaTextEmbedder`:

```python
from deepstack_integrations.components.embedders.nvidia import NvidiaTextEmbedder

text_to_embed = "I love pizza!"

text_embedder = NvidiaTextEmbedder(model="nvolveqa_40k")
text_embedder.warm_up()

print(text_embedder.run(text_to_embed))
# {'embedding': [-0.02264290489256382, -0.03457780182361603, ...}

```
### `NvidiaDocumentEmbedder`:
```python
from deepstack.dataclasses import Document
from deepstack_integrations.components.embedders.nvidia import NvidiaDocumentEmbedder

documents = [Document(content="Pizza is made with dough and cheese"),
             Document(content="Cake is made with floud and sugar"),
             Document(content="Omlette is made with eggs")]



document_embedder = NvidiaDocumentEmbedder(model="nvolveqa_40k")
document_embedder.warm_up()
document_embedder.run(documents=documents)
#{'documents': [Document(id=2136941caed9b4667d83f906a80d9a2fad1ce34861392889016830ac8738e6c4, content: 'Pizza is made with dough and cheese', embedding: vector of size 1024), ... 'meta': {'usage': {'prompt_tokens': 36, 'total_tokens': 36}}}
```


### `NvidiaGenerator`:

```python
from deepstack_integrations.components.generators.nvidia import NvidiaGenerator

generator = NvidiaGenerator(
    model="nv_llama2_rlhf_70b",
    model_arguments={
        "temperature": 0.2,
        "top_p": 0.7,
        "max_tokens": 1024,
        "seed": None,
        "bad": None,
        "stop": None,
    },
)
generator.warm_up()

result = generator.run(prompt="When was the Golden Gate Bridge built?")
print(result["replies"])
print(result["meta"])
# ['The Golden Gate Bridge was built in 1937 and was completed and opened to the public on May 29, 1937....'[{'role': 'assistant', 'finish_reason': 'stop'}]
```

## Use NVIDIA components in Deepstack pipelines

### Indexing pipeline
```python
from deepstack_integrations.components.generators.nvidia import NvidiaGenerator
from deepstack_integrations.components.embedders.nvidia import NvidiaDocumentEmbedder
from deepstack import Pipeline
from deepstack.dataclasses import Document
from deepstack.components.writers import DocumentWriter
from deepstack.document_stores.in_memory import InMemoryDocumentStore

documents = [Document(content="Tilde lives in San Francisco"),
             Document(content="Tuana lives in Amsterdam"),
             Document(content="Bilge lives in Istanbul")]

document_store = InMemoryDocumentStore()

document_embedder = NvidiaDocumentEmbedder(model="nvolveqa_40k")
writer = DocumentWriter(document_store=document_store)

indexing_pipeline = Pipeline()
indexing_pipeline.add_component(instance=document_embedder, name="document_embedder")
indexing_pipeline.add_component(instance=writer, name="writer")

indexing_pipeline.connect("document_embedder.documents", "writer.documents")
indexing_pipeline.run(data={"document_embedder":{"documents": documents}})

# Calling filter with no arguments will print the contents of the document store
document_store.filter_documents({})

```

### RAG Query pipeline
```python
from deepstack.document_stores.in_memory import InMemoryDocumentStore
from deepstack.components.retrievers.in_memory import InMemoryEmbeddingRetriever
from deepstack.components.builders import PromptBuilder
from deepstack_integrations.components.generators.nvidia import NvidiaGenerator
from deepstack_integrations.components.embedders.nvidia import NvidiaTextEmbedder

prompt = """ Answer the query, based on the
content in the documents.
If you can't answer based on the given documents, say so.

Documents:
{% for doc in documents %}
  {{doc.content}}
{% endfor %}

Query: {{query}}
"""

text_embedder = NvidiaTextEmbedder(model="nvolveqa_40k")
retriever = InMemoryEmbeddingRetriever(document_store=document_store)
prompt_builder = PromptBuilder(template=prompt)
generator = NvidiaGenerator(model="nv_llama2_rlhf_70b")
generator.warm_up()

rag_pipeline = Pipeline()

rag_pipeline.add_component(instance=text_embedder, name="text_embedder")
rag_pipeline.add_component(instance=retriever, name="retriever")
rag_pipeline.add_component(instance=prompt_builder, name="prompt_builder")
rag_pipeline.add_component(instance=generator, name="generator")

rag_pipeline.connect("text_embedder.embedding", "retriever.query_embedding")
rag_pipeline.connect("retriever.documents", "prompt_builder.documents")
rag_pipeline.connect("prompt_builder", "generator")

question = "Who lives in San Francisco?"
result = rag_pipeline.run(data={"text_embedder":{"text": question},
                                "prompt_builder":{"query": question}})
print(result)
# {'text_embedder': {'meta': {'usage': {'prompt_tokens': 10, 'total_tokens': 10}}}, 'generator': {'replies': ['Tilde'], 'meta': [{'role': 'assistant', 'finish_reason': 'stop'}], 'usage': {'completion_tokens': 3, 'prompt_tokens': 101, 'total_tokens': 104}}}

```

### License

`nvidia-deepstack` is distributed under the terms of the [Apache-2.0](https://spdx.org/licenses/Apache-2.0.html) license.
