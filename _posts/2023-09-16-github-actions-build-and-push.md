---
layout: post
title: Github Actions를 이용한 배포 자동화 (with Docker)
categories: [github-actions, docker]
tags: [github-actions, docker]
description: Docker Container 배포 자동화 파이프라인 구축을 위한 Github Actions 활용하기
---

EC2 환경에 도커를 이용한 컨테이너 배포를 진행하면서 많은 이점이 있었다. 가령 서버 환경에 맞춰 배포를 진행하는데 다음과 같은 문제점이 있었다.

- Node.js 설치 등 서버에 필요한 선행 패키지 설치 작업(이미지를 따면 되지만 그래도 뭔가 불안)
- 개발 환경과 배포 환경의 불일치로 잦은 실수 발생
- 스케일링 확장의 어려움
- 위의 실수들 또는 문제점들로 인한 보안적 문제점 발생 가능성

도커 컨테이너를 이용한 호스트 컴퓨터와의 분리는 확장성, 배포 용이성, 보안 강화 등의 잠재적 문제점들을 해결할 수 있었다. 무엇보다 일관된 배포로 서버 환경간의 차이로 발생하는 잠재적 보안 문제 해결과 개발 용이성이 가장 큰 장점일 것이다.

이러한 도커 컨테이너를 좀 더 고도화하여 배포 파이프라인을 구축하면 개발 능률을 올릴 수 있을 것이란 생각이 들었다.

### 배포 파이프라인 구축하기

배포 파이프라인을 구축하기에 앞서 다음과 같은 세팅이 선행되어야 한다.

- 컨테이너 레지스트리는 GHCR을 사용
- 당연 EC2와 같은 서버 환경이 필요함

## Github Actions

빌드, 테스트, 배포, 스케줄링 등을 자동화할 수 있게 해주는 Github 서비스이다. YML 포맷을 이용하며 이미 구현되어 있는 액션들을 적용해 볼 수 있다.

예를 들어, `main` 브랜치가 업데이트되면 컨테이너를 빌드하는 자동화를 구축한다면 다음과 같다.

```yml
name: Build your app

env:
  DOCKER_IMAGE: ghcr.io/${{ github.repository }}
  DOCKER_CONTAINER: lifesupport-backend

on:
  push:
    branches: ['main']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v3

      - name: Setup docker build
        id: buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to ghcr
        uses: docker/login-action@v2
        with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GHCR_TOKEN }}

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v3
        with:
        push: true
        tags: ${{ env.DOCKER_IMAGE }}:latest
        file: ./docker/production/Dockerfile
```

`.yml` 파일은 저장소 루트 디렉토리의 `.github/workflows/{액션파일명}.yml` 형태로 저장된다. 이제부터 요소 하나하나 알아 가보자



#### Job

작업은 독립된 환경에서 처리되는 하나의 작업을 의미한다. 수행할 작업을 `jobs` 속성에 정의하면 된다. 가령 `build` 라는 이름의 작업은 ubuntu 환경에서 돌아가며 다음과 같은 작업을 진행한다.

1. `uses: actions/checkout@v3`: 사용할 액션으로 `actions/checkout@v3`를 사용한다. 이 액션은 독립된 환경(ubuntu)에 저장소 코드를 내려 받는다. 빌드를 위해 저장소의 코드를 사용할 것이다.
2. `uses: docker/login-action@v2`: 빌드하기 전 레지스트리에 로그인하기 위한 액션이다. `username`과 `password`는 Github Actions에서 제공하는 환경변수와 저장소의 secrets 환경변수를 사용한다.
3. `uses: docker/build-push-action@v3`: 빌드를 위한 모든 선행 작업을 끝냈다. 이제 빌드 후 레지스트리에 업로드하기 위해 `docker/build-push-action@v3`를 사용한다. 태그는 Github Actions에서 지정한 환경변수를 사용하고 `Dockerfile`을 지정한다.


#### Env

`env`는 Github Actions에서 사용할 변수이다. 위의 작업에는 공통적으로 들어갈 `DOCKER_IMAGE`, `DOCKER_CONTAINER` 변수가 있다.


#### On

Github Actions가 실행될 조건으로 `main` 브랜치에 push 작업이 발생하면 작업들이 실행된다.


## 최종 Github Actions

다음은 배포 자동화를 위한 Github Actions이다.

```yml
name: Build your app

env:
  DOCKER_IMAGE: ghcr.io/${{ github.repository }}
  DOCKER_CONTAINER: nodeserver

on:
  push:
    branches: ['main']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v3

      - name: Setup docker build
        id: buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to ghcr
        uses: docker/login-action@v2
        with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GHCR_TOKEN }}

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v3
        with:
        push: true
        tags: ${{ env.DOCKER_IMAGE }}:latest
        file: ./docker/production/Dockerfile

  deploy:
    needs: build
    runs-on: [self-hosted, Linux, X64]

  steps:
  - name: Login to ghcr
    uses: docker/login-action@v2
    with:
      registry: ghcr.io
    username: ${{ github.repository_owner }}
    password: ${{ secrets.GHCR_TOKEN }}
  
  - name: Run docker
    run: |
    docker stop ${{ env.DOCKER_CONTAINER }} && docker rm ${{ env.DOCKER_CONTAINER }} && docker rmi ${{ env.DOCKER_IMAGE }}:latest
    docker run -dp 3000:3000 --network lifesupport-network --name ${{ env.DOCKER_CONTAINER }} --restart always ${{ env.DOCKER_IMAGE }}:latest

  notification:
    needs: deploy
    permissions:
      contents: read
      actions: read
    runs-on: ubuntu-latest
  
    steps:
      - name: Action Slack
        uses: 8398a7/action-slack@v3
        with:
        status: ${{ job.status }}
        author_name: boy672820
        fields: repo,message,commit,author,action,eventName,ref,workflow,job,took
  
        if_mention: failure,cancelled
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      if: always()
```


### Deploy 작업

빌드되어 레지스트리에 업로드된 이미지를 서버에서 내려받아 컨테이너로 실행하는 작업이다. 하지만 서버는 Github Actions의 작업을 어떻게 확인할까?

#### Github Self-hosted Runner

Github Actions에서 지정한 서버의 자원을 사용하는 기능이다. 주로 Github Actions에서 제공하는 독립적 환경에서 실행하기 버거운 작업(=비용)이 있을 때 유용하다.

여기서는 빌드된 이미지를 서버가 직접 내려받아 컨테이너로 실행시키기 위해 사용된다.

(설치 방법은 매우 쉬움 생략)


### Notification 작업

배포가 완료되면 알림을 발송하는 작업이다. 배포 완료되면 지정된 슬랙으로 발송됨
