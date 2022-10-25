---
layout: post
title: 블록체인 트랜잭션 대기상태
categories: [blockchain, ethereum, polygon]
tags: [blockchain, ethereum, polygon, transaction issue]
description: 트랜잭션 대기상태에 따른 이후 거래가 보류(Pending)되는 문제
---

# Transaction is stuck pending

[[Help] Polygon Network Transaction is Stuck Pending. Not sure where to start.](https://www.reddit.com/r/0xPolygon/comments/r80pe4/help_polygon_network_transaction_is_stuck_pending/)

## 원인

- 상대적으로 낮은 가스비용 때문에 채굴이 안되는 경우
- 가스비용을 지불할 거래량이 없는 경우

### 1. Alchemy 노드를 사용하는 경우

Mempool 메뉴에서 Pending 상태의 트랜잭션을 확인한다. 필자의 경우, 낮은 가스비용 때문에 채굴이되지 않아 대기상태에 빠져있었다.

![해결된 상태의 Mempool](https://boy672820.github.io/assets/images/2022-10-25-transaction-stuck-pending/Untitled.png)

해결된 상태의 Mempool

가장 오래된 Pending Transaction의 Nonce 값이 필요하다.

### 2. Metamask 사용

우측 상단에 프로필 클릭 > 설정 > 고급으로 이동한다. `거래 임시값 맞춤화`를 활성화 한다.

[https://www.notion.so](https://www.notion.so)

“보내기”를 하되 값은 0으로 지정하고 계정주소를 입력한다. 이후 `맞춤형 임시값(Nonce)`을 입력한다.(Pending Transaction Nonce)

* 가스 비용은 폴리곤 기준 100 Gwei가 적당하다.

![Untitled](https://boy672820.github.io/assets/images/2022-10-25-transaction-stuck-pending/Untitled1.png)