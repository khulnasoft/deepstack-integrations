---
layout: integration
name: vLLM Invocation Layer
description: Use the vLLM inference engine with Deepstack
authors:
  - name: Lukas Kreussel
    socials:
      github: LLukas22
pypi: https://pypi.org/project/vllm-deepstack/
repo: https://github.com/LLukas22/vLLM-deepstack-adapter
type: Model Provider
report_issue: https://github.com/LLukas22/vLLM-deepstack-adapter/issues
logo: /logos/vllm.png
version: Deepstack 2.0
toc: true
---
[![PyPI - Version](https://img.shields.io/pypi/v/vllm-deepstack.svg)](https://pypi.org/project/vllm-deepstack)
[![PyPI - Python Version](https://img.shields.io/pypi/pyversions/vllm-deepstack.svg)](https://pypi.org/project/vllm-deepstack)

Simply use [vLLM](https://github.com/vllm-project/vllm) in your deepstack pipeline, to utilize fast, self-hosted LLMs. 

<p align="center">
    <img alt="vLLM" src="https://raw.githubusercontent.com/vllm-project/vllm/main/docs/source/assets/logos/vllm-logo-text-light.png" width="45%" style="vertical-align: middle;">
    <a href="https://www.khulnasoft.com/deepstack/">
        <img src="https://raw.githubusercontent.com/khulnasoft/deepstack/main/docs/img/deepstack_logo_colored.png" alt="Deepstack" width="45%" style="vertical-align: middle;">
    </a>
</p>

### Table of Contents

- [Overview](#overview)
- [Deepstack 2.0](#deepstack-20)
  - [Installation](#installation)
  - [Usage](#usage)
- [Deepstack 1.x](#deepstack-1x)
  - [Installation (1.x)](#installation-1x)
  - [Usage (1.x)](#usage-1x)

## Overview

[vLLM](https://github.com/vllm-project/vllm) is a high-throughput and memory-efficient inference and serving engine for LLMs.
It is an open-source project that allows serving open models in production, when you have GPU resources available.

For Deepstack 1.x, the integration is available as a separate package, while for Deepstack 2.x, the integration comes out of the box.

## Deepstack 2.x

vLLM can be deployed as a server that implements the OpenAI API protocol.
This allows vLLM to be used with the [`OpenAIGenerator`](https://docs.deepstack.khulnasoft.com/docs/openaigenerator) and [`OpenAIChatGenerator`](https://docs.deepstack.khulnasoft.com/docs/openaichatgenerator) components in Deepstack.

For an end-to-end example of [vLLM + Deepstack 2.x, see this notebook](https://colab.research.google.com/github/khulnasoft/deepstack-cookbook/blob/main/notebooks/vllm_inference_engine.ipynb).


### Installation
vLLM should be installed.
- you can use `pip`: `pip install vllm` (more information in the [vLLM documentation](https://docs.vllm.ai/en/latest/getting_started/installation.html))
- for production use cases, there are many other options, including Docker ([docs](https://docs.vllm.ai/en/latest/serving/deploying_with_docker.html))

### Usage
You first need to run an vLLM OpenAI-compatible server. You can do that using [Python](https://docs.vllm.ai/en/latest/getting_started/quickstart.html#openai-compatible-server) or [Docker](https://docs.vllm.ai/en/latest/serving/deploying_with_docker.html). 

Then, you can use the `OpenAIGenerator` and `OpenAIChatGenerator` components in Deepstack to query the vLLM server.

```python
from deepstack.components.generators.chat import OpenAIChatGenerator
from deepstack.dataclasses import ChatMessage
from deepstack.utils import Secret

generator = OpenAIChatGenerator(
    api_key=Secret.from_token("VLLM-PLACEHOLDER-API-KEY"),  # for compatibility with the OpenAI API, a placeholder api_key is needed
    model="mistralai/Mistral-7B-Instruct-v0.1",
    api_base_url="http://localhost:8000/v1",
    generation_kwargs = {"max_tokens": 512}
)

response = generator.run(messages=[ChatMessage.from_user("Hi. Can you help me plan my next trip to Italy?")])
```

## Deepstack 1.x

### Installation (1.x)
Install the wrapper via pip:  `pip install vllm-deepstack`

### Usage (1.x)
This integration provides two invocation layers:
- `vLLMInvocationLayer`: To use models hosted on a vLLM server
- `vLLMLocalInvocationLayer`: To use locally hosted vLLM models

#### Use a Model Hosted on a vLLM Server
To utilize the wrapper the `vLLMInvocationLayer` has to be used. 

Here is a simple example of how a `PromptNode` can be created with the wrapper.
```python
from deepstack.nodes import PromptNode, PromptModel
from vllm_deepstack import vLLMInvocationLayer


model = PromptModel(model_name_or_path="", invocation_layer_class=vLLMInvocationLayer, max_length=256, api_key="EMPTY", model_kwargs={
        "api_base" : API, # Replace this with your API-URL
        "maximum_context_length": 2048,
    })

prompt_node = PromptNode(model_name_or_path=model, top_k=1, max_length=256)
```
The model will be inferred based on the model served on the vLLM server.
For more configuration examples, take a look at the unit-tests.

##### Hosting a vLLM Server

To create an *OpenAI-Compatible Server* via vLLM you can follow the steps in the 
Quickstart section of their [documentation](https://vllm.readthedocs.io/en/latest/getting_started/quickstart.html#openai-compatible-server).

#### Use a Model Hosted Locally
⚠️To run `vLLM` locally you need to have `vllm` installed and a supported GPU.

If you don't want to use an API-Server this wrapper also provides a `vLLMLocalInvocationLayer` which executes the vLLM on the same node Deepstack is running on. 

Here is a simple example of how a `PromptNode` can be created with the `vLLMLocalInvocationLayer`.
```python
from deepstack.nodes import PromptNode, PromptModel
from vllm_deepstack import vLLMLocalInvocationLayer

model = PromptModel(model_name_or_path=MODEL, invocation_layer_class=vLLMLocalInvocationLayer, max_length=256, model_kwargs={
        "maximum_context_length": 2048,
    })

prompt_node = PromptNode(model_name_or_path=model, top_k=1, max_length=256)
```
