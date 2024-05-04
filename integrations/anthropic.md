---
layout: integration
name: Anthropic
description: Use Anthropic Models with Deepstack
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/anthropic-deepstack/
repo: https://github.com/khulnasoft/deepstack-core-integrations/tree/main/integrations/anthropic
type: Model Provider
report_issue: https://github.com/khulnasoft/deepstack-core-integrations/issues
logo: /logos/anthropic.png
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

## Overview

This integration supports Anthropic Claude models provided through Anthropic’s own inferencing infrastructure. For a full list of available models, check out [the Anthropic Claude documentation](https://www.anthropic.com/claude).

## Deepstack 2.0

You can use Anthropic models with [`AnthropicGenerator`](https://docs.deepstack.khulnasoft.com/docs/anthropicgenerator) and [`AnthropicChatGenerator`](https://docs.deepstack.khulnasoft.com/docs/anthropicchatgenerator).

Currently, available models are:

- `claude-2.1`
- `claude-3-haiku-20240307`
- `claude-3-sonnet-20240229` (default)
- `claude-3-opus-20240229`

### Installation

```bash
pip install anthropic-deepstack
```

### Usage

Based on your use case, you can choose between [`AnthropicGenerator`](https://docs.deepstack.khulnasoft.com/docs/anthropicgenerator) or [`AnthropicChatGenerator`](https://docs.deepstack.khulnasoft.com/docs/anthropicchatgenerator) to work with Anthropic models. To learn more about the difference, visit the [Generators vs Chat Generators](https://docs.deepstack.khulnasoft.com/docs/generators-vs-chat-generators) guide.  
Before using, make sure to set the `ANTHROPIC_API_KEY` environment variable.

#### Using `AnthropicChatGenerator`

Below is an example RAG Pipeline where we answer a predefined question using the contents from the below mentioned URL pointing to Anthropic prompt engineering guide. We fetch the contents of the URL and generate an answer with the `AnthropicChatGenerator`.

```python
from deepstack import Pipeline
from deepstack.components.builders import DynamicChatPromptBuilder
from deepstack.components.converters import HTMLToDocument
from deepstack.components.fetchers import LinkContentFetcher
from deepstack.components.generators.utils import print_streaming_chunk
from deepstack.dataclasses import ChatMessage
from deepstack.utils import Secret

from deepstack_integrations.components.generators.anthropic import AnthropicChatGenerator

messages = [
    ChatMessage.from_system("You are a prompt expert who answers questions based on the given documents."),
    ChatMessage.from_user("Here are the documents: {{documents}} \\n Answer: {{query}}"),
]

rag_pipeline = Pipeline()
rag_pipeline.add_component("fetcher", LinkContentFetcher())
rag_pipeline.add_component("converter", HTMLToDocument())
rag_pipeline.add_component("prompt_builder", DynamicChatPromptBuilder(runtime_variables=["documents"]))
rag_pipeline.add_component(
    "llm",
    AnthropicChatGenerator(
        api_key=Secret.from_env_var("ANTHROPIC_API_KEY"),
        model="claude-3-sonnet-20240229",
        streaming_callback=print_streaming_chunk,
    ),
)


rag_pipeline.connect("fetcher", "converter")
rag_pipeline.connect("converter", "prompt_builder")
rag_pipeline.connect("prompt_builder", "llm")

question = "What are the best practices in prompt engineering?"
rag_pipeline.run(
    data={
        "fetcher": {"urls": ["https://docs.anthropic.com/claude/docs/prompt-engineering"]},
        "prompt_builder": {"template_variables": {"query": question}, "prompt_source": messages},
    }
)
```

#### Using `AnthropicGenerator`

Below is an example of using `AnthropicGenerator`:

```python
from deepstack_integrations.components.generators.anthropic import AnthropicGenerator

client = AnthropicGenerator(model="claude-2.1")
response = client.run("What's Natural Language Processing? Be brief.")
print(response)

>>{'replies': ['Natural language processing (NLP) is a branch of artificial intelligence focused on enabling
>>computers to understand, interpret, and manipulate human language. The goal of NLP is to read, decipher,
>> understand, and make sense of the human languages in a manner that is valuable.'], 'meta': {'model':
>> 'claude-2.1', 'index': 0, 'finish_reason': 'end_turn', 'usage': {'input_tokens': 18, 'output_tokens': 58}}}
```

## Deepstack 1.x

You can use [Anhtropic Claude](https://docs.anthropic.com/claude/reference/getting-started-with-the-api) in your Deepstack 1.x pipelines with the [PromptNode](https://docs.deepstack.khulnasoft.com/v1.25/docs/prompt_node#using-anthropic-generative-models), which can also be used with and [Agent](https://docs.deepstack.khulnasoft.com/v1.25/docs/agent).

### Installation (1.x)

```bash
pip install farm-deepstack[inference]
```

### Usage (1.x)

You can use Anthropic models in various ways:

#### Using Claude with PromptNode

To use Claude for prompting and generating answers, initialize a `PromptNode` with the model name, your Anthropic API key and a prompt template. You can then use this `PromptNode` in a question answering pipeline to generate answers based on the given context.  

Below is the example of a `PromptNode` that uses a custom `PromptTemplate`

```python
from deepstack.nodes import PromptTemplate, PromptNode

prompt_text = """
Answer the following question.
Question: {query}
Answer:
"""

prompt_template = PromptTemplate(prompt=prompt_text)

prompt_node = PromptNode(
    model_name_or_path = "claude-2",
    default_prompt_template=PromptTemplate(prompt_text),
    api_key='YOUR_ANTHROPIC_API_KEY',
    max_length=768,
    model_kwargs={"stream": True},
)
```

### Using Claude for Agents

To use Calude for an `Agent`, simply provide a `PromptNode` that uses Claude to the `Agent`:

```python
from deepstack.agents import Agent
from deepstack.nodes import PromptNode

prompt_node = PromptNode(model_name_or_path="YOUR_ANTHROPIC_API_KEY", api_key=anthropic_key, stop_words=["Observation:"])
agent = Agent(prompt_node=prompt_node)
```
