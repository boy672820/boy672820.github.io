---
layout: post
title: 표준 입출력(Standard Stream)
categories: [programming]
tags: [unix]
description: 표준 입출력이란, Standard Stream
---

# 표준 입출력, Standard Stream(표준 스트림)

표준 입출력을 표준 스트림이라고 부른다.
간단하게 생각하면 키보드 입력, 모니터 출력이 표준 스트림에 해당한다.

- 표준 입력 → stdin(Standard Input) → 키보드 입력
- 표준 출력 → stdout(Standard Output) → 모니터 출력

# Standard Stream

컴퓨터 프로그램에서 표준적으로 입력을 받고, 출력하는 매체를 총칭
예를 들어, 유닉스 쉘 프로그램은 `ls` 명령어를 키보드로 입력받고 결과를 콘솔에 출력한다.

```bash
[seonzoo@macbook ~]$ ls
Applications         Downloads            Postman
Creative Cloud Files Library              Projects
Desktop              Movies               Public
Documents            Music
Documents-assets     Pictures
```

## Standard? Stream?

### Stream(스트림)

프로그램에서 사용하는 데이터를 바이트의 **흐름**으로 표현한 단어이다.(Byte Stream)

과거에는 입출력을 다루기위해 OS에서 하드웨어 관련 설정을 직접 해줘야 했었는데, 유닉스에서 이런 번거로움을 해결하기 위해 하드웨어를 추상화하여 장치를 파일처럼 다루는 것으로 해결했다.(파일을 읽고 쓰는 한 가지 작업으로 통일)
그리고 **그 파일에서 읽히고 나가는 데이터를 *stream*** 이라고 정의했다. (실제로 리눅스에서는 ‘/dev’ 디렉토리가 추상화한 장치들을 파일 형태로 담고 있는데 그 안에 *`stdin`* *`stdout`* 등이 있는 것을 확인할 수 있다.)

### Standard(표준)

모든 프로그램은 입출력이 필요하다. 이 입출력을 프로그램 개발 시 지정해주면 좋은데, **기본적으로 사용할 입출력 대상을 표준 입출력이라고 한다.**

표준 입출력은 아래와 같이 나눌 수 있다.

- 표준 입력
- 표준 출력 → 표준 출력, 표준 에러

## Standard Input

장비나 파일 등 데이터를 입력 받는 표준적인 출처, 표준 입출력이 유닉스에서 유래되어 `stdin`이라고 부른다.
리눅스에서 파일 디스크립터*가 0으로 설정되어 있다.

* 파일을 고유하게 구별하는 식별자

## Standard Output

장비, 파일 등 출력 받는 표준적인 출처, `stdout` 이라고 부른다.
표준 출력과 표준 에러(stderr)로 구분할 수 있다. 반환되는 방향이 정상적이면 전자이고 비정상 종료 시에 반환되면 후자이다.
리눅스에서 파일 디스크립터가 1로 설정되어 있고, 표준 에러의 경우 2로 설정되어 있다.

* 서버 프로그래밍을 할 때 일반적인 접속 로그는 `access.log`에, 에러 로그는 `error.log`에 담기는 것을 우리는 이제는 이해할 수 있다.