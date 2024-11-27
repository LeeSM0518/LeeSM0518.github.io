---
title: OpenAI API Guide
date: 2024-11-25 13:00:00 +0900
categories: openai
tags:
  - openai
  - docs
  - api
description: OpenAI의 API를 활용하기 위한 가이드 문서 정리
---
## 1. 빠른 시작

### 1.1. 개발자를 위한 빠른 시작

OpenAI API는 자연어 처리, 이미지 생성, 의미 검색, 음성 인식을 위한 API를 제공한다.

다음 과정을 통해 첫 번째 API 요청을 하는 방법을 알아보자.

#### 1.1.1. [Billing 페이지](https://platform.openai.com/settings/organization/billing/overview)에서 카드를 등록한다.

![chatgpt4](/assets/img/chatgpt4.png)

![chatgpt5](/assets/img/chatgpt5.png)

#### 1.1.2. [Dashboard 페이지](https://platform.openai.com/api-keys)에서 키를 생성한다.
![chatgpt1](/assets/img/chatgpt1.png)
![chatgpt2](/assets/img/chatgpt2.png)
![chatgpt3](/assets/img/chatgpt3.png)

#### 1.1.3. 명령어를 통해 API를 호출한다.

```bash
curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "model": "gpt-4o-mini",
        "messages": [
            {
                "role": "system",
                "content": "You are a helpful assistant."
            },
            {
                "role": "user",
                "content": "Write a haiku that explains the concept of recursion."
            }
        ]
    }'
```
{: file='텍스트 생성 명령어'}

```json
{
  "id": "chatcmpl-AXVtElkFhR88Y8CZFDSDcdfRjbvBw",
  "object": "chat.completion",
  "created": 1732550772,
  "model": "gpt-4o-mini-2024-07-18",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Nested calls unfold,  \nEach step mirrors the last—  \nSelf-reflecting paths.",
        "refusal": null
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 28,
    "completion_tokens": 18,
    "total_tokens": 46,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "system_fingerprint": "fp_0705bf87c0"
}
```
{: file='텍스트 생성 결과'}

<br/>

```bash
curl "https://api.openai.com/v1/images/generations" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "prompt": "A cute baby sea otter",
        "n": 2,
        "size": "1024x1024"
    }'
```
{: file='이미지 생성 명령어'}

```json
{
  "created": 1732585271,
  "data": [
    {
      "url": "${결과 이미지 URL}"
    },
    {
      "url": "${결과 이미지 URL}"
    }
  ]
}
```
{: file='이미지 생성 결과'}

<br/>

```bash
curl "https://api.openai.com/v1/embeddings" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "input": "The food was delicious and the waiter...",
        "model": "text-embedding-3-large"
    }'
```
{: file='벡터 임베딩 생성'}

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [
	    -0.011019929,
	  ],
    },
  ],
  "model": "text-embedding-3-large",
  "usage": {
	"prompt_tokens": 8,
	"total_tokens": 8
  }
}
```
{: file='벡터 임베딩 결과'}

<br/>

## 2. 모델

### 2.1. 주요 모델

- GPT-4o
  - 텍스트 및 이미지 입력, 텍스트 출력
  - 128k 컨텍스트 길이
  - 더 스마트한 모델, 토큰당 높은 가격
- GPT-4o mini
  - 텍스트 및 이미지 입력, 텍스트 출력
  - 128k 컨텍스트 길이
  - 더 빠른 모델, 토큰당 낮은 가격
- o1-preview & o1-mini
  - 텍스트 입력, 텍스트 출력
  - 128k 컨텍스트 길이
  - 더 높은 지연 시간, 토큰을 사용하여 생각

### 2.2. 모델 개요

|모델|설명|
|---|---|
|[GPT-4o](https://platform.openai.com/docs/models#gpt-4o)|복잡하고 여러 단계로 구성된 작업을 위한 고지능 모델|
|[GPT-4o mini](https://platform.openai.com/docs/models#gpt-4o-mini)|빠르고 가벼운 작업을 위한 저렴하고 지능적인 소형 모델|
|[o1-preview and o1-mini](https://platform.openai.com/docs/models#o1)|복잡한 추론을 수행하기 위해 강화 학습으로 훈련된 언어 모델|
|[GPT-4 Turbo and GPT-4](https://platform.openai.com/docs/models#gpt-4-turbo-and-gpt-4)|이전 고지능 모델 세트|
|[GPT-3.5 Turbo](https://platform.openai.com/docs/models#gpt-3-5-turbo)|간단한 작업을 위한 빠르고 저렴한 모델|
|[DALL·E](https://platform.openai.com/docs/models#dall-e)|자연어 프롬프트가 주어지면 이미지를 생성하고 편집할 수 있는 모델|
|[TTS](https://platform.openai.com/docs/models#tts)|텍스트를 자연스러운 음성 오디오로 변환할 수 있는 모델 세트|
|[Whisper](https://platform.openai.com/docs/models#whisper)|오디오를 텍스트로 변환할 수 있는 모델|
|[Embeddings](https://platform.openai.com/docs/models#embeddings)|텍스트를 숫자 형태로 변환할 수 있는 모델 세트|
|[Moderation](https://platform.openai.com/docs/models#moderation)|텍스트가 민감하거나 안전하지 않을 수 있는지 감지할 수 있는 미세 조정된 모델|
|[Deprecated](https://platform.openai.com/docs/deprecations)|더 이상 사용되지 않는 모델의 전체 목록과 제안된 대체 모델|

### 2.3. GPT-4o

복잡하고 여러 단계의 작업을 위한 고지능 모델

- 최대 입력 토큰 : 128,000
- 최대 출력 토큰 : 16,384

### 2.4. GPT-4o mini

빠르고 가벼운 작업을 위한 저렴하고 지능적인 소형 모델

- 최대 입력 토큰 : 128,000
- 최대 출력 토큰 : 16,384

### 2.5. GPT-4o Realtime + Audio

WebSocket 인터페이스를 통해 오디오 및 텍스트 입력에 응답할 수 있다.

- 최대 입력 토큰 : 128,000
- 최대 출력 토큰 : 4,096

### 2.6. o1-preview and o1-mini

복잡한 추론을 수행하기 위해 강화 학습으로 훈련된 모델

- o1-preview
  - 여러 도메인에 걸쳐 어려운 문제를 해결하는 추론 모델
  - 최대 입력 토큰 : 128,000
  - 최대 출력 토큰 : 32,768
- o1-mini
  - 코딩, 수학, 과학에 특히 적합한 더 빠르고 저렴한 추론 모델
  - 최대 입력 토큰 : 128,000
  - 최대 출력 토큰 : 65,536

### 2.7. GPT-4 Turbo and GPT-4

- GPT-4 Turbo
  - 최대 입력 토큰 : 128,000
  - 최대 출력 토큰 : 4,096
- GPT-4
  - 최대 입력 토큰 : 8,192
  - 최대 출력 토큰 : 8,192

### 2.8. DALL·E

- DALL·E 3
  - 특정 크기의 새 이미지를 만드는 기능을 제공
- DALL·E 2
  - 기존 이미지를 편집하거나 사용자가 제공한 이미지의 변형을 만드는 기능도 지원

### 2.9. TTS

텍스트를 자연스러운 음성 텍스트로 변환하는 AI 모델

- tts-1 : 속도에 최적화된 최신 텍스트 음성 변환 모델
- tts-1-hd : 품질이 최적화된 최신 텍스트 음성 변환 모델

### 2.10. Whisper

범용 음성 인식 모델. 다국어 음성 인식과 음성 번역 및 언어 식별을 수행하는 모델.

### 2.11. Embeddings

임베딩은 두 텍스트 간의 관련성을 측정하는 데 사용할 수 있는 텍스트의 수치적 표현이다. 검색, 클러스터링, 추천, 이상 감지 및 분류 작업에 유용하다.

### 2.12. Moderation

증오, 자해, 성적 콘텐츠, 폭력 등의 범주에서 콘텐츠를 찾는 분류 기능을 제공한다.

### 2.13. Model endpoint

|Endpoint|Latest models|
|---|---|
|/v1/assistants|All GPT-4o (except `chatgpt-4o-latest`), GPT-4o-mini, GPT-4, and GPT-3.5 Turbo models. The `retrieval` tool requires `gpt-4-turbo-preview` (and subsequent dated model releases) or `gpt-3.5-turbo-1106` (and subsequent versions).|
|/v1/audio/transcriptions|`whisper-1`|
|/v1/audio/translations|`whisper-1`|
|/v1/audio/speech|`tts-1`,  `tts-1-hd`|
|/v1/chat/completions|All GPT-4o (except for Realtime preview), GPT-4o-mini, GPT-4, and GPT-3.5 Turbo models and their dated releases. `chatgpt-4o-latest` dynamic model. [Fine-tuned](https://platform.openai.com/docs/guides/fine-tuning) versions of `gpt-4o`,  `gpt-4o-mini`,  `gpt-4`,  and `gpt-3.5-turbo`.|
|/v1/completions (Legacy)|`gpt-3.5-turbo-instruct`,  `babbage-002`,  `davinci-002`|
|/v1/embeddings|`text-embedding-3-small`,  `text-embedding-3-large`,  `text-embedding-ada-002`|
|/v1/fine_tuning/jobs|`gpt-4o`,  `gpt-4o-mini`,  `gpt-4`,  `gpt-3.5-turbo`|
|/v1/moderations|`text-moderation-stable`,  `text-moderation-latest`|
|/v1/images/generations|`dall-e-2`,  `dall-e-3`|
|/v1/realtime (beta)|`gpt-4o-realtime-preview`, `gpt-4o-realtime-preview-2024-10-01`|

<br/>

## 3. 글 생성

프롬프트로 텍스트를 생성하는 방법에 대해 알아보자.

### 3.1. 빠른 시작

다음과 같이 REST API를 호출할 수 있다.

#### 3.1.1. 산문 생성

```bash
curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "model": "gpt-4o",
        "messages": [
            {
                "role": "system",
                "content": "You are a helpful assistant."
            },
            {
                "role": "user",
                "content": "Write a haiku about recursion in programming."
            }
        ]
    }'
```
{: file='산문 생성 명령어'}

```json
{
  "id": "chatcmpl-AXfn5QIhqcUtMgkMz0qniRy73rkDL",
  "object": "chat.completion",
  "created": 1732588831,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Code calls on itself,  \nInfinite loops intertwine.  \nLogic finds a way.",
        "refusal": null
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 26,
    "completion_tokens": 17,
    "total_tokens": 43,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "system_fingerprint": "fp_831e067d82"
}
```
{: file='산문 생성 결과'}

<br/>

#### 3.1.2. 이미지 분석

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "text",
            "text": "What is in this image?"
          },
          {
            "type": "image_url",
            "image_url": {
              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
            }
          }
        ]
      }
    ]
  }'
```
{: file='이미지 분석 명령어'}

```json
{
  "id": "chatcmpl-AXkB29sBylL0SicUNIqp6l1ViYJg9",
  "object": "chat.completion",
  "created": 1732605692,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The image depicts a scenic landscape with a wooden boardwalk leading through a grassy field. The sky is blue with some clouds, and trees and bushes are visible in the distance. The overall scene has a peaceful and open atmosphere.",
        "refusal": null
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 1118,
    "completion_tokens": 45,
    "total_tokens": 1163,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "system_fingerprint": "fp_831e067d82"
}
```
{: file='이미지 분석 결과'}

<br/>

#### 3.1.3. JSON 데이터 생성

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o-2024-08-06",
    "messages": [
      {
        "role": "system",
        "content": "You extract email addresses into JSON data."
      },
      {
        "role": "user",
        "content": "Feeling stuck? Send a message to help@mycompany.com."
      }
    ],
    "response_format": {
      "type": "json_schema",
      "json_schema": {
        "name": "email_schema",
        "schema": {
            "type": "object",
            "properties": {
                "email": {
                    "description": "The email address that appears in the input",
                    "type": "string"
                }
            },
            "additionalProperties": false
        }
      }
    }
  }'
```
{: file='JSON 데이터 생성 명령어'}

```json
{
  "id": "chatcmpl-AXkH56VM3zY0QyMTeMaxZZw3VyE9j",
  "object": "chat.completion",
  "created": 1732606067,
  "model": "gpt-4o-2024-08-06",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "{\"email\":\"help@mycompany.com\"}",
        "refusal": null
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 69,
    "completion_tokens": 9,
    "total_tokens": 78,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "system_fingerprint": "fp_7f6be3efb0"
}
```
{: file='JSON 데이터 생성 결과'}

<br/>

### 3.2. 모델 선택

텍스트 생성을 요청하기 전에 해야 할 것은 응답을 생성할 모델을 선택하는 것이다.

- `gpt-4o` : 토큰당 비용이 높지만 높은 수준의 지능과 강력한 성능을 제공한다.
- `gpt-4o-mini` : 더 빠르고 비용이 저렴하다.
- `o1` : 결과를 반환하는 속도가 느리고 더 많은 토큰을 사용하지만, 고급 추론 이나 코딩 및 다단계 계획이 가능하다.

[Playground](https://platform.openai.com/playground)에서 다양한 모델을 실험해 보고 어떤 모델이 적합한지 확인할 수 있다.

### 3.3. 프롬프트 구성

 모델에서 올바른 출력을 얻기 위해 프롬프트를 만드는 과정을 프롬프트 엔지니어링 이라고 한다. 모델에 정확한 지침을 제공하면 모델 출력의 품질과 정확도를 개선할 수 있다.

chat completions API에서는 모델에 대한 지침이 포함된 `meesage` 배열을 제공하여 프롬프트를 생성한다. 각 메시지는 서로 다른 `role` 을 가질 수 있으며, 이는 모델이 입력을 해석하는 방법에 영향을 미친다.

#### 3.3.1. 사용자 메시지 (`user`)

User 메시지에는 모델에 특정 유형의 출력을 요청하는 지침이 포함되어 있다. `user` 메시지는 최종 사용자가 ChatGPT에 입력하는 메시지라고 생각하면 된다.

다음은 gpt-4o 모델이 프롬프트에 따라 시를 생성하도록 요청하는 사용자 메시지 프롬프트 예시이다.

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Write a haiku about programming."
        }
      ]
    }
  ]
});
```


#### 3.3.2. 시스템 메시지 (`system`)

`system` 메시지는 모델에 대한 최상위 수준의 지침 역할을 하며, 일반적으로 모델이 무엇을 해야 하는지, 그리고 일반적으로 어떻게 동작하고 응답해야 하는지를 설명한다.

다음은 `user` 메시지에 대한 응답을 생성할 때 모델의 동작을 수정하는 시스템 메시지의 예이다.

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": `
            You are a helpful assistant that answers programming questions 
            in the style of a southern belle from the southeast United States.
          `
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "Are semicolons optional in JavaScript?"
        }
      ]
    }
  ]
});
```

#### 3.3.3. 보조 메시지 (`assistant`)

`assistant` 메시지는 모델에 의해 생성된 메시지이다. 또한 요청에 어떻게 응답해야 하는지에 대한 예를 모델에 제공하는 데 사용할 수도 있다.

이전 텍스트 생성 결과를 캡처하고, 그에 따라 새로운 요청을 만드는데 보조 메시지를 사용할 수 있다.

```javascript
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [
    {
      "role": "user",
      "content": [{ "type": "text", "text": "knock knock." }]
    },
    {
      "role": "assistant",
      "content": [{ "type": "text", "text": "Who's there?" }]
    },
    {
      "role": "user",
      "content": [{ "type": "text", "text": "Orange." }]
    }
  ]
});
```

<br/>

## 4. 이미지 생성

DALL·E 활용하여 이미지를 생성하거나 수정하는 방법을 알아보자.

### 4.1. 소개

이미지 API는 이미지와 상호작용하기 위한 세 가지 방법을 제공한다.

1. 텍스트 프롬프트를 기반으로 이미지 생성 (DALL·E 3, DALL·E 2)
2. 텍스트 프롬프트를 기반으로 기존 이미지의 일부 영역을 모델로 대체하여 편집된 이미지 생성 (DALL·E 2)
3. 기존 이미지의 변형 생성 (DALL·E 2)

### 4.2. 사용법

#### 4.2.1. 생성

이미지 생성 엔드포인트를 사용하면 텍스트 프롬프트가 주어진 원본 이미지를 만들 수 있다. DALL·E 3을 사용할 때 이미지 크기는 1024x1024, 1024x1792, 1792x1024 가 될 수 있다.

기본적으로 이미지는 `standard` 품질로 생성되지만 DALL·E 3을 사용할 때는 향상된 디테일을 위해 `hd` 품질을 설정할 수 있다. 

DALL·E 3을 사용하면 한 번에 1개의 이미지를 요청할 수 있으며 DALL·E 2를 n 매개변수와 함께 사용하면 한 번에 최대 10개의 이미지를 요청할 수 있다.

```bash
curl https://api.openai.com/v1/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "dall-e-3",
    "prompt": "a white siamese cat",
    "n": 1,
    "size": "1024x1024"
  }'
```
{: file='이미지 생성 명령어'}

```json
{
  "created": 1732684084,
  "data": [
    {
      "revised_prompt": "Imagine an elegantly poised Siamese cat, renowned for its distinctive, short, cream-colored fur. Its face, paws, and tail exhibit the characteristic dark points, striking a stunning contrast with the rest of its body. The cat's deep blue almond-shaped eyes are piercing and expressive, reflecting an air of wisdom and curiosity. Its slender and agile body with graceful, long limbs, is perfected by a delicately tapering tail. The Siamese cat's ears are set high on its head, accentuating its triangular face. The cat poses with an air of royal calmness yet readiness, characteristic of the breed.",
      "url": "{이미지결과URL}"
    }
  ]
}
```
{: file='이미지 생성 결과 (JSON)'}

- URL은 1시간 후에 만료된다.

![chatgpt6](/assets/img/chatgpt6.png)
_이미지 생성 결과 (PNG)_

<br/>

#### 4.2.2. 편집 (DALL·E 2 only)

이미지 편집 엔드포인트를 사용하면 이미지를 업로드하고 어떤 영역을 바꿔야 하는지 나타내는 마스크를 지정하여 이미지를 편집하거나 확장할 수 있다. 투명한 영역은 이미지를 편집해야 하는 위치를 나타내며, 프롬프트는 지워진 영역만이 아니라 이미지 전체를 설명해야 한다.

업로드한 이미지와 마스크는 모두 크기가 4MB 미만인 정사각형 PNG 이미지여야 하며, 서로 동일한 크기를 가져야 한다.


```bash
curl https://api.openai.com/v1/images/edits \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F model="dall-e-2" \
  -F image="@sunlit_lounge.png" \
  -F mask="@mask.png" \
  -F prompt="A sunlit indoor lounge area with a pool containing a flamingo" \
  -F n=1 \
  -F size="1024x1024"
```
{: file='이미지 편집 명령어'}

![chatgpt7](/assets/img/chatgpt7.png)
_이미지 편집 결과_

#### 4.2.3. 변형 (DALL·E 2 only)

이미지 변형 엔드포인트를 사용하면 주어진 이미지의 변형을 생성할 수 있다. 입력 이미지는 크기가 4MB 미만인 정사각형 PNG 이미지여야 한다.

```bash
curl https://api.openai.com/v1/images/variations \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F model="dall-e-2" \
  -F image="@corgi_and_cat_paw.png" \
  -F n=1 \
  -F size="1024x1024"
```
{: file='이미지 변형 명령어'}

![chatgpt8](/assets/img/chatgpt8.png)
_이미지 변형 결과_

<br/>

## 5. 이미지 분석

비전 기능을 사용하여 이미지를 분석하는 방법을 알아보자.

### 5.1. 빠른 시작

`user` 메시지를 통해 질문과 이미지(base64, URL)를 전달할 수 있다.

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "text",
            "text": "What’s in this image?"
          },
          {
            "type": "image_url",
            "image_url": {
              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
            }
          }
        ]
      }
    ],
    "max_tokens": 300
  }'
```
{: file='이미지 분석 명령어'}

```json
{
  "id": "chatcmpl-AY4sJI9oMSkupRm9iyb18SksCpOkv",
  "object": "chat.completion",
  "created": 1732685255,
  "model": "gpt-4o-mini-2024-07-18",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The image depicts a serene landscape featuring a wooden walkway or boardwalk running through a lush, green field. The field is filled with tall grass and bordered by trees and bushes. The sky above is bright with a few clouds, suggesting a clear day. The scene conveys a sense of tranquility and natural beauty.",
        "refusal": null
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 36848,
    "completion_tokens": 61,
    "total_tokens": 36909,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "system_fingerprint": "fp_3de1288069"
}
```
{: file='이미지 분석 결과'}

### 5.2. 다중 이미지 입력

여러 이미지와 질문을 통해 분석 결과를 답변 받을 수 있다.

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "text",
            "text": "What are in these images? Is there any difference between them?"
          },
          {
            "type": "image_url",
            "image_url": {
              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
            }
          },
          {
            "type": "image_url",
            "image_url": {
              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
            }
          }
        ]
      }
    ],
    "max_tokens": 300
  }'
```
{: file='다중 이미지 입력 명령어'}

<br/>

## 6. 오디오 생성

텍스트나 오디오 프롬프트로 오디오를 생성하는 방법에 대해 알아보자.

오디오 생성 기능을 사용하여 다음을 수행할 수 있다.
1. 본문 텍스트의 음성 오디오 요약 생성 (텍스트 입력, 오디오 출력)
2. 녹음에 대한 감정 분석 수행 (오디오 입력, 텍스트 출력)
3. 모델과의 비동기 음성 대 음성 상호 작용 (오디오 입력, 오디오 출력)

### 6.1. 빠른 시작

다음과 같이 오디오 생성 엔드포인트를 호출하면 된다.

```bash
curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
      "model": "gpt-4o-audio-preview",
      "modalities": ["text", "audio"],
      "audio": { "voice": "alloy", "format": "wav" },
      "messages": [
        {
          "role": "user",
          "content": "Is a golden retriever a good family dog?"
        }
      ]
    }'
```
{: file='오디오 생성 명령어'}

```json
{  
  "id": "chatcmpl-AY59JQQp7S7j6n0FCFAkzDK8RSMa4",  
  "object": "chat.completion",  
  "created": 1732686309,  
  "model": "gpt-4o-audio-preview-2024-10-01",  
  "choices": [  
    {  
      "index": 0,  
      "message": {  
        "role": "assistant",  
        "content": null,  
        "refusal": null,  
        "audio": {  
          "id": "audio_6746b1e7e9e08190bcc4e0b746c9d5f7",  
          "data": "${오디오파일}",  
          "expires_at": 1732689911,  
          "transcript": "Yes, golden retrievers are known for their friendly and gentle temperament, which makes them great family dogs. They are typically good with children, loyal, and eager to please, which makes them easy to train and a wonderful addition to family life."  
        }  
      },  
      "finish_reason": "stop"  
    }  
  ],  
  "usage": {  
    "prompt_tokens": 17,  
    "completion_tokens": 369,  
    "total_tokens": 386,  
    "prompt_tokens_details": {  
      "cached_tokens": 0,  
      "audio_tokens": 0,  
      "text_tokens": 17,  
      "image_tokens": 0  
    },  
    "completion_tokens_details": {  
      "reasoning_tokens": 0,  
      "audio_tokens": 299,  
      "accepted_prediction_tokens": 0,  
      "rejected_prediction_tokens": 0,  
      "text_tokens": 70  
    }  
  },  
  "system_fingerprint": "fp_130ac2f073"  
}
```
{: file='오디오 생성 결과'}

### 6.2. 여러 차례 대화

다음과 같이 `message.audio.id` 값을 전달하여 대화를 수행할 수 있다.

```bash
curl "https://api.openai.com/v1/chat/completions" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -d '{
        "model": "gpt-4o-audio-preview",
        "modalities": ["text", "audio"],
        "audio": { "voice": "alloy", "format": "wav" },
        "messages": [
            {
                "role": "user",
                "content": "Is a golden retriever a good family dog?"
            },
            {
                "role": "assistant",
                "audio": {
                    "id": "audio_abc123"
                }
            },
            {
                "role": "user",
                "content": "Why do you say they are loyal?"
            }
        ]
    }'
```

<br/>

## 7. 텍스트 음성 변환

텍스트를 음성 오디오로 변환하는 방법을 알아보자.

### 7.1. 개요

오디오 API는 TTS(텍스트 음성 변환) 모델을 기반으로 한 음성 엔드포인트를 제공한다.

> OpenAI의 이용 정책에 따라 최종 사용자에게 들리는 TTS 음성이 인간의 음성이 아니라 AI가 생성한 음성이라는 점을 명확하게 공개해야 한다.
{: .prompt-warning }

### 7.2. 빠른 시작

음성 엔드포인트는 세 가지 입력을 받는다. 모델, 오디오로 변환해야 하는 텍스트, 오디오 생성에 사용할 음성이다. 요청은 다음과 같다.

```bash
curl https://api.openai.com/v1/audio/speech \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tts-1",
    "input": "Today is a wonderful day to build something people love!",
    "voice": "alloy"
  }' \
  --output speech.mp3
```
{: file='텍스트 음성 변환 명령어'}

#### 7.2.1. 오디오 품질

실시간 애플리케이션의 경우 표준 tts-1 모델은 가장 낮은 지연 시간을 제공하지만 tts-1-hd 모델보다 품질은 낮다. 

#### 7.2.2. 음성 옵션

음성은 alloy(남성1), echo(남성2), fable(남성3), onyx(남성4), nova(여성1), shimmer(여성2)을 제공한다.

#### 7.2.3. 출력 형식

기본 응답 형식은 "mp3" 이지만 "opus", "aac", "flac", "pcm" 등 다른 형식도 사용할 수 있다.

<br/>

## 8. 음성을 텍스트로 변환

오디오를 텍스트로 변환하는 방법을 알아보자.

### 8.1. 빠른 시작

파일 업로드는 현재 25MB로 제한되어 있으며 다음과 같은 입력 파일 유형이 지원된다.
- mp3, mp4, mpeg, mpga, m4a, wav, webm

#### 8.1.1. Transcriptions (변환)

다음과 같이 변환하려는 오디오 파일과 출력 파일 형식을 입력으로 전달한다.

```bash
curl --request POST \  
  --url https://api.openai.com/v1/audio/transcriptions \  
  --header "Authorization: Bearer $OPENAI_API_KEY" \  
  --form "file=@${파일경로}" \  
  --form model=whisper-1
```
{: file='변환 명령어'}

```json
{
  "text": "실시간 애플리케이션의 경우 표준 TTS1 모델은 가장 낮은 지연 시간을 제공하지만 TTS1 HD 모델보다 품질은 낮다."
}
```
{: file='변환 결과'}

<br/>

## 9. 함수 호출

함수 호출(Function calling)을 통해 개발자는 언어 모델을 외부 데이터 및 시스템에 연결할 수 있다.

### 9.1. 개요

애플리케이션에서 작업을 트리거하거나 외부 시스템과 상호 작용하기 위해 사용자 정의 함수를 호출하는 모델이 필요하다.

모델에서 사용할 도구로 함수를 정의하는 방법은 다음과 같다.

```javascript
import { OpenAI } from "openai";
const openai = new OpenAI();

const tools = [
    {
        type: "function",
        function: {
            name: "get_weather",
            parameters: {
                type: "object",
                properties: {
                    location: { type: "string" },
                    unit: { type: "string", enum: ["c", "f"] },
                },
                required: ["location", "unit"],
                additionalProperties: false,
            },
        },
    }
];

const response = await openai.chat.completions.create({
    model: "gpt-4o",
    messages: [{"role": "user", "content": "What's the weather like in Paris today?"}],
    tools,
});

console.log(response.choices[0].message.tool_calls);
```
{: file='노드 기반 함수 호출 예시'}

함수 호출이 쓰일 수 있는 예시는 다음과 같다.
1. 데이터 가져오기 : 대화형 도우미가 사용자에게 응답하기 전에 내부 시스템에서 데이터를 검색할 수 있도록 한다.
2. 조치 실행 : 비서가 대화에 따라 회의 일정을 잡거나 주문한 상품을 반품하는 등의 조치를 취할 수 있도록 한다.
3. 워크플로우 구축 : 데이터 추출 파이프라인이나 콘텐츠 개인화와 같은 워크플로우를 실행할 수 있도록 한다.
4. UI와 상호작용 : 지도에 핀을 렌더링하거나 웹사이트를 탐색하는 것처럼 사용자 입력을 기반으로 UI를 업데이트 할 수 있다.

<br/>

## 10. 구조화된 출력

### 10.1. 소개

모델이 JSON 스키마를 준수하는 응답을 생성하도록 보장하는 기능이므로 모델이 필수 키를 생략하거나 잘못된 열거형 값을 응답하는 것을 걱정할 필요없다.

```javascript
import OpenAI from "openai";
import { zodResponseFormat } from "openai/helpers/zod";
import { z } from "zod";

const openai = new OpenAI();

const CalendarEvent = z.object({
  name: z.string(),
  date: z.string(),
  participants: z.array(z.string()),
});

const completion = await openai.beta.chat.completions.parse({
  model: "gpt-4o-2024-08-06",
  messages: [
    { role: "system", content: "Extract the event information." },
    { role: "user", content: "Alice and Bob are going to a science fair on Friday." },
  ],
  response_format: zodResponseFormat(CalendarEvent, "event"),
});

const event = completion.choices[0].message.parsed;
```

### 10.2. 예시

#### 10.2.1. 단계적 설명

사용자가 솔루션을 탐색할 수 있도록 안내하기 위해 모델에 체계적이고 단계별로 답변을 출력하도록 요청할 수 있다,

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-2024-08-06",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful math tutor. Guide the user through the solution step by step."
      },
      {
        "role": "user",
        "content": "how can I solve 8x + 7 = -23"
      }
    ],
    "response_format": {
      "type": "json_schema",
      "json_schema": {
        "name": "math_reasoning",
        "schema": {
          "type": "object",
          "properties": {
            "steps": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "explanation": { "type": "string" },
                  "output": { "type": "string" }
                },
                "required": ["explanation", "output"],
                "additionalProperties": false
              }
            },
            "final_answer": { "type": "string" }
          },
          "required": ["steps", "final_answer"],
          "additionalProperties": false
        },
        "strict": true
      }
    }
  }'
```
{: file='단계적 설명 명령어'}

```json
{
  "steps": [
    {
      "explanation": "Start with the equation 8x + 7 = -23.",
      "output": "8x + 7 = -23"
    },
    {
      "explanation": "Subtract 7 from both sides to isolate the term with the variable.",
      "output": "8x = -23 - 7"
    },
    {
      "explanation": "Simplify the right side of the equation.",
      "output": "8x = -30"
    },
    {
      "explanation": "Divide both sides by 8 to solve for x.",
      "output": "x = -30 / 8"
    },
    {
      "explanation": "Simplify the fraction.",
      "output": "x = -15 / 4"
    }
  ],
  "final_answer": "x = -15 / 4"
}
```
{: file='단계적 설명 결과'}

#### 10.2.2. 사용자 입력에 대한 검토

여러 카테고리에 따라 입력을 분류할 수 있으며, 이는 검토를 수행하는 일반적인 방법이다.

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-2024-08-06",
    "messages": [
      {
        "role": "system",
        "content": "Determine if the user input violates specific guidelines and explain if they do."
      },
      {
        "role": "user",
        "content": "How do I prepare for a job interview?"
      }
    ],
    "response_format": {
      "type": "json_schema",
      "json_schema": {
        "name": "content_compliance",
        "description": "Determines if content is violating specific moderation rules",
        "schema": {
          "type": "object",
          "properties": {
            "is_violating": {
              "type": "boolean",
              "description": "Indicates if the content is violating guidelines"
            },
            "category": {
              "type": ["string", "null"],
              "description": "Type of violation, if the content is violating guidelines. Null otherwise.",
              "enum": ["violence", "sexual", "self_harm"]
            },
            "explanation_if_violating": {
              "type": ["string", "null"],
              "description": "Explanation of why the content is violating"
            }
          },
          "required": ["is_violating", "category", "explanation_if_violating"],
          "additionalProperties": false
        },
        "strict": true
      }
    }
  }'
```
{: file='검토 명령어'}

```json
{
  "is_violating": false,
  "category": null,
  "explanation_if_violating": null
}
```
{: file='검토 결과'}


---

## Reference

<https://platform.openai.com/docs>