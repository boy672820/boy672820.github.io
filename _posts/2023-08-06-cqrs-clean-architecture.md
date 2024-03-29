---
layout: post
title: CQRS와 Clean Architecture 도입 사례
categories: [cqrs, clean-architecture]
tags: []
description: CQRS 패턴과 Clean Architecture 도입 사례에 대하여
---

# 개요
---

프로젝트를 진행하면서 느꼈던 각종 이슈들이 있었고, 이를 극복하기 위해 Back-End 시스템은 많은 변화를 거쳤습니다. 보편화된 CRUD에서 CQRS 패턴 도입, Clean Architecture로 전환 사례에 대해 발표하겠습니다.

### 1. 진행한 프로젝트

	1.1. ARCA 월렛
	1.2. 파라다이스 게임
	1.3. 진이의 하루

### 2. 기존 시스템의 문제점

	2.1. 복잡한 비즈니스 모델
	2.2. 자주 변경되는 요구사항

### 3. CQRS와 Clean Architecture

	3.1. 명령(Command)과 조회(Query)
	3.2. 이벤트 소싱(Event Sroucing)으로 상태 관리
	3.3. 클린 아키텍처(Clean Architecture)


# 1. 진행한 프로젝트
---

## 1.1. ARCA 월렛

![Pasted_image_20230729230233.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230729230233.png)
이더리움 기반 BUP 코인을 이용한 현물 결제, 기프티콘 구매, 각종 이벤트 등을 진행

- 블록체인 기반 뱅킹 시스템
- 키오스크 결제 시스템
- 채굴을 통한 보상 지급 등 다양한 이벤트


## 1.2. 파라다이스 게임

![Pasted_image_20230729230312.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230729230312.png)
이더리움 GNI 코인을 이용하여 게임 포인트로 활용할 수 있는 Open API 제공, 유저간 NFT 거래 등

- 블록체인 기반 게임 연계형 뱅킹 시스템
- Open API 제공


## 1.3. 진이의 하루

![jin300](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230729230359.png)
사회적으로 고립된 사람들의 고독사를 방지하기 위한 서비스

- 관리 대상자 앱을 통한 모니터링
- 복지사 앱을 통한 빠르고 쉬운 대응이 가능
- AI 기술을 통한 알림 및 리포트 자동화




# 2. 기존 시스템의 문제점
---

대부분의 백엔드 시스템은 기본적으로 *CRUD(Create, Read, Update, Delete)* 패턴을 이용해 개발해 왔다. 간단한 게시판 등을 개발할 때는 적합하지만, 복잡한 비즈니스 로직을 요구하는 경우에는 부적합하다.


## MVC 패턴

백엔드 시스템은 기본적으로 *MVC(Model-View-Controller)* 구조로 이루어져 있다.

![Pasted_image_20230729233535.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230729233535.png)

- ***Controller***: 사용자 요청을 받고 비즈니스 로직(Service)을 제어한다.
- ***Model***: 애플리케이션의 정보 즉, 무엇을 할 것인지를 정의하는 부분이다. 컨트롤러에서 받은 데이터를 조작하고, 사용자에게 출력할 데이터를 다룬다.
	- ***Service***: 컨트롤러 제어에 따른 비즈니스 로직(데이터를 조작: *CRUD*)를 처리한다.
	- ***DTO***: Data Transfer Object는 사용자가 서버에게 보낸 데이터 검사하고 변환하여 Controller, Service, Repository 계층간 데이터를 교환한다.
	- ***Repository***: 비즈니스 로직이 데이터베이스에 접근할 수 있도록 일관된 인터페이스 제공하는 역할을 담당한다.

>**View는 어디에?**
>현재 실감콘텐츠팀의 서비스는 Front-End 파트가 나누어져 있어, Back-End에서 클라이언트에게 View(SSR 웹 등)를 제공하지 않는다. 예를 들어, 모바일 애플리케이션, Next.js로 개발하고 있는 웹 애플리케이션이 View를 담당하고 있다.


## 요구사항 예시

예를 들어, 회원가입의 요구사항은 다음과 같다.

##### 유저 회원가입

1. 아이디와 비밀번호를 입력하여 로그인
2. 이메일 인증을 통한 회원가입
3. 사용자는 실명을 추가로 입력 받는다.

간단해 보이지만, 외주 운영사 또는 기획자가 모르는 개발적인 요소가 있다.

##### 유저 회원가입

1. 아이디와 비밀번호를 입력하여 로그인
	- ==모든 회원의 각각에 아이디는 고유해야 됨==
	- ==보안을 위해 비밀번호는 영어 + 숫자 + 특수문자 등의 조합으로 이루어짐==
2. 이메일 인증을 통한 회원가입
	- ==이메일 발송을 위한 SMTP 서버 또는 외부 API 서비스 연동==
	- ==메일서버 관리 및 비용 발생==
3. 사용자는 실명을 추가로 입력 받는다.
	- ==개인정보 제공 동의서 작성 및 동의 여부 확인==
	- ==개인정보 보안을 위한 암호화 대책 강구==
	- ==회원탈퇴 시 개인정보는 완전 말소==




# 2.1. 복잡한 비즈니스 모델

요구사항이 간단해 보이지만 그렇지 않은 경우가 더 많다. 왜냐면 개발자가 고려해야할 사항이 한 둘이 아니기 때문이다.
"이메일을 통한 인증" 요구사항은 사용자 입장에서 간단해 보일 수 있지만, 시간 대비 개발 공수와 유지보수 비용은 생각보다 많이 들어간다.

![Pasted_image_20230729183009.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230729183009.png)

인증번호 발송과 인증은 다음과 같은 요소가 필요하다.

- 이메일 발송을 위해 외부 API 연동(또는 SMTP 서버 인스턴스)
- 인증 토큰 생성
- 생성된 토큰이 맞는지 인증 절차 진행
- 저장된 사용자들 중에서 해당 이메일을 사용하고 있는지 확인

![a300][https://boy672820.github.io/assets/images//Pasted_image_20230729184243.png]

>****인증 토큰**은 보안을 위해 암호화된 문자를 말한다.
>비밀번호를 통해 생성되며, 해당 비밀번호로 복호화한 후 인증을 진행한다.


## 복잡한 문제를 쉽게 풀어갈 경우

이러한 요구사항들을 기존 계층형 구조를 토대로하여 개발해보자

##### Controller
1. 입력 받은 데이터 유효성 검사
2. 이미 사용중인 이메일인지 확인(비즈니스 로직 경유)

##### Service
1. 인증번호를 이메일로 발송하기
2. 입력받은 인증번호 확인
3. 사용자 생성
4. 사용자 인증 토큰 생성
5. 사용자 정보와 인증 토큰을 반환


![Pasted_image_20230730001939.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230730001939.png)


### 서비스 계층의 과도한 업무 분담

요청 제어와 비즈니스 로직을 나누어 보안성과 빠른 기능 개발이 가능하다. 그러나 문제점은 Service 계층에 요구사항이 집중되어 있다는 점이다.

과도한 업무 분담에 따른 문제점은 다음과 같다.

- 유연하지 않은 구조로 버그 발생 시 빠른 대처가 불가능하다.
- 추가 기능 요청 시 확정하기 힘든 구조이다.
- 유닛 테스트가 어렵다.
- 성능 이슈 있음

![Pasted_image_20230730004128.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230730004128.png)

이러한 문제점이 생겼을 경우, 서비스 계층 전체를 다시 개발해야 하는 사례가 종종 있었다.




# 2.2. 자주 변경되는 요구사항

만약, 유저 회원가입 기능의 이메일이 전화번호로 변경된다면 어떻게 될까? 아마 비즈니스 로직을 제외한 거의 대부분을 변경해야 할 것이다.

![change_unique500](https://boy672820.github.io/assets/images/Pasted_image_20230804014315.png)

## 엔티티(Entity) 변경에 따른 악영향

>**엔티티(Entity) 란?**
>비즈니스 모델의 주체를 모델링한 것이다. 예를 들어, 회원가입의 주체는 회원가입할 유저이다.

회원가입의 유저 엔티티는 다음과 같다.

| 속성 | 설명 |
| --- | --- |
| **이메일** | 사용자 이메일 주소 (Identifier)
| 비밀번호 | 암호화 된 로그인 비밀번호 |
| 이름 | 사용자 실명 |

특히 이메일 속성의 경우, 엔티티의 식별자(Identifier) 역할을 한다. 식별자는 고유하며 값이 변하지 않는다. 또한, 빈 값 또는 `Null`이 될 수 없다.

만약 비즈니스 모델의 엔티티가 변경 된다면 어떻게 될까? 프로젝트 초기의 설계 단계에서는 별다른 영향은 없을 것이다. 문제는 개발이 어느정도 진행 된 상태이다. 개발 설계 단계부터 시작해서 거의 대부분의 비즈니스 로직, 구조, 데이터베이스까지 큰 영향이 간다. 때문에 프로젝트 기간도 짧아지며, 이는 비용적인 문제로 직결된다.

![Pasted_image_20230804122946.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230804122946.png)

회원가입의 유저 엔티티가 변경될 경우 다음과 같은 

- **DTO**: 이메일 속성이 제거되거 전화번호 속성 추가, 유효성 검사 변경
- **Service**: 비즈니스 로직이 전반적으로 변경 됨(외부 API 연동, 기존 JWT 방식에서 캐싱으로 변경, 고유번호 생성 방식 등)
- **Repository**: `find` 인터페이스 변경, 인덱스 재설정, 복잡한 쿼리가 들어가는 인터페이스의 경우 잘 고려하여 재설계 필요함
- **Database**: 고유 식별자 변경, 이미 존재하는 데이터는 마이그레이션 필요

![Pasted_image_20230804122255.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230804122255.png)


## 어떻게 해결할 것인가

이러한 요구사항에 따라 기존 구조로 빠른 문제 해결이 가능하지만, 결과적으로 봤을 때 비교적 작은 문제를 큰 문제로 만든 셈이 되었다.

![Pasted_image_20230801185709.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230801185709.png)








# 3. CQRS와 Clean Architecture
---

**CQRS(Command Query Responsibility Segregation)** 은 명령(Command)과 조회(Query)의 분리를 의미한다.

![Pasted_image_20230804203441.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230804203441.png)

**클린 아키텍처(Clean Architecture)** 는 기존 계층형 구조의 의존성 문제를 해결하기 위해 고안된 구조이다. 『클린 코드(Clean Code)』를 저술한 로버트 마틴이 제안한 구조로 데이터베이스 주도 설계로 인한 문제를 해결하고 유연하고 테스트할 수 있는 코드를 작성하게 해준다.

![Pasted_image_20230804210550.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230804210550.png)




## 3.1. 명령(Command)과 조회(Query)

CQRS는 데이터베이스에 대한 읽기 및 업데이트 작업을 구분한다. 명령 모델은 시스템의 상태를 변경, 추가, 삭제를 담당하고, 조회 모델은 읽기를 담당한다.

### 기존 아키텍처의 문제점

![Pasted_image_20230805000913.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230805000913.png)

기존 아키텍처는 동일한 데이터 모델을 사용한다. 간단한 CRUD 작업에는 적합하지만, 복잡한 비즈니스 모델에서는 이 방법이 어려울 수 있다. 예를 들어 뱅킹 시스템의 계좌이체의 경우는 다음과 같다.

1. 받는 계좌 조회
2. 보유 금액이 이체할 금액보다 충분한지 확인
3. 상대 계좌 보유금액 확인
4. 내 계좌의 보유 금액 차감
5. 받는 계좌의 보유 금액 증가
6. 거래내역 저장
7. 푸시알림 발송
8. 트랜잭션 실패 시 롤백
9. 데이터 로깅

상대 계좌와 내 계좌 조회 후 소유 금액 증감한다. 이후 거래내역은 저장한다. 만약 트랜잭션 실패 시 증감했던 소유 금액은 다시 롤백해야 한다.

이처럼 복잡한 기능을 하나의 데이터 모델을 사용할 경우 다음과 같은 문제가 발생한다.

- 하나의 데이터 모델을 이용하기 때문에 데이터 불일치가 발생할 수 있다.
- 읽기와 쓰기 작업을 같은 데이터 모델에 사용하기 때문에 복잡성이 증가하고 권한 및 보안 관리가 복잡해질 수 있다.
- 경우에 따라 데이터베이스와 데이터 접근 계층에 많은 부하가 간다.

![Pasted_image_20230805010127.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230805010127.png)


### 읽기와 쓰기의 분리

이 같은 문제를 해결하기 위해 명령(쓰기)와 조회(읽기)를 다른 모델로 분리한다.

- 명령은 데이터 중심이 아닌 작업 기반
- 명령은 동기적으로 처리되지 않고 비동기 처리를 위해 큐에 배치될 수 있음
- 조회는 데이터베이스를 변경하지 않음
- 조회는 도메인 정보를 캡슐화하지 않는 DTO를 반환

![Pasted_image_20230805120327.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230805120327.png)

더 높은 격리 수준과 성능 등을 고려해 봤을 때 읽기와 쓰기 데이터베이스를 분리할 수 있다. 이 경우 읽기 데이터베이스는 쿼리에 대한 최적화된 고유한 스키마를 사용할 수 있다. 또한 성능을 위해 쓰기는 관계현 데이터베이스를 사용 하지만, 읽기는 문서형 데이터베이스를 사용할 수 있다.

데이터베이스를 분리 하려면 동기화를 유지해야 한다. 일반적으로 쓰기 데이터베이스가 업데이트 될 때마다 트리거 또는 이벤트를 통해 읽기 데이터베이스를 동기화할 수 있다.



## 3.2. 이벤트 소싱(Event Sourcing)으로 상태 관리

CQRS 패턴은 이벤트 소싱 패턴과 함께 사용되는 경우가 많다. 애플리케이션의 상태는 이벤트 시퀸스로 저장되며, 각 이벤트는 데이터에 대한 변경을 나타낸다. 이를 통해 데이터 변경에 대한 추적, 로깅, 디버깅, 쓰기 작업으로 인한 읽기의 복제와 동기화에 유용하다. 또한 요청에 따른 빠른 응답이 가능하다.

![Pasted_image_20230805123857.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230805123857.png)

특히 이벤트 소싱은 쓰기 모델로 사용되어 읽기 모델을 비동기적으로 생성할 수 있다.

장점

- 데이터 변경에 완벽한 기록
- 명령과 조회 분리를 위한 최적화
- 도메인 모델링 용이성



### CQRS와 결합하기

그러면 이벤트 소싱이 CQRS와 어떻게 결합 되는지 계좌이체 기능을 통해 알아보자

1. 받는 계좌 조회
2. 보유 금액이 이체할 금액보다 충분한지 확인
3. 상대 계좌 보유금액 확인
4. 내 계좌의 보유 금액 차감
5. 받는 계좌의 보유 금액 증가
6. 거래내역 저장
7. 푸시알림 발송
8. 트랜잭션 실패 시 롤백
9. 데이터 로깅

여기서 상대 계좌 조회와 잔액 증감은 트랜잭션으로 처리한다.

>**트랜잭션**은 은행 [ATM](http://wiki.hash.kr/index.php/ATM "ATM")이나 [데이터베이스](http://wiki.hash.kr/index.php/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4 "데이터베이스") 등의 시스템에서 사용되는 더 이상 쪼갤 수 없는 업무 처리의 최소 단위이다. 예를 들어, A라는 사람이 B라는 사람에게 1,000원을 지급하고 B가 그 돈을 받은 경우, 이 거래 기록은 더 이상 작게 쪼갤 수가 없는 하나의 트랜잭션을 구성한다. 만약 A는 돈을 지불했으나 B는 돈을 받지 못했다면 그 거래는 성립되지 않는다. 이처럼 A가 돈을 지불하는 행위와 B가 돈을 받는 행위는 별개로 분리될 수 없으며 하나의 거래내역으로 처리되어야 하는 단일 거래이다. 이런 거래의 최소 단위를 트랜잭션이라고 한다. 트랜잭션 처리가 정상적으로 완료된 경우 [커밋](http://wiki.hash.kr/index.php/%EC%BB%A4%EB%B0%8B "커밋")(commit)을 하고, 오류가 발생할 경우 원래 상태대로 [롤백](http://wiki.hash.kr/index.php/%EB%A1%A4%EB%B0%B1 "롤백")(rollback)을 한다.

#### 작업 흐름

기본적으로 내 계좌 잔액이 충분한지 확인한다. 충분하다면 상대 계좌의 잔액을 조회하고 증가, 동시에 내 계좌 잔액은 감소 시킨다. 상대 계좌 조회와 잔액 증감은 하나의 트랜잭션에서 이루어져야 한다.

![Pasted_image_20230806114456.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806114456.png)

각각 다른 작업으로 진행할 경우 잔액 증감 시 데이터 불일치가 발생할 수 있다.

![Pasted_image_20230806122011.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806122011.png)

#### 명령과 이벤트로 분리하기

내 잔액을 조회하고 계좌 잔액을 증감하는 걸 하나의 작업으로 볼 수 있다.

![Pasted_image_20230806122941.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806122941.png)

>**잔액 조회가 Query에서 이루어지지 않는 이유**
>일반적으로 CQRS 패턴에서는 Command와 Query를 분리하고 서로 독립적으로 작동하도록 한다. 이는 명령(Command)과 조회(Query)가 서로 다른 목적을 가지기 때문이다.
>따라서 CQRS 패턴에서는 일반적으로 Command에서 Query를 직접 호출하지 않는다. 하지만 예외적으로 Query를 호출하는 경우도 있을 수 있다. 예를 들어, 명령이 실행된 후 결과를 확인하기 위해 해당 명령에 대한 조회를 수행하는 경우가 있을 수 있다. 그러나 이런 경우는 주의해서 사용해야 하며, 일반적으로는 Query를 호출하지 않는 것이 좋다.


이후 거래내역 저장을 이벤트 소싱을 통해 수행한다. 이를 통해 서로 다른 두 모델의 관심사를 분리하고, 거래내역 저장을 비동기로 수행하여 응답 속도를 빠르게 한다.(성능 개선)

![Pasted_image_20230806124035.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806124035.png)

#### 도메인 로직

이벤트는 도메인 로직과 결합하여 호출된다. 이는 이벤트가 도메인의 상태 변화를 표현하고 외부 시스템이나 다른 컴포넌트에 알리는 중요한 수단으로 사용되기 때문이다.

- 도메인 상태 변화 표현
- 느슨한 결합(Loose Coupling): 도메인 로직이 이벤트를 발생시키면 다른 시스템의 상태나 로직에 대한 사전 지식을 갖지 않아도 되며, 의존성을 최소화할 수 있다.
- 이벤트 소싱과 CQRS: 명령(Command)이 실행되면 해당 명령에 대한 상태 변화를 이벤트로 기록, 데이터 변화를 표현

이벤트는 도메인의 상태 변화를 표현하고 외부 시스템과 느슨한 결합을 구현한다.

![Pasted_image_20230806132205.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806132205.png)

>도메인(Doamin)이란?
>영어 원문은 영역, 범위, 영토란 뜻이다. 소프트웨어 개발에서 특정 문제 영역 또는 비즈니스 영역을 나타내는 개념이다. 구체적으로 애플리케이션이 다루는 문제 영역에 대한 지식과 규칙을 포함하는 범위를 의미한다.




## 3.3. 클린 아키텍처(Clean Architecture)

『클린 코드(Clean Code)』를 저술한 로버트 마틴(Robert C. Martin)이 제안한 시스템 아키텍처로, 기존의 계층형 아키텍처가 가지던 의존성에서 벗어나도록 하는 설계를 제공한다.

기존 계층형 구조의 문제점은 의존성으로 각 계층이 서로 강력하게 연결되어 있다. 때문에 최종 목적지인 데이터베이스 계층의 변경이 상위 계층에 많은 영향을 준다. 이로 인해 기능 추가나 변경이 어려워질 수 있다.

![Pasted_image_20230806192627.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806192627.png)

클린 아키텍처는 이러한 문제점을 해결해준다. 각 계층의 의존성을 느슨하게 만들고 테스트 용이한 아키텍처를 제공하는 방법 중 하나이다.

### 규칙

클린 아키텍처는 4가지 영역으로 구분되어 있다. 안쪽부터 살펴보자

![Pasted_image_20230804210550.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230804210550.png)

| | |
| - | - |
| 엔터프라이즈 업무 규칙 | `엔티티` |
| 애플리케이션 업무 규칙 | `유즈 케이스` |
| 인터페이스 어댑터 | `컨트롤러` `게이트웨이` `프레젠터` |
| 프레임워크와 드라이버 | `기기` `웹` `데이터베이스` `UI` `외부 인터페이스` |

#### 엔티티(Entity)

- 핵심 업무 규칙을 캡슐화한다.
- 메서드를 가지는 객체, 일련의 데이터 구조와 함수의 집합이다.
- 가장 변하지 않으며 외부로부터 영향을 받지 않는 영역이다.

#### 유즈 케이스(Use Cases)

- 애플리케이션의 특화된 업무 규칙을 포함한다.
- 시스템의 모든 유즈 케이스를 캡슐화하고 구현한다.
- 엔티티로 들어오고 나가는 데이터 흐름을 조정하고 조작한다.

#### 인터페이스 어댑터(Interface Adapter)

- 외부 인터페이스에서 들어오는 데이터를 유즈 케이스와 엔티티에서 처리하기 편한 방식으로 변환하며, 유즈 케이스와 엔티티에서 나가는 데이터를 외부 인터페이스에서 처리하기 편한 방식으로 변환한다.
- 컨트롤러, 프레젠터, 게이트웨이 등이 여기에 속한다.

#### 프레임워크와 드라이버(Framework & Drivers)

- 시스템의 핵심 업무와는 관련 없는 세부 사항이다.
- 프레임워크나, 데이터베이스, 웹 서버 등이 여기에 해당한다.

### 경계(Boundary)

이때 클린 아키텍처는 경계를 가장 중요하게 생각한다. 경계는 소프트웨어 요소를 분리하고, 경계 한편에 있는 요소가 반대편 요소를 알지 못하도록 막는다.

![Pasted_image_20230806195328.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806195328.png)

> 로버트 마틴은 소프트웨어 아키텍처는 선을 긋는 기술이며, 나는 이러한 선을 경계(boundary)라고 부른다.

클린 아키텍처의 의존성은 화살표 방향처럼 밖에서 안으로 향하고, 바깥 원은 안쪽 원에 영향을 미치지 않는다. 경계 바깥에서 안쪽으로 갈수록 저수준에서 고수준으로 표현된다. **이를 통해 업무 로직(고수준 정책)은 세부 사항들(저수준 정책)의 변경에 영향을 받지 않도록 할 수 있다.**

![Pasted_image_20230806200236.png](https://boy672820.github.io/assets/images/2023-08-06-cqrs-clean-architecture/Pasted_image_20230806200236.png)

>**고수준과 저수준**
>고수준은 상위 수준의 개념으로 좀 더 추상화된 개념이다. 저수준은 추상화된 개념을 어떻게 구현할지에 대한 세부적인 개념이다. 예를 들어 고수준에서 저수준으로 이동한다면, "운동을 한다"에서 "스쿼트를 12개 수행한다"로 표현할 수 있다.

