---
title: AWS DB Migration Seminar
date: 2025-08-22 09:00:00 +0900
categories: aws
tags:
  - aws
  - db
description: AWS DB Migration Seminar 청강
---

## DB Migration Overview
---

- relocate
    - aws evs
- rehosting
    - 기존 온프레미스 인프라 환경을 그대로 옮기는 방법
    - aws ec2
- reflatform
    - 기존 온프레미스에 맞게 구성되어 있던 OS, WEB, DB 등과 같은 소프트웨어 환경을 변경해 옮기는 방법
    - aws ecs or eks
    - aws rds, fsx, workspaces
- refactoring
    - 기존 온프레미스의 환경을 클라우드 네이티브 환경으로 재 구축해 옮기는 방법
    - aws lambda

<br/>

## Agenda
---

- Oracle to PostgreSQL 진행
- AWS SCT(Schema Conversion Tool)
    - 데이터베이스의 스키마 변환
    - 서로 다른 데이터베이스 엔진 간 마이그레이션할 때 사용
- AWS DMS(Database Migration Service)
    - 실제 데이터를 옮기는 작업
    - 데이터를 소스 DB에서 대상 DB로 마이그레이션할 때 사용

<br/>

## Reference
---

[2025.08.22 - DMS Immersion Day](https://cotton-chartreuse-96c.notion.site/2025-08-22-DMS-Immersion-Day-256258b2de34807a837fda9522ec0b13)

[AWS Database Migration Workshop](https://catalog.workshops.aws/databasemigration/en-US)