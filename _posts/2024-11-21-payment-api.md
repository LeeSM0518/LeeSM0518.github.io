---
title: 결제 API 연동 튜토리얼
date: 2024-11-21 07:00:00 +0900
categories: tutorial
tags:
  - tutorial
  - guide
description: 결제 API를 연동하는 튜토리얼
---
## 1. 개요

사이드 프로젝트에서 결제 기능을 적용하기 위해 튜토리얼 프로젝트를 진행한다.

결제 기능을 무료로 적용해 볼 수 있는 [PortOne](https://portone.io/korea/ko/service/one-payment-infra#integration)을 활용해 튜토리얼을 진행해보자.

<br/>

## 2. 포트원 결제 연동 가이드

### 2.1. 결제 연동 준비하기

1. [PortOne](https://portone.io/korea/ko)에서 회원가입 수행
2. 연동 정보를 `테스트` 로 변경
  ![portone1](/assets/img/portone1.png)

3. 사용할 `결제대행사` 를 선택하고, `결제 모듈` 선택
  ![portone2](/assets/img/portone2.png)

4. `다음` 으로 이동한 후, *PG상점아이디 (MID)* 를 클릭하면 나오는 것을 클릭하여 자동 완성
  ![portone4](/assets/img/portone4.png)