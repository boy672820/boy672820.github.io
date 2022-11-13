---
layout: post
title: HTTP 쿠키(Cookie)와 세션(Session)
categories: [http]
tags: [http, cookie, session]
description: HTTP에서의 쿠키와 세션에 대해서
---

# 쿠키(Cookie)와 세션(Session)

HTTP 특성상 연결된 상태를 유지할 수 없고(비연결지향, Connectionless) 정보 또한 유지하지 않는다.(상태 유지안함, Stateless)

그러나, 실제로 데이터 유지가 필요한 경우가 많다. 예를 들면 로그인 등인데, 쿠키와 세션은 이런 HTTP의 단점을 보완한다.

### 쿠키, Cookies

방문 사이트에서 사용자의 정보를 기록하는 데이터파일이다. 클라이언트에 정보를 저장해두었다가 필요 시 정보를 재사용한다.

### 세션, Session

사이트 접속 시점에서 종료까지 일련에 요청(상태)을 일정하게 유지하는 기술 즉, 방문자가 사이트에 접속해있는 상태를 하나의 단위로 본다. 데이터는 서버에 저장된다.

쿠키와 세션이 다른 것처럼 보이지만 **세션을 사용하기 위해서 쿠키가 사용된다.**

```
Cookie: MY_SESSION_ID=ww91IGdvd...
```

서버는 임의에 ID값을 클라이언트(브라우저)에 보내고 추후 재접속 시 해당 세션ID를 이용해 세션 연결을 시도한다.

PHP와 같은 언어에서는 저장 시스템이 내장되어 있으나 기본적으로 로컬 파일 시스템에 데이터를 저장한다. Node.js 환경에서는 메모리에 있으며 서버가 다시 시작되면 사라진다.

### JWT, 함호화된 토큰

세션을 이용한 사용자 인증은 메모리에 저장되는 세션 특성상 Redis, Memcached와 같은 인메모리 데이터베이스를 사용하였다. JWT는 암호화된 인증 정보를 클라이언트에 저장 후 인증하는 방식으로 많은 인기를 가지고 있다.

참조: [https://cheershennah.tistory.com/135](https://cheershennah.tistory.com/135)

참조2: [https://evertpot.com/jwt-is-a-bad-default/](https://evertpot.com/jwt-is-a-bad-default/)