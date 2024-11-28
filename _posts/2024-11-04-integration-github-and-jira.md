---
title: GitHub과 Jira 연동 가이드
date: 2024-11-04 00:00:00 +0900
categories: github
tags:
  - github
  - jira
  - integration
description: GitHub과 Jira를 연동하여 브랜치나 커밋 이력을 Jira에 자동 등록되도록 하는 가이드 문서
---

## 1. 개요

​​Github Repository에 브랜치를 생성하거나 커밋을 작성하여 올릴 경우, Jira의 티켓에 자동으로 할당되도록 Github Repository와 Jira 연동이 필요하다.

다음은 Github Repository와 Jira를 연동하는 과정이다.

## 2. GitHub과 Jira 연동

![integration1](/assets/img/github-jira-integration1.png)
_앱 > 더 많은 앱 살펴보기 이동_

![integration2](/assets/img/github-jira-integration2.png)
_검색 창에 github for jira 검색_

![integration3](/assets/img/github-jira-integration3.png)
_Get it now 클릭_

![integration4](/assets/img/github-jira-integration4.png)
_Get it now 클릭_

![integration5](/assets/img/github-jira-integration5.png)
_앱 > 앱 관리 이동_

![integration6](/assets/img/github-jira-integration6.png)
_GitHub for Jira의 Get started 클릭_

![integration7](/assets/img/github-jira-integration7.png)
_GitHub Cloud를 사용할 경우 Next 클릭_

![integration8](/assets/img/github-jira-integration8.png)
_Authorize Jira 클릭_

![integration9](/assets/img/github-jira-integration9.png)
_연결을 원하는 Repository 설정_

![integration10](/assets/img/github-jira-integration10.png)
_Jira에서 연결 결과 확인_

![integration11](/assets/img/github-jira-integration11.png)
_GitHub에서 연결 결과 확인_

<br/>

## 3. 브랜치 및 커밋 생성

### 3.1. Jira 티켓 번호로 브랜치 생성

![integration12](/assets/img/github-jira-integration12.png)
_DVK-53 티켓_

![integration13](/assets/img/github-jira-integration13.png)
_DVK-53 브랜치 생성 후 push_

### 3.2. 연동 확인

![integration14](/assets/img/github-jira-integration14.png)
_Jira 티켓에 연결된 브랜치 확인_
