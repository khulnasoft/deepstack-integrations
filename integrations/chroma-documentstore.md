---
layout: integration
name: Chroma
description: A Document Store for storing and retrieval from Chroma
authors:
  - name: Massimiliano Pippi
    socials:
      github: masci
  - name: khulnasoft
    socials:
      github: khulnasoft
      twitter: khulnasoft_ai
      linkedin: khulnasoft
pypi: https://pypi.org/project/chroma-store
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/chroma
type: Document Store
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/chroma.png
version: Deepstack 2.0
toc: true
---

[![PyPI - Version](https://img.shields.io/pypi/v/chroma-deepstack.svg)](https://pypi.org/project/chroma-deepstack)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/chroma-deepstack.svg)](https://pypi.org/project/chroma-deepstack)
[![test](https://github.com/masci/chroma-deepstack/actions/workflows/test.yml/badge.svg)](https://github.com/masci/chroma-deepstack/actions/workflows/test.yml)

-----

**Table of Contents**

- [Chroma Document Store for Deepstack](#chroma-document-store-for-deepstack)
  - [Installation](#installation)
  - [Examples](#examples)
  - [License](#license)

## Installation
Use `pip` to install Chroma:

```console
pip install chroma-deepstack
```
## Usage
Once installed, initialize your Chroma database to use it with Deepstack 2.0:

```python
from deepstack_integrations.document_stores.chroma import ChromaDocumentStore

# Chroma is used in-memory so we use the same instances in the two pipelines below
document_store = ChromaDocumentStore()
```

### Writing Documents to ChromaDocumentStore
To write documents to `ChromaDocumentStore`, create an indexing pipeline.

```python
from deepstack.components.converters import TextFileToDocument
from deepstack.components.writers import DocumentWriter

indexing = Pipeline()
indexing.add_component("converter", TextFileToDocument())
indexing.add_component("writer", DocumentWriter(document_store))
indexing.connect("converter", "writer")
indexing.run({"converter": {"sources": file_paths}})
```

## Examples
You can find a code example showing how to use the Document Store and the Retriever under the `example/` folder of [this repo](https://github.com/khulnasoft/deepstack-core-integrations/blob/main/integrations/chroma).

## License

`chroma-deepstack` is distributed under the terms of the [Apache-2.0](https://spdx.org/licenses/Apache-2.0.html) license.
