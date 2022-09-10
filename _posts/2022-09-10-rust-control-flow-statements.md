---
layout: post
title: Rust#4 제어문
categories: [rust]
tags: [basic, rust]
description: Rust 입문, if, loop, while, for 제어문 다루기
---

# 제어문

조건에 따른 코드 실행 여부, 반복 수행 등은 프로그래밍 언어의 기초 문법이다.

## if 표현식

조건에 따라 코드를 실행하거나 분기할 수 있다. **조건은 참(true) 또는 거짓(false)의 `bool` 타입만 허용한다.**

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

조건 `number < 5` 에 따라 분기된다.(`else`는 조건이 거짓(false)일 경우)

### 변수에 if 사용하기

if 자체가 표현식으로 변수에 할당할 수 있다.

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

만약 반환되는 타입이 다를 경우 에러가 발생한다.

```rust
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

```rust
error[E0308]: if and else have incompatible types
 --> src/main.rs:4:18
  |
4 |       let number = if condition {
  |  __________________^
5 | |         5
6 | |     } else {
7 | |         "six"
8 | |     };
  | |_____^ expected integral variable, found reference
  |
  = note: expected type `{integer}`
             found type `&str`
```

이런 경우 **변수의 타입을 정의**한다.

- 해당 number 변수의 타입이 유효한지 보증할 수 있다.
- 컴파일러가 타입을 추적하여 복잡해지는 것을 방지한다.

## 반복문

Rust는 `loop` `while` `for` 를 제공한다.

### **loop를 통한 무한반복**

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

`break` 키워드를 통해 루프를 멈출 수 있다.

### while 조건부 반복

조건이 충족할 경우 반복문을 수행한다. `loop`는 `if`, `else`, `break`를 혼합하여 사용하지만 while은 보다 간결해진다. 조건은 `bool` 타입만 허용하며 `break` 키워드를 사용할 수 있다.

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

### for 콜렉션 반복

배열과 같은 콜렉션을 순회하여 각 요소들을 다룰 수 있다.

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index = index + 1;
    }
}
```

그러나 이런 방식은 배열의 길이가 바뀔 경우 에러가 발생하기 쉽다. 보다 효율적인 방법으로 Rust에서 기본 라이브러리로 제공하는 `Range` 를 사용할 수 있다.

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

`rev` 메소드를 사용하여 배열의 역순으로 순회할 수 있다.

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```