---
layout: post
title: Rust#1 변수와 상수
categories: [rust]
tags: [basic, rust]
description: Rust 입문, 변수와 상수에 대해서
---

# 변수와 상수

변수 선언은 아래와 같이 한다.

```rust
let a = 'apple';
```

같은 변수명으로 **shadowing** 할 수 있다.(마지막 선언된 변수로 이전 변수를 가리는 것을 말함)

```rust
let a = 'apple';
let a = a.len; // -> 5
```

변수는 기본적으로 불변하기 떄문에 아래와 같이 대입할 수 없다.

```rust
let a = 'apple';
a = 'afreeca'; // error[E0384]: cannot assign twice to immutable variable `a`
```

이를 immuable하다고 표현한다. `mut` 예약어로 변수로서의 역활을 수행할 수 있다.

```rust
let mut a = 'apple';
```

단, 변수 타입이 바뀌면 안된다.

```rust
let mut a = 'apple';
a = a.len; // error[E0308]: mismatched types
```

### 상수

상수는 기본적으로 불변 그자체이기 때문에 변경할 수 없다. 때문에 선언 시 대입하는 값은 상수여야 한다.(함수의 반환처럼, 런타임 시 동적인 값은 불가능)

선언 시 타입을 지정해야 한다.

```rust
const MAX_POINTS: u32 = 100_000;
```