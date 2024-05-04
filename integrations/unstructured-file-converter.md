---
layout: integration
name: Unstructured File Converter
description: Component to easily convert files and directories into Documents using the Unstructured API
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/unstructured-fileconverter-deepstack/
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/unstructured
type: Data Ingestion
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/unstructured.svg
version: Deepstack 2.0
---

Component for the Deepstack (2.x) LLM framework to easily convert files and directories into Documents using the Unstructured API.

**[Unstructured](https://unstructured-io.github.io/unstructured/index.html)** provides a series of tools to do **ETL for LLMs**. This component calls the Unstructured API that simply extracts text and other information from a vast range of file formats. See [supported file types](https://unstructured-io.github.io/unstructured/api.html#supported-file-types).

## Installation

```bash
pip install unstructured-fileconverter-deepstack
```

### Hosted API
If you plan to use the hosted version of the Unstructured API, you just need the **(free) Unsctructured API key**. You can get it by signing up [here](https://unstructured.io/api-key-free).

### Local API (Docker)
If you want to run your own local instance of the Unstructured API, you need Docker and you can find instructions [here](https://unstructured-io.github.io/unstructured/api.html#using-docker-images).

In short, this should work:
```bash
docker run -p 8000:8000 -d --rm --name unstructured-api quay.io/unstructured-io/unstructured-api:latest --port 8000 --host 0.0.0.0
```

## Usage

If you plan to use the hosted version of the Unstructured API, set the Unstructured API key as an environment variable `UNSTRUCTURED_API_KEY`:
```bash
export UNSTRUCTURED_API_KEY=your_api_key
```

### In isolation
```python
import os
from deepstack_integrations.components.converters.unstructured import UnstructuredFileConverter

converter = UnstructuredFileConverter()
documents = converter.run(paths = ["a/file/path.pdf", "a/directory/path"])["documents"]
```

### In a Deepstack Pipeline
```python
import os
from deepstack import Pipeline
from deepstack.components.writers import DocumentWriter
from deepstack.document_stores.in_memory import InMemoryDocumentStore
from deepstack_integrations.components.converters.unstructured import UnstructuredFileConverter

document_store = InMemoryDocumentStore()

indexing = Pipeline()
indexing.add_component("converter", UnstructuredFileConverter())
indexing.add_component("writer", DocumentWriter(document_store))
indexing.connect("converter", "writer")

indexing.run({"converter": {"paths": ["a/file/path.pdf", "a/directory/path"]}})
```