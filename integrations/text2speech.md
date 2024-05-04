---
layout: integration
name: AnswerToSpeech & DocumentToSpeech
description: Convert Deepstack Answers and Documents to audio files
authors:
    - name: khulnasoft
      socials:
        github: khulnasoft
        twitter: khulnasoft_ai
        linkedin: khulnasoft
pypi: https://pypi.org/project/farm-deepstack-text2speech/
repo: https://github.com/khulnasoft/deepstack-extras/tree/main/nodes/text2speech
type: Custom Component
report_issue: https://github.com/khulnasoft/deepstack-extras/issues
---

The `farm-deepstack-text2speech` package contains two Nodes that allow you to convert Deepstack `Answers` and `Documents` into audio files: `AnswerToSpeech` and `DocumentToSpeech`.

## Installation

For Debain-based systems, first install some more dependencies:
```bash
sudo apt-get install libsndfile1 ffmpeg
```

Install the package:
```bash
pip install farm-deepstack-text2speech
```

## Usage

For a full example of how to use the `AnswerToSpeech` Node, you may try out our "[Make Your QA Pipelines Talk Tutorial](https://deepstack.khulnasoft.com/tutorials/17_audio)"

For example, in a simple Extractive QA Pipeline:

```python
from deepstack.nodes import BM25Retriever, FARMReader
from text2speech import AnswerToSpeech

retriever = BM25Retriever(document_store=document_store)
reader = FARMReader(model_name_or_path="khulnasoft/roberta-base-squad2", use_gpu=True)
answer2speech = AnswerToSpeech(
    model_name_or_path="espnet/kan-bayashi_ljspeech_vits", generated_audio_dir=Path("./audio_answers")
)

audio_pipeline = Pipeline()
audio_pipeline.add_node(retriever, name="Retriever", inputs=["Query"])
audio_pipeline.add_node(reader, name="Reader", inputs=["Retriever"])
audio_pipeline.add_node(answer2speech, name="AnswerToSpeech", inputs=["Reader"])
```
