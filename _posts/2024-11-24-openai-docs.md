---
title: OpenAI API Documents
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

1. [Billing 페이지](https://platform.openai.com/settings/organization/billing/overview)에서 카드를 등록한다.
  ![chatgpt4](/assets/img/chatgpt4.png)
  ![chatgpt5](/assets/img/chatgpt5.png)
1. [Dashboard 페이지](https://platform.openai.com/api-keys)에서 키를 생성한다.
  ![chatgpt1](/assets/img/chatgpt1.png)
  ![chatgpt2](/assets/img/chatgpt2.png)
  ![chatgpt3](/assets/img/chatgpt3.png)
3. 다음 명령어를 통해 API를 호출한다.
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

