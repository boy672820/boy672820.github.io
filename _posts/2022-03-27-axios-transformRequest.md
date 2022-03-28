---
layout: post
title:  Axios 옵션 transformRequest 사용하기
categories: [Tip]
tags: [axios, web, nodejs, javascript]
description: Axios transformRequest 옵션 사용 시 타입에러가 발생하는 문제 해결
---

[transformResponse docs misleading · Issue #1910 · axios/axios](https://github.com/axios/axios/issues/1910#issuecomment-590258745)

`transformRequest` 옵션은 요청 시 본문과 헤더를 변경할 때 사용된다. 콜백함수로 받으며 배열로 설정할 수 있다. POST, PUT, PATCH에만 옵션이 적용된다.

문제는 마지막 요소의 콜백함수는 Buffer 형식의 문자열 또는 인스턴스를 반환해야하는데, 이를 Axios에서 defaults로 구현된 함수로 반환하면 쉽게 문제를 해결할 수 있다.

```tsx
{
	transformRequest: [
	  (data) => ({
	    templateId: this.templateId,
	    sendNo: this.sendNo,
	    body: '',
	    ...data,
	  }),
	  ...(axios.defaults.transformRequest as AxiosRequestTransformer[]),
	],
}
```