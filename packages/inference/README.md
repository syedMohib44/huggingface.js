# 🤗 Hugging Face Inference API

A Typescript powered wrapper for the Hugging Face Inference API. Learn more about the Inference API at [Hugging Face](https://huggingface.co/docs/api-inference/index). It also works with [Inference Endpoints](https://huggingface.co/docs/inference-endpoints/index).

Check out the [full documentation](https://huggingface.co/docs/huggingface.js/inference/README).

You can also try out a live [interactive notebook](https://observablehq.com/@huggingface/hello-huggingface-js-inference) or see some demos on [hf.co/huggingfacejs](https://huggingface.co/huggingfacejs).

## Install

```console
npm install @huggingface/inference

yarn add @huggingface/inference

pnpm add @huggingface/inference
```

## Usage

❗**Important note:** Using an access token is optional to get started, however you will be rate limited eventually. Join [Hugging Face](https://huggingface.co/join) and then visit [access tokens](https://huggingface.co/settings/tokens) to generate your access token for **free**.

Your access token should be kept private. If you need to protect it in front-end applications, we suggest setting up a proxy server that stores the access token.

### Basic examples

```typescript
import { HfInference } from '@huggingface/inference'

const hf = new HfInference('your access token')

// Natural Language

await hf.fillMask({
  model: 'bert-base-uncased',
  inputs: '[MASK] world!'
})

await hf.summarization({
  model: 'facebook/bart-large-cnn',
  inputs:
    'The tower is 324 metres (1,063 ft) tall, about the same height as an 81-storey building, and the tallest structure in Paris. Its base is square, measuring 125 metres (410 ft) on each side. During its construction, the Eiffel Tower surpassed the Washington Monument to become the tallest man-made structure in the world, a title it held for 41 years until the Chrysler Building in New York City was finished in 1930.',
  parameters: {
    max_length: 100
  }
})

await hf.questionAnswering({
  model: 'deepset/roberta-base-squad2',
  inputs: {
    question: 'What is the capital of France?',
    context: 'The capital of France is Paris.'
  }
})

await hf.tableQuestionAnswering({
  model: 'google/tapas-base-finetuned-wtq',
  inputs: {
    query: 'How many stars does the transformers repository have?',
    table: {
      Repository: ['Transformers', 'Datasets', 'Tokenizers'],
      Stars: ['36542', '4512', '3934'],
      Contributors: ['651', '77', '34'],
      'Programming language': ['Python', 'Python', 'Rust, Python and NodeJS']
    }
  }
})

await hf.textClassification({
  model: 'distilbert-base-uncased-finetuned-sst-2-english',
  inputs: 'I like you. I love you.'
})

await hf.textGeneration({
  model: 'gpt2',
  inputs: 'The answer to the universe is'
})

for await (const output of hf.textGenerationStream({
  model: "google/flan-t5-xxl",
  inputs: 'repeat "one two three four"',
  parameters: { max_new_tokens: 250 }
})) {
  console.log(output.token.text, output.generated_text);
}

await hf.tokenClassification({
  model: 'dbmdz/bert-large-cased-finetuned-conll03-english',
  inputs: 'My name is Sarah Jessica Parker but you can call me Jessica'
})

await hf.translation({
  model: 't5-base',
  inputs: 'My name is Wolfgang and I live in Berlin'
})

await hf.zeroShotClassification({
  model: 'facebook/bart-large-mnli',
  inputs: [
    'Hi, I recently bought a device from your company but it is not working as advertised and I would like to get reimbursed!'
  ],
  parameters: { candidate_labels: ['refund', 'legal', 'faq'] }
})

await hf.conversational({
  model: 'microsoft/DialoGPT-large',
  inputs: {
    past_user_inputs: ['Which movie is the best ?'],
    generated_responses: ['It is Die Hard for sure.'],
    text: 'Can you explain why ?'
  }
})

await hf.sentenceSimilarity({
  model: 'sentence-transformers/paraphrase-xlm-r-multilingual-v1',
  inputs: {
    source_sentence: 'That is a happy person',
    sentences: [
      'That is a happy dog',
      'That is a very happy person',
      'Today is a sunny day'
    ]
  }
})

await hf.featureExtraction({
  model: "sentence-transformers/distilbert-base-nli-mean-tokens",
  inputs: "That is a happy person",
});

// Audio

await hf.automaticSpeechRecognition({
  model: 'facebook/wav2vec2-large-960h-lv60-self',
  data: readFileSync('test/sample1.flac')
})

await hf.audioClassification({
  model: 'superb/hubert-large-superb-er',
  data: readFileSync('test/sample1.flac')
})

// Computer Vision

await hf.imageClassification({
  data: readFileSync('test/cheetah.png'),
  model: 'google/vit-base-patch16-224'
})

await hf.objectDetection({
  data: readFileSync('test/cats.png'),
  model: 'facebook/detr-resnet-50'
})

await hf.imageSegmentation({
  data: readFileSync('test/cats.png'),
  model: 'facebook/detr-resnet-50-panoptic'
})

await hf.textToImage({
  inputs: 'award winning high resolution photo of a giant tortoise/((ladybird)) hybrid, [trending on artstation]',
  model: 'stabilityai/stable-diffusion-2',
  parameters: {
    negative_prompt: 'blurry',
  }
})

await hf.imageToText({
  data: readFileSync('test/cats.png'),
  model: 'nlpconnect/vit-gpt2-image-captioning'
})

// Multimodal

await hf.visualQuestionAnswering({
  model: 'dandelin/vilt-b32-finetuned-vqa',
  inputs: {
    question: 'How many cats are lying down?',
    image: await (await fetch('https://placekitten.com/300/300')).blob()
  }
})

await hf.documentQuestionAnswering({
  model: 'impira/layoutlm-document-qa',
  inputs: {
    question: 'Invoice number?',
    image: await (await fetch('https://huggingface.co/spaces/impira/docquery/resolve/2359223c1837a7587402bda0f2643382a6eefeab/invoice.png')).blob(),
  }
})

// Custom call, for models with custom parameters / outputs
await hf.request({
  model: 'my-custom-model',
  inputs: 'hello world',
  parameters: {
    custom_param: 'some magic',
  }
})

// Custom streaming call, for models with custom parameters / outputs
for await (const output of hf.streamingRequest({
  model: 'my-custom-model',
  inputs: 'hello world',
  parameters: {
    custom_param: 'some magic',
  }
})) {
  ...
}

// Using your own inference endpoint: https://hf.co/docs/inference-endpoints/
const gpt2 = hf.endpoint('https://xyz.eu-west-1.aws.endpoints.huggingface.cloud/gpt2');
const { generated_text } = await gpt2.textGeneration({inputs: 'The answer to the universe is'});
```

## Supported Tasks

### Natural Language Processing

- [x] Fill mask
- [x] Summarization
- [x] Question answering
- [x] Table question answering
- [x] Text classification
- [x] Text generation
- [x] Text2Text generation
- [x] Token classification
- [x] Named entity recognition
- [x] Translation
- [x] Zero-shot classification
- [x] Conversational
- [x] Feature extraction
- [x] Sentence Similarity

### Audio

- [x] Automatic speech recognition
- [x] Audio classification

### Computer Vision

- [x] Image classification
- [x] Object detection
- [x] Image segmentation
- [x] Text to image
- [x] Image to text

### Multimodal
- [x] Document question answering
- [x] Visual question answering

## Tree-shaking

You can import the functions you need directly from the module, rather than using the `HfInference` class:

```ts
import {textGeneration} from "@huggingface/inference";

await textGeneration({
  accessToken: "hf_...",
  model: "model_or_endpoint",
  inputs: ...,
  parameters: ...
})
```

This will enable tree-shaking by your bundler.

## Running tests

```console
HF_ACCESS_TOKEN="your access token" npm run test
```

## Finding appropriate models

We have an informative documentation project called [Tasks](https://huggingface.co/tasks) to list available models for each task and explain how each task works in detail.

It also contains demos, example outputs, and other resources should you want to dig deeper into the ML side of things.
