---
title: 자바스크립트는 어떻게 동작하는가 (feat. 이벤트 루프)
date: 2022-02-07
categories: [TIL]
tags: [자바스크립트] # TAG names should always be lowercase
image:
  src: https://noticon-static.tammolo.com/dgggcrkxq/image/upload/v1567008394/noticon/ohybolu4ensol1gzqas1.png
  width: 100 # in pixels
  height: 100 # in pixels
  alt: JavaScript
mermaid: true
published: false
---

## status

```js
{
  status: "pending",
  message: "영상에 대한 내용이 모두 정리되고 && 비동기 코드의 동작을 코드 예시로 완벽하게 설명이 가능하면 퍼블리시합니다."
}
```

## 목표

자바스크립트가 어떻게 동작하는지 설명합니다.

## 자바스크립트는 어떤 언어인가

> A single-threaded non-blocking asynchronous concurrent language

자바스크립트는 싱글스레드 언어입니다. 여기서 싱글스레드란 콜 스택이 하나만 존재한다는 뜻입니다. 콜 스택이 하나라는 것은 한 번에 하나의 일만 처리할 수 있음을 뜻합니다.

```js
Single threaded

one thread === one call stack === one thing at a time
```

### 콜 스택이란 무엇인가

콜 스택이란 무엇일까요?

콜 스택(call stack), 다른 말로 호출 스택은 여러 함수를 호출하는 스크립트 언어에서 현재 실행하고 있는 스크립트의 위치를 기억하는 자료구조입니다. 현재 어떤 함수가 동작하고 있는지, 해당 함수가 종료되면 다음에 어떤 함수를 호출해야 하는지를 제어합니다.

다음 스크립트를 실행하면 콜 스택의 형태가 다음과 변화합니다.

```js
function foo() {
  console.log("foo");
}

function bar() {
  foo();
}

function baz() {
  bar();
}

baz();

// call stack changes
|      |    |      |    |      |    |      |    | foo  |    |      |    |      |    |      |    |      |
|      | => |      | => |      | => | bar  | => | bar  | => | bar  | => |      | => |      | => |      |
|      |    |      |    | baz  |    | baz  |    | baz  |    | baz  |    | baz  |    |      |    |      |
|      |    | main |    | main |    | main |    | main |    | main |    | main |    | main |    |      |
            파일 실행      baz 호출     bar 호출     foo 호출     foo 종료     bar 종료     baz 종료     파일 종료
```

## See also

- [토스ㅣSLASH 21 - 프론트엔드 웹 서비스에서 우아하게 비동기 처리하기](https://youtu.be/FvRtoViujGg)
