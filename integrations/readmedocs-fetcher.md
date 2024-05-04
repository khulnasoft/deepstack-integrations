---
layout: integration
name: ReadMeDocs Fetcher
description: Fetch documentation pages from ReadMe docs sites.
authors:
    - name: Tuana Çelik
      socials:
        github: tuanacelik
        twitter: tuanacelik
        linkedin: tuanacelik
    - name: Silvano Cerza
      socials:
        github: silvanocerza
        linkedin: silvanocerza
pypi: https://pypi.org/project/readmedocs-fetcher-deepstack/
repo: https://github.com/tuanacelik/readmedocs-fetcher-deepstack
type: Data Ingestion
report_issue: https://github.com/tuanacelik/readmedocs-fetcher-deepstack/issues
toc: true
---

This custom component for Deepstack is designed to fetch documentation pages from the [ReadMe](https://readme.com/) documentation you have access to. It uses a `MarkdownConverter` to convert all of your documentation pages to a list of Deepstack `Documents`. You can use this node as a standalone node or within an indexing pipeline. 

## Installation

```bash
pip install readmedocs-fetcher-deepstack
```

## Usage

1. To initialize a `ReadmeDocsFetcher` you have to provide an `api_key` parameter. This is your ReadMe Docs API Key.
2. There are 3 optional parameters to initialize the `ReadmeDocsFetcher`
    - `slugs`: To fetch a list of specific pages from your documentation. E.g. if you have want to fetch "https://docs.deepstack.khulnasoft.com/docs/installation" the slug would be `installation`. If not set, all of the available pages will be fetched.
    - `base_url`: Optionally provide this to add the full url of a documentation page to the `meta` of the created document. For example `base_url='https://docs.deepstack.khulnasoft.com'`
    - `version`: If not set, the latest stable version of tour docs will be fetched. 
    - `markdown_converter`: When documents are fetched from ReadMe, temporary `.md` files are created and we use a [`MarkdownConverter`](https://docs.deepstack.khulnasoft.com/v1.25/reference/file-converters-api#markdownconverter) to create a list of deepstack `Documents`. If not provided at initialization, then a `MarkdownConverter` with the default parameters is used.

### Standalone
```python
import os
from dotenv import load_dotenv
from deepstack.nodes import MarkdownConverter
from readmedocs_fetcher_deepstack import ReadmeDocsFetcher

load_dotenv()
README_API_KEY = os.getenv('README_API_KEY')

converter = MarkdownConverter(remove_code_snippets=False)
readme_fetcher = ReadmeDocsFetcher(api_key=README_API_KEY, markdown_converter=converter, base_url="https://docs.deepstack.khulnasoft.com")
readme_fetcher.fetch_docs()
```

To fetch a single doc from a specific version:
```python
readme_fetcher.fetch_docs(slugs=["nodes_overview"], version="v1.18")
```
### In a Pipeline

```python
import os
from dotenv import load_dotenv
from deepstack import Pipeline
from deepstack.nodes import MarkdownConverter, PreProcessor
from deepstack.document_stores import InMemoryDocumentStore
from readmedocs_fetcher_deepstack import ReadmeDocsFetcher

load_dotenv()
README_API_KEY = os.getenv('README_API_KEY')

converter = MarkdownConverter(remove_code_snippets=False)
readme_fetcher = ReadmeDocsFetcher(api_key=README_API_KEY, markdown_converter=converter, base_url="https://docs.deepstack.khulnasoft.com"))

preprocessor = PreProcessor()
doc_store = InMemoryDocumentStore()

pipe = Pipeline()
pipe.add_node(component=readme_fetcher, name="ReadmeFetcher", inputs=["File"])
pipe.add_node(component=preprocessor, name="Preprocessor", inputs=["ReadmeFetcher"])
pipe.add_node(component=doc_store, name="DocumentStore", inputs=["Preprocessor"])
pipe.run()
```

To fetch a single documentation page:
```python
pipe.run(params={"ReadmeFetcher":{"slugs": ["nodes_overview"]}})
```
