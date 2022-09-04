---
layout: post
title: Rust#2 데이터 타입
categories: [rust]
tags: [basic, rust]
description: Rust 입문, 데이터 타입에 대하여
---

# 데이터 타입

Rust는 기본적으로 어떤 타입인지 명시하여 데이터를 다룬다. 타입은 크게 스칼라와 컴파운드로 나눈다.

Rust는 타입이 고정된 언어로 모든 변수의 타입이 컴파일 시에 반드시 정해져 있어야 한다. 컴파일러가 타입을 추측할 수 있지만 형변환 시 타입의 선택 폭이 넓은 경우 반드시 타입 명시를 첨가해야 한다.

예를 들어 `String` 을 `parse`하여 숫자로 변환했을 경우

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

타입을 명시하지 않을 경우 Rust는 에러를 발생시킨다.

```rust
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^
  |         cannot infer type for `_`
```

## 스칼라 타입

하나의 값으로 표현되는 타입을 말한다. 정수형, 부동 소수점, boolean, 문자 타입이 스칼라 타입이다.

### 정수형

소수점 없는 숫자이다.

| 8bit | i8 | u8 |
| --- | --- | --- |
| 16bit | i16 | u16 |
| 32bit | i32 | u32 |
| 64bit | i64 | u64 |
| arch | isize | usize |

정리하면 다음과 같다.

- 대부분 프로그래밍 언어가 그렇듯이, 음수의 경우 2의 보수 형태로 저장된다.
- `isize` & `usize` 타입은 프로그램이 동작하는 컴퓨터 환경이 64bit or 32bit 환경인지에 따라 결정된다.
- 정수 리터럴 접미사 허용
- 기본값은 `i32` 로 모든 환경에서 빠르며, 확실히 정해진 경우가 아니면 좋은 선택이다.

### 부동 소수점

실수 표현은 부동 소수점으로 두 가지 타입이 있다.

| 32bit | f32 |
| --- | --- |
| 64bit | f64 |

기본 타입은 `f64` 이유는 최신 CPU 상에서는 `f32` 와 대략 비슷한 속도를 내면서 더 정밀한 표현이 가능하다.(단, 그 만큼 메모리 공간 차지)

부동 소수점은 IEEE-754 표준에 따라 표현한다. 표준 구현 특성상 많은 공간을 차지하는 `f64` 가 정밀도가 더 높다.(`f32`는 1배수의 정밀도, `f64`는 2배수의 정밀도)

### 수학적 연산

기본적인 사칙연산은 다음과 같다.

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;

    // remainder
    let remainder = 43 % 5;
}
```

### boolean

`true` 와 `false` 이다. `bool` 로 명시

```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

### 문자

`String`이 큰따옴표를 쓰는 것에 반하여 `char` 는 언어의 가장 근본적인 알파벳 타입이다.(또한 많은 표현이 가능, 한국어/중국어/일본어/이모티콘/공백 등)

## 컴파운드 타입

컴파운드(복합) 타입은 다른 타입의 다양한 값들을 하나의 타입으로 묶어 사용할 수 있다.

### 튜플

여러 타입의 집합을 말한다. 타입 명시는 선택 사항이다.

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

튜플은 단일(스칼라 타입) 요소를 위한 복합계로 고려되었기 때문에 `tup` 에는 튜플 전체가 바인드된다. 다음과 같이 개별적으로 값을 추출할 수 있다.

**패턴 매칭을 통한 값 추출**

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

**색인을 통한 값 추출**

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

### 배열

배열은 튜플과 다르게 모든 요소가 같은 타입이여야 한다. 배열이 유용할 때는 데이터를 heap보다 stack에 할당하거나, 항상 고정된 요소를 같는 집합에 유용하다. 예를 들면 달력이 있다.

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Rust의 배열이 다른 언어의 배열과 다른 점은 고정된 길이를 갖는다는 점이다. 이는 배열의 끝을 넘어선 요소에 접근하려고 한다면 다음과 같은 에러가 난다.

```rust
$ cargo run
   Compiling arrays v0.1.0 (file:///projects/arrays)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/arrays`
thread '<main>' panicked at 'index out of bounds: the len is 5 but the index is
 10', src/main.rs:6
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

컴파일 시 에러가 발생하지 않지만, 프로그램 결과는 실행 중에 에러가 발생했고 성공적으로 종료되지 못했다고 나온다. 이처럼 프로그램 오류와 함께 종료될 때 Rust에서는 “패닉(Panic)” 상태라고 표현한다.

이것은 Rust의 안전 원칙이 동작하는 첫 번째 예로, 많은 저수준 언어에서 이러한 타입의 검사는 수행되지 않으며 잘못된 색인을 제공하면 잘못된 메모리에 접근할 수 있다. **Rust는 메모리 접근을 허용하는 대신 즉시 종료하여 이러한 종류의 오류로부터 사용자를 보호한다.**

```jsx
const arr = [1, 2, 3];
console.log(arr[3]); // undefined
```